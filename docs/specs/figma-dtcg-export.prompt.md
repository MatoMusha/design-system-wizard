# Build Prompt — Figma → DTCG Token Export (`figc tokens export`)

> **Status:** Build specification / self-contained implementation brief.
> **Audience:** A strong JS engineer or coding agent who knows JavaScript and the Figma Plugin API surface, but does **not** know this codebase.
> **Deliverable:** A new `figc tokens export` subcommand that performs a one-way, read-only export of Figma variables **and text styles** into W3C DTCG-format JSON, ready for consumption by Style Dictionary v4.

This document is the authoritative build prompt. Implement exactly what it specifies. Where it says "DECIDE," the decision is already made and stated — do not relitigate it. The **worked example (§7)** is the ground truth: if any part of your implementation disagrees with the worked example, your implementation is wrong.

---

## 1. Goal & scope

Build a command that reads the design tokens modelled as **Figma Variables** — and the typography modelled as **Figma text styles** (§2.7) — in the currently-open Figma file and writes them out as **DTCG (Design Tokens Community Group) JSON** files on disk. The export is:

- **One-way.** Figma is the single source of truth. The exporter never writes back to Figma. There is no import, no merge, no reconciliation. Code is downstream of design.
- **Read-only against Figma.** It calls only *read* Plugin-API methods (`getLocalVariableCollectionsAsync`, `getVariableByIdAsync`). It never mutates a node, style, or variable.
- **Deterministic.** Given the same Figma file, byte-identical output every run: stable key ordering, stable number formatting, stable file names. Clean diffs are a feature, not an accident.
- **Two-collection model.** The export understands exactly two logical collections:
  - **`primitives`** — the raw, context-free scales: color ramps, spacing steps, radii, type primitives. Leaf values are literals (a hex color, a pixel dimension, a number). Nothing aliases out of here; primitives are the bottom of the graph.
  - **`semantic`** — role tokens whose values are **aliases** into `primitives`, and whose names are engineered so that the downstream CSS variable names are exactly the fixed identifiers the target component library expects (`--background`, `--foreground`, `--primary`, `--primary-foreground`, `--border`, `--ring`, `--radius`, …). Semantic is the layer that gives raw values *meaning* and *theming*.
- **Mode-aware.** A collection's **modes** (e.g. `Light`, `Dark`) are the theming axis. Each mode holds one value per variable. This maps to a light base (`:root`) plus a dark override (`.dark`) in the emitted CSS.

**Text styles are in scope** (§2.7): Figma **text styles** are a separate Plugin-API primitive (not variables) that carry the type ramp, and the export reads them and emits them as DTCG `typography` composite tokens. Still **out of scope**: paint styles, effect styles, grid styles, components, non-variable/non-text-style data, any REST API usage, any network access beyond figc's existing local WebSocket.

### Why the Plugin API, not the REST API

figc is a **CLI + self-contained Figma plugin** that drives the *currently open* file over a **local WebSocket** using the **Figma Plugin API** — no personal access token, no cloud round-trip. The existing `figc vars` command deliberately returns only `{id, name, resolvedType}` per variable; it exposes **no values, no modes, no aliases**. Therefore the full token graph — values, modes, and alias edges — must be gathered by **running Plugin-API code** inside the plugin sandbox (via figc's existing `execute_code` transport), not by hitting `GET /v1/files/{key}/variables/local`. This is a hard constraint of the architecture, and it shapes the two-stage implementation in §6.

---

## 2. DTCG output format (the core of this document)

### 2.1 DTCG structure primer

DTCG files are JSON. The shape is a tree of **groups** (plain nested objects) with **tokens** at the leaves. A token leaf is an object carrying reserved `$`-prefixed members:

```jsonc
{
  "$value": <the value>,
  "$type": "<type keyword>",
  "$description": "<optional human text>"
}
```

Rules the exporter relies on:

- **Any object key not starting with `$` is a nesting level** (a group or a token). Keys starting with `$` are reserved metadata (`$value`, `$type`, `$description`, `$extensions`, and group-level `$type`).
- **Groups may carry a group-level `$type`** that all descendant tokens inherit unless they override it. The exporter uses this to reduce repetition (e.g. a `color` group stamps `$type: "color"` once at the top).
- **A token is identified by having `$value`.** No `$value` ⇒ it's a group.
- **References** between tokens are written as a curly-brace dotted path string in `$value`: `"{color.blue.500}"`. Style Dictionary resolves these and, with `outputReferences: true`, preserves them as `var(--…)` chains in the CSS.

### 2.2 `$type` mapping — from Figma `resolvedType` + semantics

Figma's `resolvedType` is one of `COLOR | FLOAT | STRING | BOOLEAN`. That is too coarse for DTCG, which distinguishes `dimension` from `number` and `fontFamily` from `string`. The exporter therefore derives `$type` from **`resolvedType` plus a naming convention** on the variable's path.

| Figma `resolvedType` | Condition (by top group segment / convention) | DTCG `$type` |
|---|---|---|
| `COLOR` | always | `color` |
| `FLOAT` | top group ∈ dimension set: `spacing`, `size`, `sizing`, `radius`, `radii`, `border-width`, `blur` | `dimension` |
| `FLOAT` | top group ∈ unitless set: `line-height`, `z-index`, `opacity`, `font-weight`, `number` | `number` |
| `FLOAT` | no matching group ⇒ **default** | `dimension` *(see note)* |
| `STRING` | top group ∈ `font-family`, `font`, `family` | `fontFamily` |
| `STRING` | otherwise | `string` |
| `BOOLEAN` | always | `boolean` |

**Disambiguation rule (define one, use it everywhere): the FLOAT split is driven by the first path segment of the variable name** (the segment before the first `/`). Maintain two allow-lists in code — `DIMENSION_GROUPS` and `NUMBER_GROUPS` — matched case-insensitively against that first segment. This is deterministic, requires no per-variable configuration in Figma, and is auditable. A FLOAT whose first segment is in neither list defaults to `dimension` **and emits a warning** to stderr (`unclassified FLOAT '<name>' defaulted to dimension`) so the maintainer can extend the allow-list or rename. Semantic tokens that are dimensions (e.g. `radius`) are classified the same way, by their own first segment.

> Rationale for the default: in these systems the overwhelming majority of bare FLOATs are dimensional (spacing/size/radius). Defaulting to `dimension` and warning is safer than silently emitting a unitless number that Style Dictionary would render without `px`.

### 2.3 `$value` formats per type

**`color` — DECISION: emit 6- or 8-digit lowercase hex string.**
Figma gives color as `{r,g,b,a}` floats in `0..1`. Convert each channel to an integer `0..255` via `Math.round(c * 255)`, format as two lowercase hex digits. If `a === 1` (within epsilon), emit 6-digit `#rrggbb`; otherwise emit 8-digit `#rrggbbaa`.

```
"$value": "#3b82f6"          // opaque
"$value": "#3b82f680"        // 50% alpha
```

*Why hex, not the DTCG color object:* the downstream target is CSS custom properties consumed by a component library whose theme contract is hex/`hsl`-style channel values. Hex is unambiguous, round-trips cleanly, and Style Dictionary's `css` transform group passes it through untouched. **oklch is offered as an opt-in** via `--color oklch` (converts the sRGB triple to `oklch(L C H)` / `oklch(L C H / a)` string); default is `hex`. The DTCG structured color object (`{colorSpace, components, alpha}`) is intentionally **not** used here to avoid a transform step and keep name/value parity trivial to verify.

**`dimension` — DECISION: emit the DTCG dimension object `{ "value": <number>, "unit": "px" }`.**
Style Dictionary v4 is DTCG-native and consumes the object form directly; the `css/variables` output renders it as `8px`. Figma FLOATs used as dimensions are unitless pixel counts, so `unit` is always `"px"` on export (rem conversion is a downstream Style Dictionary transform concern, not an export concern).

```
"$value": { "value": 8, "unit": "px" }
```

> Do **not** emit dimensions as the bare string `"8px"`. SD v4 accepts both, but the object form is the DTCG-canonical shape and keeps the value numerically inspectable for tests. Pick the object form and use it consistently.

**`number` — raw JSON number.** No unit, no string. `"$value": 1.5`.

**`fontFamily` — string, or array of strings for a stack.** If the Figma STRING contains commas, split on `,`, trim each, emit an array; otherwise emit a single string. `"$value": ["Inter", "sans-serif"]` or `"$value": "Inter"`.

**`string` / `boolean` — raw JSON string / boolean.**

### 2.4 Aliases → DTCG references

A Figma variable value can be an **alias**: `{ type: 'VARIABLE_ALIAS', id: 'VariableID:...' }` pointing at another variable. In DTCG this becomes a **reference string**: the `$value` is the target token's dotted path wrapped in braces, e.g. `"{color.blue.500}"`.

**Resolution algorithm (mandatory — build the index first):**

1. **Stage A dump** collects *every* variable across *all* exported collections, so the transform has the whole graph in memory before it resolves anything.
2. Build a single **`idToPath` index**: `Map<VariableID, string>` where the value is the variable's **global dotted DTCG path** (see the namespace convention in §2.6). Populate it in one pass over the whole graph, before emitting any token.
3. When a mode's value is an alias, look up `alias.id` in `idToPath`. Emit `"$value": "{" + idToPath.get(alias.id) + "}"`.
4. **Do not flatten.** Never resolve the alias to the primitive's literal value. Preserving the reference is what lets Style Dictionary's `outputReferences: true` emit `var(--color-blue-500)` inside the semantic property instead of an inlined hex — that indirection is the whole point of the two-tier model.

**Dangling alias:** if `alias.id` is not in `idToPath` (target not among exported collections, or deleted), that is a hard error — see §5 exit codes. Never emit a reference to a path that won't exist in the output.

**Cyclic alias guard:** while primitives-never-alias should make cycles impossible, defend anyway. When building references, walk the alias chain with a visited-set; if a variable's alias chain returns to itself, abort with a cyclic-alias error naming the cycle. (This is a validation pass, not a resolution — you still emit the one-hop reference, but you refuse to export a graph that contains a cycle.)

### 2.5 Cross-collection references and the global namespace

The critical subtlety: a **semantic** token aliases a **primitives** token — the reference must cross the collection boundary. DTCG references are resolved by Style Dictionary against **one merged token tree**, so both collections must live in a **single global path namespace** where every token has exactly one canonical dotted path, and that path is identical whether you're defining the token or referencing it.

**Convention (see §3 for the full transform):**

- **Primitives are namespaced** under their group structure exactly as named in Figma: `color/blue/500` → path `color.blue.500`. The collection name `primitives` is **not** a path segment — the primitive groups (`color`, `spacing`, `radius`, …) are already globally unique top-level namespaces.
- **Semantic tokens are NOT namespaced under a collection prefix either**, and are NOT nested under `color`/`spacing`. A semantic variable named `primary` becomes the top-level path `primary`; `primary/foreground` becomes `primary.foreground`. This is deliberate: the final CSS var must be `--primary`, `--primary-foreground` with no prefix, so the DTCG path that produces them must be `primary`, `primary.foreground` with no prefix.

Because primitives occupy `color.*`, `spacing.*`, `radius.*`, … and semantic tokens occupy `background`, `foreground`, `primary`, `primary.foreground`, `border`, `ring`, `radius`, … the two collections share **one flat global namespace with no collisions** (as long as no semantic role name equals a primitive top group — validate this; a semantic token literally named `color` would collide and must error). A reference from semantic to primitive is then just the primitive's global path: `"{color.blue.500}"`. Style Dictionary merges all input files into one tree and resolves it.

> **Edge case — semantic `radius`:** the semantic collection defines `radius` as a dimension token (often itself an alias to `primitives` `radius/md` or a literal). It lives at global path `radius`, producing `--radius`. The primitive ramp lives at `radius.sm`, `radius.md`, … (paths `radius.sm`…). Note `radius` (a token) and `radius.sm` (a token under an implied `radius` group) **cannot coexist** in strict DTCG — a node cannot be both a token and a group. Resolve this by naming the primitive ramp under a distinct group, e.g. Figma `radii/md` → `radii.md`, keeping the semantic `radius` token unambiguous. Document this naming rule for the design side; the exporter validates it and errors on a token/group collision.

### 2.6 `$extensions` — Figma provenance

Every emitted token carries an `$extensions` block namespaced under a reverse-DNS key so tooling can trace it back to Figma and so agents can debug parity issues:

```jsonc
"$extensions": {
  "com.designsystemwizard.figma": {
    "variableId": "VariableID:12:345",
    "collection": "semantic",
    "mode": "Light",
    "figmaName": "primary"
  }
}
```

- `variableId` — the Figma variable id (stable within the file; primary key for the `idToPath` index).
- `collection` — the Figma collection name the variable came from.
- `mode` — the mode this file/value represents (see §4; in multi-file mode output each file is single-mode, so this is unambiguous per file).
- `figmaName` — the original `/`-delimited Figma name, preserved verbatim for round-trip auditing (the DTCG path is derived and lossy w.r.t. casing; this is the source string).

`$extensions` is ignored by Style Dictionary's CSS output but survives in the token objects, so it is safe to carry and invaluable for the acceptance checks in §8.

---

### 2.7 Text styles → DTCG `typography` composite tokens

Typography is **not** modelled as variables. It is built as Figma **text styles** — a distinct Plugin-API primitive read via `figma.getLocalTextStylesAsync()`, which returns an array of `TextStyle` objects. Each `TextStyle` bundles several type fields (font, size, line height, letter spacing, …) into one named unit. The export reads them and emits each one as a single DTCG **`typography` composite token** (`$type: "typography"`, whose `$value` is an object of sub-members), the DTCG-canonical way to represent a whole text style as one token.

**Read is still read-only.** `getLocalTextStylesAsync()` is a *read* method; the exporter never calls `TextStyle` setters. This extends the Stage A dump (§6.2) with one extra call; it mutates nothing.

**Output layout.** All text styles land in a **single, non-mode-scoped file** at `<out>/typography.json`:

- **Text styles are NOT mode-scoped** the way variable collections are. A `TextStyle` holds exactly one value per field (not a per-mode map), so there is no `typography.light.json` / `typography.dark.json` split — there is one `typography.json`.
- **Mode variance still reaches typography through bound variables.** A text style field can *bind a variable* (§2.7 `boundVariables`, resolved in §2.9). When it does, the composite emits an **alias reference** to that variable's global path, and the variable — which *is* mode-scoped — carries the light/dark difference in its own `semantic.<mode>.json` file. So `typography.json` stays single-file, and Style Dictionary picks up the per-mode value by resolving the reference against whichever mode file it merged for that build. Do not try to fan `typography.json` out per mode.
- **Naming.** A text style's `name` uses `/` groups exactly like a variable name (`heading/lg` → path `heading.lg`). It runs through the **same** `pathSegments` + `normalizeSegment` transform as variables (§3) — no collection prefix — so the nesting, casing, and kebab rules are identical. Text-style paths share the one global namespace with variables; a text-style path that collides with a variable token/group path is a namespace collision (§5 exit 6), same as any other.

### 2.8 Text-style field → DTCG typography member mapping

Each `TextStyle` field maps to a DTCG `typography` sub-member (or to `$extensions` when it has no standard member). Conversions are exact and deterministic.

| `TextStyle` field | DTCG typography sub-member | Conversion rule |
|---|---|---|
| `fontName.family` | `fontFamily` | Literal string (e.g. `"Inter"`). If it contains commas, split/trim into a string array (same rule as the `fontFamily` variable type, §2.3). **If `boundVariables.fontFamily` exists ⇒ alias reference instead (§2.9).** |
| `fontName.style` | `fontWeight` | Map the weight word in the style name to a number (table below). **If `boundVariables.fontWeight` exists ⇒ alias reference instead.** |
| `fontSize` | `fontSize` | Dimension as a CSS-ready string `"<n>px"` (e.g. `"32px"`). **If `boundVariables.fontSize` exists ⇒ alias reference instead.** |
| `lineHeight` | `lineHeight` | `PERCENT` ⇒ **unitless multiplier** number = `value / 100` (`125` ⇒ `1.25`); `PIXELS` ⇒ dimension string `"<n>px"`; `AUTO` ⇒ the string `"normal"` (documented default — maps 1:1 to CSS `line-height: normal`). **If `boundVariables.lineHeight` exists ⇒ alias reference instead.** |
| `letterSpacing` | `letterSpacing` | `PERCENT` ⇒ em: `"<value/100>em"` (`0` ⇒ `"0em"`, `2` ⇒ `"0.02em"` — Figma percent is a fraction of the font size, i.e. em); `PIXELS` ⇒ `"<n>px"`. **If `boundVariables.letterSpacing` exists ⇒ alias reference instead.** |
| `textCase` | — | `$extensions` (not a standard typography member). |
| `textDecoration` | — | `$extensions`. |
| `paragraphSpacing` | — | `$extensions`. If `boundVariables.paragraphSpacing` exists, emit the reference there. |
| `paragraphIndent` | — | `$extensions`. If `boundVariables.paragraphIndent` exists, emit the reference there. |
| `id` | — | `$extensions.styleId` (provenance; primary key for the text style). |

> **Why string dimensions inside the composite, not the `{value,unit}` object.** Standalone `dimension` *variables* emit the DTCG object form (§2.3). But inside a `typography` composite `$value`, the sub-members are emitted as CSS-ready strings (`"32px"`, `"0em"`) so Style Dictionary's `typography/css/shorthand` transform (§2.10) can assemble a `font` shorthand without a per-sub-member unit step. Keep the object form for variables; use string form for typography composite members. Both are deterministic.

**Weight-name → number map** (matched case-insensitively against the tokens in `fontName.style`, after stripping any italic marker):

| Style word(s) | `fontWeight` |
|---|---|
| `Thin`, `Hairline` | `100` |
| `ExtraLight`, `UltraLight` | `200` |
| `Light` | `300` |
| `Regular`, `Normal`, `Book`, *(empty / unmatched)* | `400` |
| `Medium` | `500` |
| `SemiBold`, `DemiBold` | `600` |
| `Bold` | `700` |
| `ExtraBold`, `UltraBold` | `800` |
| `Black`, `Heavy` | `900` |

**Italic / oblique handling.** `fontName.style` may combine a weight word with an italic marker (`"Bold Italic"`, `"Italic"`, `"Medium Oblique"`). Strip the `Italic`/`Oblique` token first, map the remaining weight word (a bare `"Italic"` ⇒ weight `400`), and record the italic axis in `$extensions` as `"fontStyle": "italic"` (DTCG's `typography` composite has no `fontStyle` member; do not invent one in `$value`). A style whose weight word is unmatched after stripping defaults to `400` **and emits a warning** to stderr (`unmapped font style '<style>' defaulted to weight 400`), consistent with the FLOAT-default warning in §2.2.

### 2.9 Bound-variable handling in text styles

A `TextStyle` exposes a `boundVariables` map (`{ fontSize: {type:'VARIABLE_ALIAS', id}, fontFamily: {...}, … }`). Bindable fields include `fontSize`, `fontFamily`, `fontStyle`, `fontWeight`, `lineHeight`, `letterSpacing`, `paragraphSpacing`, `paragraphIndent`. For each mapped sub-member (§2.8):

1. If `boundVariables[field]` is present, **emit that sub-member as a DTCG alias reference** — `"{" + idToPath.get(alias.id) + "}"` — resolved through the **same `idToPath` index built for the variable export (§2.4/§2.6)**. Do **not** emit the literal. This preserves parity/`outputReferences` exactly as for variable aliases: a bound `fontSize` becomes `"{font-size.xl}"`, never `"32px"`.
2. If `boundVariables[field]` is absent, emit the literal per §2.8.

Because text styles resolve against the shared index, a text style **must be dumped alongside the variable collections** so the index is complete before any typography token is emitted (§6.2). The alias guards are identical to §2.4: a bound field whose `alias.id` is not in `idToPath` is a **dangling alias** (exit 3, naming the text style and field); the **cyclic-alias** visited-set walk (exit 5) covers text-style references too. A bound `fontFamily`/`fontWeight`/`fontSize`/`lineHeight`/`letterSpacing` should point at a primitive of the matching `$type` (a `fontSize` binding at a `dimension`, a `fontWeight` binding at a `number`, a `fontFamily` binding at a `fontFamily`); a gross type mismatch is a warning, not a hard error, since Figma permits it.

### 2.10 Dual output — atomic type primitives **and** composite typography

**RECOMMENDATION (do this):** expose the type system at **both** granularities.

- **Atomic type primitives as ordinary variables/tokens** — model `font-family/*`, `font-size/*`, `line-height/*`, `letter-spacing/*`, `font-weight/*` as Figma **variables**, exported by the normal variable path (§2–§3) into `primitives.json`. Extend the §2.2 allow-lists so these classify correctly: add `font-size` and `letter-spacing` to `DIMENSION_GROUPS`; `line-height` and `font-weight` are already in `NUMBER_GROUPS`; `font-family` STRINGs already classify as `fontFamily`.
- **Composite `typography` tokens** — the text styles of §2.7–§2.9, whose bound sub-members **reference** those primitives, exported into `typography.json`.

**Why both — the Style Dictionary implication.** SD v4's `css` transformGroup includes a `typography/css/shorthand` transform that turns a `typography` composite token into a single CSS **`font` shorthand** value (weight + size/line-height + family), optionally as a utility. Crucially, a composite typography token does **not** expand into one `--var` per sub-value — SD emits **one** custom property holding the shorthand, not `--heading-lg-font-size`, `--heading-lg-line-height`, etc. So the composite alone gives you a bundled `font:` string but **no** individual `--font-size-*` / `--line-height-*` custom properties for a utility framework to map. Exposing the atomic primitives too fills that gap:

- The atomic primitives emit flat custom properties — `--font-size-xl: 32px;`, `--font-family-sans: Inter, sans-serif;`, `--font-weight-bold: 700;`, `--line-height-tight: 1.25;`, `--letter-spacing-wide: 0.02em;` — which a **Tailwind** `@theme` (v4) or `theme.extend` (v3) maps straight onto utilities (`text-xl`, `font-sans`, `font-bold`, `leading-tight`, `tracking-wide`), keeping the utility layer and the component-library (shadcn/Radix) `--var` contract in parity.
- The composite typography tokens ride on top, emitting a bundled shorthand — e.g. `--heading-lg: 700 var(--font-size-xl)/1.25 var(--font-family-sans);` (with `outputReferences: true`, the bound sub-members stay `var(--…)` chains back to the atomic primitives) — ready to drop into a `.text-heading-lg { font: var(--heading-lg); }` utility/component class.
- **`letterSpacing` is not expressible in the CSS `font` shorthand**, so the shorthand transform drops it; a non-zero `letterSpacing` therefore needs its own declaration (`letter-spacing: var(--letter-spacing-wide);`). This is a second concrete reason to expose the atomic `letter-spacing` primitive, not only the composite.

Net: atomic primitives keep the Tailwind mapping and shadcn/Radix parity clean; composite typography keeps each semantic text-style bundle intact. The exporter produces both — variables via the existing path, text styles via §2.7–§2.9 — with no duplication of source-of-truth (the composite *references* the primitives rather than restating their values).

---

## 3. Name → token-path transform (deterministic, parity-preserving)

This is where CSS-name parity is won or lost. The transform turns a Figma `(collection, variable.name)` pair into a DTCG nesting path, which Style Dictionary then turns into a CSS custom property via its `name/kebab` transform. Every step must be lossless in the ways that matter and deterministic.

### 3.1 Algorithm

```
pathSegments(collection, figmaName):
  1. segments = figmaName.split('/')           // "primary/foreground" → ["primary","foreground"]
  2. segments = segments.map(normalizeSegment)  // see 3.2
  3. if collection is a PRIMITIVE collection:
        return segments                          // no collection prefix; groups are global namespaces
  4. if collection is the SEMANTIC collection:
        return segments                          // ALSO no prefix — names ARE the shadcn identifiers
  5. dtcgPath = segments.join('.')
```

Key decision, restated: **the collection name is never a path segment.** Primitives are globally namespaced by their own group roots (`color`, `spacing`, `radii`, …); semantic names must emit the fixed component-library identifiers directly, so they cannot be buried under a prefix. The two namespaces are kept disjoint by the naming discipline in §2.5, and the exporter validates disjointness.

### 3.2 `normalizeSegment` — casing & character rules

Figma names cannot contain `.`, `{`, `}` (guaranteed by Figma), which keeps them safe as DTCG path segments and reference bodies. Additional normalization to guarantee kebab parity:

- **Lowercase** the segment. (shadcn identifiers are all lowercase; `Primary` and `primary` must not diverge.)
- **Spaces and underscores → hyphens.** `"primary foreground"` and `"primary_foreground"` → `"primary-foreground"`.
- **Collapse repeated hyphens**, trim leading/trailing hyphens.
- **Preserve existing hyphens and digits.** `blue`, `500`, `2xl` pass through. Numeric-only segments (`500`, `4`) are valid path keys.
- **Reject** any segment that, after normalization, is empty or contains a character outside `[a-z0-9-]`. Error out (unsupported name) rather than silently mangle — silent mangling breaks parity invisibly.

Because segments are already kebab-safe (`[a-z0-9-]`), Style Dictionary's `name/kebab` transform (which joins the path with `-` and kebab-cases) produces exactly the intended identifier with no surprises.

### 3.3 Full chain for one real token

Figma semantic variable `primary`, Light mode, aliasing primitive `color/blue/500`:

```
Figma:        collection="semantic", name="primary"           (value = alias → color/blue/500)
  ↓ pathSegments (semantic ⇒ no prefix)
DTCG path:    ["primary"]  →  "primary"
  ↓ emitted token in tokens/semantic.light.json
              "primary": { "$value": "{color.blue.500}", "$type": "color", ... }
  ↓ Style Dictionary merges primitives + semantic, applies css transformGroup + name/kebab
CSS var name: --primary
  ↓ with outputReferences: true, value is the reference, not the literal
CSS:          --primary: var(--color-blue-500);
```

And the primitive it points at:

```
Figma:        collection="primitives", name="color/blue/500"  (value = {r,g,b})
DTCG path:    color.blue.500
CSS var:      --color-blue-500: #3b82f6;
```

Parity achieved: the semantic role `primary` emits `--primary`; the reference chain to `--color-blue-500` is preserved.

---

## 4. Modes / theming output strategy

**DECISION: multi-file, one file per (collection, mode).** For a `semantic` collection with `Light` and `Dark` modes plus a `primitives` collection with a single `Value`/`Mode 1` mode, emit:

```
tokens/
  primitives.json                 # primitives, single mode
  semantic.light.json             # semantic, Light mode values
  semantic.dark.json              # semantic, Dark mode values
```

Each semantic file is a **complete, single-mode** DTCG document: the same token paths in every mode file, differing only in `$value` (and the `mode` in `$extensions`). Primitives typically have one mode; if a primitive collection has multiple modes, emit `primitives.<mode>.json` per mode by the same rule.

**Why multi-file, not single-file-with-per-mode-extensions:** Style Dictionary builds one **platform per theme output**. The clean, idiomatic SD v4 setup is:

- **`:root` (light)** build: `source: ["tokens/primitives.json", "tokens/semantic.light.json"]`
- **`.dark` override** build: `source: ["tokens/primitives.json", "tokens/semantic.dark.json"]`, emitted under a `.dark` selector (via the `css/variables` `options.selector` or a custom format/filter).

Each build sees exactly one value per token, so references resolve unambiguously and the CSS for each theme is a flat, correct set of custom properties. A single file smuggling all modes into `$extensions` would force a custom SD parser to pick a mode — more moving parts, less idiomatic, harder to diff. Multi-file keeps each theme's diff isolated (a dark-only color change touches only `semantic.dark.json`).

**SD config implication (informative, not built here):**

```jsonc
// style-dictionary.config.js (downstream, illustrative)
export default {
  platforms: {
    cssLight: {
      transformGroup: 'css',                    // includes name/kebab
      source: ['tokens/primitives.json', 'tokens/semantic.light.json'],
      files: [{
        destination: 'theme.css', format: 'css/variables',
        options: { outputReferences: true, selector: ':root' }
      }]
    },
    cssDark: {
      transformGroup: 'css',
      source: ['tokens/primitives.json', 'tokens/semantic.dark.json'],
      files: [{
        destination: 'theme.dark.css', format: 'css/variables',
        options: { outputReferences: true, selector: '.dark' }
      }]
    }
  }
}
```

The exporter's contract is only to produce the per-mode files with matching token paths; wiring SD is the consumer's job, but the file naming above is chosen to make that wiring trivial.

---

## 5. File/output contract & CLI

### 5.1 Command

```
figc tokens export [--out <dir>]
                    [--collections primitives,semantic]
                    [--modes Light,Dark]
                    [--color hex|oklch]
                    [--text-styles | --no-text-styles]
                    [--format dtcg]
                    [--json]
```

| Flag | Default | Meaning |
|---|---|---|
| `--out <dir>` | `./tokens` | Output directory. Created if missing. |
| `--collections` | `primitives,semantic` | Which Figma collections to export, by name (case-insensitive match). Order is preserved for deterministic SD `source` ordering. |
| `--modes` | *(all modes of each collection)* | Restrict to named modes. Unknown mode name ⇒ error. |
| `--color` | `hex` | Color value form: 6/8-digit hex, or `oklch(...)`. |
| `--text-styles` / `--no-text-styles` | **on** | Export local Figma text styles as DTCG `typography` composite tokens into `typography.json` (§2.7–§2.9). On by default; `--no-text-styles` skips the text-style dump entirely (variable export is unaffected). |
| `--format` | `dtcg` | Output format. Only `dtcg` is supported in v1; the flag reserves the namespace. |
| `--json` | off | Also print a machine-readable summary of what was written (files, token counts, warnings) to stdout as JSON, for agent consumption. |

### 5.2 Output layout

```
<out>/
  primitives.json
  semantic.light.json
  semantic.dark.json
  typography.json            # text styles as DTCG typography composites (single, non-mode file; §2.7)
  .figc-tokens.meta.json     # export manifest: figc version, timestamp, source file key/name,
                             #   collections+modes exported, text-style count, per-file token counts, warnings
```

`typography.json` is omitted when `--no-text-styles` is passed or the file has no local text styles (an empty text-style set is a warning, not an error).

### 5.3 Determinism

- **Sort every object's keys** recursively before serialization, EXCEPT that `$`-members within a token are emitted in a fixed canonical order: `$value`, `$type`, `$description`, `$extensions`. Group `$type` is emitted first in a group. This yields byte-stable output regardless of Figma's `variableIds` ordering.
- **Number formatting:** emit numbers with JS default `JSON.stringify` (no trailing `.0`); round color channels deterministically (`Math.round`); do not emit `-0`.
- **Trailing newline**, 2-space indent, LF line endings.
- File names are lowercased and derived from `(collection, mode)` deterministically.

### 5.4 Exit codes & error handling

| Code | Condition |
|---|---|
| `0` | Success. Files written. Warnings may have been printed to stderr. |
| `1` | Generic/unexpected failure (transport error, write failure). |
| `2` | **Missing collection** — a `--collections` name matched no Figma collection. |
| `3` | **Dangling alias** — an alias target id is not among exported variables. Message names the source token and the missing id. |
| `4` | **Unsupported type / unclassifiable value** — e.g. a value shape the mapper doesn't handle, or a segment with illegal characters after normalization. |
| `5` | **Cyclic alias** — a cycle detected in the alias graph; message names the cycle. |
| `6` | **Namespace collision** — a semantic role name collides with a primitive top group, or a token/group path collision (§2.5 `radius`/`radii` case). |

Warnings (non-fatal, stderr): unclassified FLOAT defaulted to dimension; a mode present in Figma but excluded by `--modes`; an empty collection. All warnings are also collected into the `.figc-tokens.meta.json` manifest and the `--json` summary.

---

## 6. Implementation notes for the figc author

### 6.1 Where it plugs into figc

figc's pattern (see `src/codegen.js` + `src/cli.js`): each command has a `codegen.*Code()` function returning a **string of Plugin-API code**, which the CLI ships to the plugin via `call('execute_code', { code })`; the plugin runs it in the Figma sandbox and returns JSON. `figc vars` already does this narrowly. Extend the same way, but split the work into **two stages** so the DTCG transform runs in Node (testable) rather than in the sandbox.

### 6.2 Two-stage architecture

**Stage A — raw graph dump (in Figma sandbox).** Add `codegen.tokensExportCode(collectionNames)` returning Plugin-API code that walks the requested collections and returns the **complete raw variable graph** as plain JSON — no transformation, no DTCG, no opinions. Shape:

```jsonc
{
  "collections": [
    {
      "name": "primitives",
      "id": "VariableCollectionId:...",
      "defaultModeId": "1:0",
      "modes": [{ "modeId": "1:0", "name": "Value" }],
      "variables": [
        {
          "id": "VariableID:10:1",
          "name": "color/blue/500",
          "resolvedType": "COLOR",
          "valuesByMode": { "1:0": { "r": 0.231, "g": 0.510, "b": 0.965, "a": 1 } }
        }
        // ...
      ]
    }
    // semantic collection, etc.
  ]
}
```

The Stage A code calls only `figma.variables.getLocalVariableCollectionsAsync()` and `figma.variables.getVariableByIdAsync(id)`, filters collections by the requested names, and serializes `valuesByMode` verbatim (literals stay literals; aliases stay `{type:'VARIABLE_ALIAS', id}`). It performs **zero** interpretation. This keeps the sandbox code trivial and stable.

**Stage A also dumps text styles** (unless `--no-text-styles`). The same Plugin-API code additionally calls `figma.getLocalTextStylesAsync()` and appends a sibling `textStyles` array to the dump — again verbatim, no interpretation. `getLocalTextStylesAsync` is read-only, keeping the read-only guarantee intact:

```jsonc
{
  "collections": [ /* … as above … */ ],
  "textStyles": [
    {
      "id": "S:abc123",
      "name": "heading/lg",
      "fontName": { "family": "Inter", "style": "Bold" },
      "fontSize": 32,
      "lineHeight": { "value": 125, "unit": "PERCENT" },
      "letterSpacing": { "value": 0, "unit": "PERCENT" },
      "paragraphSpacing": 0,
      "paragraphIndent": 0,
      "textCase": "ORIGINAL",
      "textDecoration": "NONE",
      "boundVariables": {
        "fontSize":   { "type": "VARIABLE_ALIAS", "id": "VariableID:10:9" },
        "fontFamily": { "type": "VARIABLE_ALIAS", "id": "VariableID:10:8" }
      }
    }
    // …
  ]
}
```

Serialize `fontName`, `lineHeight`, `letterSpacing`, and `boundVariables` exactly as the API returns them; the `PERCENT`/`PIXELS`/`AUTO` interpretation and the weight-name mapping are Stage B's job (§2.8), not the sandbox's.

**Stage B — DTCG transform (in Node).** The CLI (`case 'tokens':`) receives the raw graph JSON and runs a set of **pure functions** in Node to produce the DTCG files:

- `buildIdIndex(graph) → Map<id, dtcgPath>` (§2.4/§2.6)
- `classifyType(resolvedType, firstSegment) → dtcg $type` (§2.2)
- `formatValue(value, $type, colorMode) → $value` (§2.3; handles alias → reference via the index)
- `pathSegments(collection, figmaName) → string[]` + `normalizeSegment` (§3)
- `buildTypographyTokens(graph.textStyles, index, opts) → { [path]: dtcgTypographyToken }` — pure transform of the text-style dump into `typography` composites: maps each field per §2.8 (`mapFontWeight`, `convertLineHeight`, `convertLetterSpacing` helpers), resolves each `boundVariables[field]` to an alias reference via the **same `index`** (§2.9), attaches `$extensions`, and nests by `pathSegments`.
- `assembleDocs(graph, index, opts) → { [filename]: dtcgObject }` (nest tokens into groups, attach `$extensions`, stamp group `$type`; folds `buildTypographyTokens` output into `typography.json`)
- `validate(graph, index)` → throws typed errors for exit codes 2–6 (covers text-style dangling/cyclic bound-variable references too, §2.9)
- `serialize(dtcgObject) → string` (deterministic key sort, §5.3)

Then write files + manifest. Because Stage B is pure `graph → files` with no Figma or filesystem coupling inside the transform functions, it is **fully unit-testable** against fixture graphs — mirror figc's existing `test/` layout with fixtures for: a color primitive, a dimension primitive, a semantic alias across collections, multi-mode splitting, a dangling alias, a cyclic alias, a namespace collision, **a text style with all-literal fields, a text style with bound `fontSize`/`fontFamily` (reference output), and the weight-name → number map (incl. an italic style)**. Test the worked example in §7 end-to-end as a golden-file test.

### 6.3 CLI wiring sketch

```js
// src/cli.js — new case, following the existing pattern
case 'tokens': {
  if (rest[0] !== 'export') throw new Error('Usage: figc tokens export [--out dir] ...');
  const cols = (flags.collections || 'primitives,semantic').split(',');
  const textStyles = flags['text-styles'] !== false;   // default on; --no-text-styles disables
  const raw = await call('execute_code', { code: codegen.tokensExportCode(cols, { textStyles }) });
  const result = transformToDtcg(raw, {          // pure Stage B, from src/tokens/*.js
    out: flags.out || './tokens',
    modes: flags.modes ? String(flags.modes).split(',') : null,
    color: flags.color || 'hex',
    textStyles,                                  // Stage B builds typography.json when true
  });
  writeDtcgFiles(result, deps.writeFile || writeFileSync);  // injectable for tests
  return flags.json ? out(result.summary) : { output: `wrote ${result.files.length} files to ${result.outDir}` };
}
```

Keep the transform modules under `src/tokens/` so `codegen.js` stays a thin string-emitter and the testable logic is isolated.

---

## 7. Fully worked example (mandatory, internally consistent)

Four Figma variables across two collections and two semantic modes.

### 7.1 Source variables in Figma

- **`primitives`** collection, one mode `Value` (`modeId "1:0"`):
  - `color/blue/500` — COLOR — `{ r: 0.231, g: 0.510, b: 0.965, a: 1 }`
  - `spacing/4` — FLOAT — `16`
- **`semantic`** collection, two modes `Light` (`modeId "2:0"`), `Dark` (`modeId "2:1"`):
  - `primary` — COLOR — Light: alias → `color/blue/500`; Dark: alias → `color/blue/500`
  - `primary/foreground` — COLOR — Light: alias → (a white primitive, `color/base/white`); Dark: alias → `color/blue/500`

*(For brevity the white primitive `color/base/white` = `{r:1,g:1,b:1,a:1}` is assumed present in `primitives`; shown in output.)*

### 7.2 (a) Raw Plugin-API graph JSON (Stage A output)

```json
{
  "collections": [
    {
      "name": "primitives",
      "id": "VariableCollectionId:1:0",
      "defaultModeId": "1:0",
      "modes": [{ "modeId": "1:0", "name": "Value" }],
      "variables": [
        { "id": "VariableID:10:1", "name": "color/base/white", "resolvedType": "COLOR",
          "valuesByMode": { "1:0": { "r": 1, "g": 1, "b": 1, "a": 1 } } },
        { "id": "VariableID:10:2", "name": "color/blue/500", "resolvedType": "COLOR",
          "valuesByMode": { "1:0": { "r": 0.231, "g": 0.510, "b": 0.965, "a": 1 } } },
        { "id": "VariableID:10:3", "name": "spacing/4", "resolvedType": "FLOAT",
          "valuesByMode": { "1:0": 16 } }
      ]
    },
    {
      "name": "semantic",
      "id": "VariableCollectionId:2:0",
      "defaultModeId": "2:0",
      "modes": [{ "modeId": "2:0", "name": "Light" }, { "modeId": "2:1", "name": "Dark" }],
      "variables": [
        { "id": "VariableID:20:1", "name": "primary", "resolvedType": "COLOR",
          "valuesByMode": {
            "2:0": { "type": "VARIABLE_ALIAS", "id": "VariableID:10:2" },
            "2:1": { "type": "VARIABLE_ALIAS", "id": "VariableID:10:2" }
          } },
        { "id": "VariableID:20:2", "name": "primary/foreground", "resolvedType": "COLOR",
          "valuesByMode": {
            "2:0": { "type": "VARIABLE_ALIAS", "id": "VariableID:10:1" },
            "2:1": { "type": "VARIABLE_ALIAS", "id": "VariableID:10:2" }
          } }
      ]
    }
  ]
}
```

### 7.3 idToPath index (Stage B, built first)

```
VariableID:10:1 → color.base.white
VariableID:10:2 → color.blue.500
VariableID:10:3 → spacing.4
VariableID:20:1 → primary
VariableID:20:2 → primary.foreground
```

### 7.4 (b) DTCG files produced

**`tokens/primitives.json`**

```json
{
  "color": {
    "$type": "color",
    "base": {
      "white": {
        "$value": "#ffffff",
        "$type": "color",
        "$extensions": {
          "com.designsystemwizard.figma": {
            "variableId": "VariableID:10:1",
            "collection": "primitives",
            "mode": "Value",
            "figmaName": "color/base/white"
          }
        }
      }
    },
    "blue": {
      "500": {
        "$value": "#3b82f6",
        "$type": "color",
        "$extensions": {
          "com.designsystemwizard.figma": {
            "variableId": "VariableID:10:2",
            "collection": "primitives",
            "mode": "Value",
            "figmaName": "color/blue/500"
          }
        }
      }
    }
  },
  "spacing": {
    "4": {
      "$value": { "value": 16, "unit": "px" },
      "$type": "dimension",
      "$extensions": {
        "com.designsystemwizard.figma": {
          "variableId": "VariableID:10:3",
          "collection": "primitives",
          "mode": "Value",
          "figmaName": "spacing/4"
        }
      }
    }
  }
}
```

*(Color channel check: `0.231×255=58.9→59=0x3b`, `0.510×255=130.05→130=0x82`, `0.965×255=246.1→246=0xf6` ⇒ `#3b82f6`. ✓)*

**`tokens/semantic.light.json`**

```json
{
  "primary": {
    "$value": "{color.blue.500}",
    "$type": "color",
    "$extensions": {
      "com.designsystemwizard.figma": {
        "variableId": "VariableID:20:1",
        "collection": "semantic",
        "mode": "Light",
        "figmaName": "primary"
      }
    },
    "foreground": {
      "$value": "{color.base.white}",
      "$type": "color",
      "$extensions": {
        "com.designsystemwizard.figma": {
          "variableId": "VariableID:20:2",
          "collection": "semantic",
          "mode": "Light",
          "figmaName": "primary/foreground"
        }
      }
    }
  }
}
```

> **Note the `primary` token/group coexistence:** `primary` is a token (`$value`) *and* has a child `foreground`. DTCG permits members alongside `$value` (keys not starting with `$` are children); Style Dictionary treats `primary` as a token and `primary.foreground` as a nested token, yielding `--primary` and `--primary-foreground`. This is the intended shape and is the reason semantic names use `/` for the foreground pairing. (Contrast the `radius`/`radii` rule in §2.5, which avoids a *primitive ramp* colliding with a *semantic scalar* — a different situation.)

**`tokens/semantic.dark.json`**

```json
{
  "primary": {
    "$value": "{color.blue.500}",
    "$type": "color",
    "$extensions": {
      "com.designsystemwizard.figma": {
        "variableId": "VariableID:20:1",
        "collection": "semantic",
        "mode": "Dark",
        "figmaName": "primary"
      }
    },
    "foreground": {
      "$value": "{color.blue.500}",
      "$type": "color",
      "$extensions": {
        "com.designsystemwizard.figma": {
          "variableId": "VariableID:20:2",
          "collection": "semantic",
          "mode": "Dark",
          "figmaName": "primary/foreground"
        }
      }
    }
  }
}
```

*(In Dark, `primary/foreground` aliases blue/500 instead of white — the only value difference from the light file, exactly as designed.)*

### 7.5 (c) CSS emitted by Style Dictionary

Light build (`source: primitives.json + semantic.light.json`, `selector: ':root'`, `outputReferences: true`):

```css
:root {
  --color-base-white: #ffffff;
  --color-blue-500: #3b82f6;
  --spacing-4: 16px;
  --primary: var(--color-blue-500);
  --primary-foreground: var(--color-base-white);
}
```

Dark build (`source: primitives.json + semantic.dark.json`, `selector: '.dark'`):

```css
.dark {
  --primary: var(--color-blue-500);
  --primary-foreground: var(--color-blue-500);
}
```

Parity confirmed end-to-end: `semantic/primary` → `--primary`; `semantic/primary/foreground` → `--primary-foreground`; alias chains preserved as `var(--…)`; the only light/dark delta is the one intended value.

### 7.6 Text style worked example (typography composite with bound fields)

Extends the same file with two extra **primitive variables** (in the `primitives` collection, mode `Value`) and one **text style**:

- `font-family/sans` — STRING (`fontFamily`) — `"Inter, sans-serif"` — id `VariableID:10:8` → path `font-family.sans`
- `font-size/xl` — FLOAT (dimension; `font-size` added to `DIMENSION_GROUPS`, §2.10) — `32` — id `VariableID:10:9` → path `font-size.xl`
- Text style `heading/lg` — `fontName {family:"Inter", style:"Bold"}`, `fontSize` **bound** to `font-size/xl`, `fontFamily` **bound** to `font-family/sans`, `lineHeight {value:125, unit:"PERCENT"}`, `letterSpacing {value:0, unit:"PERCENT"}`.

The `idToPath` index (§7.3) gains: `VariableID:10:8 → font-family.sans`, `VariableID:10:9 → font-size.xl`, and the text style is keyed by its own `styleId`.

**(a) Raw Plugin-API dump (Stage A `textStyles[]` entry)**

```json
{
  "id": "S:abc123",
  "name": "heading/lg",
  "fontName": { "family": "Inter", "style": "Bold" },
  "fontSize": 32,
  "lineHeight": { "value": 125, "unit": "PERCENT" },
  "letterSpacing": { "value": 0, "unit": "PERCENT" },
  "paragraphSpacing": 0,
  "paragraphIndent": 0,
  "textCase": "ORIGINAL",
  "textDecoration": "NONE",
  "boundVariables": {
    "fontSize":   { "type": "VARIABLE_ALIAS", "id": "VariableID:10:9" },
    "fontFamily": { "type": "VARIABLE_ALIAS", "id": "VariableID:10:8" }
  }
}
```

**(b) DTCG typography token produced — `tokens/typography.json`**

`fontSize` and `fontFamily` are bound ⇒ alias references (§2.9); `fontWeight` derives from `"Bold"` ⇒ `700` (§2.8 map); `lineHeight` `125 PERCENT` ⇒ unitless `1.25`; `letterSpacing` `0 PERCENT` ⇒ `"0em"`; `textCase`/`textDecoration`/`paragraphSpacing`/`paragraphIndent` go to `$extensions`.

```json
{
  "heading": {
    "lg": {
      "$type": "typography",
      "$value": {
        "fontFamily": "{font-family.sans}",
        "fontSize": "{font-size.xl}",
        "fontWeight": 700,
        "lineHeight": 1.25,
        "letterSpacing": "0em"
      },
      "$extensions": {
        "com.designsystemwizard.figma": {
          "styleId": "S:abc123",
          "figmaName": "heading/lg",
          "textCase": "ORIGINAL",
          "textDecoration": "NONE",
          "paragraphSpacing": 0,
          "paragraphIndent": 0
        }
      }
    }
  }
}
```

The two bound primitives also appear as ordinary tokens in `tokens/primitives.json` (dual output, §2.10):

```json
{
  "font-family": { "sans": { "$value": ["Inter", "sans-serif"], "$type": "fontFamily", "$extensions": { "com.designsystemwizard.figma": { "variableId": "VariableID:10:8", "collection": "primitives", "mode": "Value", "figmaName": "font-family/sans" } } } },
  "font-size":   { "xl":   { "$value": { "value": 32, "unit": "px" }, "$type": "dimension", "$extensions": { "com.designsystemwizard.figma": { "variableId": "VariableID:10:9", "collection": "primitives", "mode": "Value", "figmaName": "font-size/xl" } } } }
}
```

**(c) CSS emitted by Style Dictionary** (`css` transformGroup incl. `typography/css/shorthand`, `outputReferences: true`)

The atomic primitives emit flat custom properties; the composite emits a single `font` shorthand var that references them (bound sub-members stay `var(--…)`):

```css
:root {
  --font-family-sans: Inter, sans-serif;
  --font-size-xl: 32px;
  /* composite typography → one shorthand var, NOT one var per sub-value */
  --heading-lg: 700 var(--font-size-xl)/1.25 var(--font-family-sans);
}
```

Consumed as `.text-heading-lg { font: var(--heading-lg); }`. Note `letterSpacing: 0em` is absent from the shorthand (`font:` cannot carry it); a non-zero value would need `letter-spacing: var(--letter-spacing-*);` from the atomic primitive — the §2.10 rationale in miniature. Tailwind maps the atomics to utilities (`text-xl`, `font-sans`) in the same pass.

---

## 8. Acceptance criteria

Ship only when all of the following hold. Each is a testable assertion; several map directly to the golden-file test in §6.2/§7.

- [ ] **Round-trip name parity.** For every semantic variable, the emitted CSS custom property equals the component-library identifier exactly (`semantic/primary` ⇒ `--primary`, `semantic/primary/foreground` ⇒ `--primary-foreground`, `--background`, `--foreground`, `--border`, `--ring`, `--radius`, …). No prefix, correct kebab-casing, correct lowercasing.
- [ ] **Aliases preserved as references.** No semantic color's `$value` is a literal; each is a `"{...}"` reference resolving to an existing primitive path. With `outputReferences: true`, the CSS shows `var(--…)`, not inlined hex.
- [ ] **Cross-collection references resolve.** A `semantic` token referencing a `primitives` token produces valid CSS after SD merges both files; no unresolved-reference errors.
- [ ] **Modes split correctly.** One file per (collection, mode); every mode file has identical token paths; only `$value` and `$extensions.mode` differ across a collection's mode files.
- [ ] **`$type` mapping correct.** COLOR⇒`color`; dimensional FLOAT⇒`dimension` (object `{value,unit:"px"}`); unitless FLOAT⇒`number`; font-family STRING⇒`fontFamily`; other STRING⇒`string`; BOOLEAN⇒`boolean`. Unclassified FLOAT defaults to `dimension` **and** emits a warning.
- [ ] **Deterministic output.** Running the export twice on an unchanged file yields byte-identical files (sorted keys, canonical `$`-member order, stable number/color formatting, trailing newline). A no-op re-export is an empty `git diff`.
- [ ] **No hardcoded values leaking.** Semantic layer contains references only; literals live solely in `primitives`. Colors are valid `#rrggbb`/`#rrggbbaa` (or `oklch(...)` under `--color oklch`); dimensions carry `unit:"px"`.
- [ ] **Provenance present.** Every token carries `$extensions."com.designsystemwizard.figma"` — variable tokens with `variableId`, `collection`, `mode`, `figmaName`; typography tokens with `styleId`, `figmaName`, and the non-standard fields (`textCase`, `textDecoration`, `paragraphSpacing`, `paragraphIndent`).
- [ ] **Text styles exported as typography composites.** Each local text style becomes one `$type: "typography"` token in `typography.json`, keyed by its `/`-path through the §3 transform. `typography.json` is single-file (not mode-split); omitted only under `--no-text-styles` or when no text styles exist.
- [ ] **Text-style field mapping correct.** `fontFamily`/`fontSize`/`fontWeight`/`lineHeight`/`letterSpacing` populated per §2.8: `lineHeight` `PERCENT`⇒unitless number, `PIXELS`⇒`"Npx"`, `AUTO`⇒`"normal"`; `letterSpacing` `PERCENT`⇒`em`, `PIXELS`⇒`px`; `fontSize`⇒`"Npx"` string.
- [ ] **Weight-name mapping correct.** `fontName.style` maps by the §2.8 table (Thin 100 … Regular 400 … Medium 500 … SemiBold 600 … Bold 700 … Black 900); italic marker stripped before mapping and recorded as `$extensions.fontStyle`; unmatched style ⇒ weight 400 + warning.
- [ ] **Bound text-style fields preserved as references.** A `boundVariables[field]` emits `"{path}"` resolved via the shared `idToPath` index — never the literal — and resolves to an existing primitive after SD merges. Dangling ⇒ exit 3; cyclic ⇒ exit 5.
- [ ] **Dual output present.** Atomic type primitives (`font-family/*`, `font-size/*`, `line-height/*`, `letter-spacing/*`, `font-weight/*`) export as ordinary tokens **and** the composite typography tokens reference them; SD emits flat `--font-size-*` etc. custom properties plus one shorthand `--<name>` per composite (not one var per sub-value).
- [ ] **Error paths exit correctly.** Missing collection ⇒ exit 2; dangling alias ⇒ exit 3; unsupported type / illegal segment ⇒ exit 4; cyclic alias ⇒ exit 5; namespace/token-group collision ⇒ exit 6. Each with a message naming the offending token.
- [ ] **Read-only.** The export makes zero mutating Plugin-API calls; the Figma file is untouched. Stage A calls only `getLocalVariableCollectionsAsync` / `getVariableByIdAsync` / `getLocalTextStylesAsync`.
- [ ] **Manifest written.** `.figc-tokens.meta.json` records figc version, timestamp, source file identity, exported collections+modes, per-file token counts, and all warnings; `--json` prints the same summary to stdout.
- [ ] **Worked example is a golden test.** The §7 fixture graph produces exactly the §7.4 files and the §7.5 CSS, and the §7.6 text style produces exactly the §7.6 `typography.json` token and its shorthand CSS (via a real SD v4 build in CI).
```