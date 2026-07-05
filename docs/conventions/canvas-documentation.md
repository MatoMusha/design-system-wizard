# The Canvas Documentation Standard

**Status:** Normative standard for `design-system-wizard`
**Applies to:** every foundation and every component in a design system managed by this plugin
**Consumed by:** `/setup` (emits foundation + component doc pages), `/extend` (emits a doc frame per new component), `/audit` (scores via the **Canvas Documentation Checklist**, §11)
**Related conventions:** [Component Contract](./component-contract.md) · [Craft & Measurement](./craft-and-measurement.md) · [Agent-Readability](./agent-readability.md)
**Built by:** `figma-doc-builder` (renders) via `figc-operator` (the sole Figma interface)

---

## 0. Mandatory and non-skippable

The canvas documentation layer is **always built** — it is not an optional finishing step, and no run of `/setup` or `/extend` is complete without it. `figma-doc-builder` **MUST** emit the full canonical file structure (§2) on every `/setup`, and append a conformant doc card on every `/extend`. There is no mode, flag, or shortcut that skips it.

`/audit` treats a **missing** doc layer, a **stale** doc layer, or a **structurally wrong** doc layer (scattered components, overlapping frames, absolute positioning, missing canonical pages) as a **hard failure** — not a warning (§11). "The tokens and components exist but the Foundations/Components pages were never built" is exactly the failure this standard exists to prevent.

Every doc frame **MUST also conform to [Craft & Measurement](./craft-and-measurement.md)**: auto-layout only (nothing absolute-positioned, nothing overlapping), all spacing/padding/gaps bound to `space/*` tokens on the 4px base, consistent outer margins, Lucide as the icon foundation, and the craft-verification loop (`figc shot` + inspect) run on every frame before it is considered done. This standard defines *what* the canvas docs contain and *how they are laid out at the file level*; Craft & Measurement defines the *pixel-level spatial discipline* every frame here must satisfy.

---

## 1. Two surfaces, one content

A design system managed by this plugin has **two documentation surfaces**, and they render the **same underlying content**:

| Surface | Medium | Governed by | Audience |
|---|---|---|---|
| **Code / web docs** | Markdown / MDX / Storybook doc blocks | [Component Contract](./component-contract.md) | Read in the editor, the repo, the docs site |
| **Canvas docs** | Documentation **frames rendered on the Figma canvas** | **this standard** | Read by designers *inside the Figma file*, where the work happens |

Both are **projections of one source**: `ds-manifest.json` (tokens, component inventory, relationships) plus each component's **Component Contract** record. The web docs render that source to Markdown/Storybook; the canvas docs render the *same* source to Figma frames. There is no third, hand-drawn "design copy" of the documentation — a swatch's value, a specimen's specs, a component's Do/Don't all trace to the same fields the Contract exposes.

**The core rule is the same as the Component Contract's: one source, two projections.** If a fact appears on the canvas but is not derivable from the manifest/Contract (or vice-versa), the canvas doc is malformed.

### 1.1 The sync rule (generation, not hand-authoring)

Canvas docs are **generated and regenerated** from the source, never hand-maintained. Concretely:

1. Every canvas doc **frame** is produced by `figma-doc-builder` from a source slice of `ds-manifest.json` / the Contract (§9).
2. Each frame records, in the manifest's `canvasDocs` index (§10), the **`sourceHash`** it was generated from and the **Figma node id** it lives at.
3. A canvas doc is **fresh** when its recorded `sourceHash` equals the current hash of its source slice. When the source changes (a token value edits, a variant is added, a Contract prop changes), the recorded hash no longer matches and the frame is **stale**.

`/audit` checks **presence AND freshness**: a missing doc page fails; a **stale** doc page (present but `sourceHash` ≠ current) **also fails**. Staleness is not a warning — a canvas doc that has drifted from the manifest is treated as wrong, because a designer reading it would be misled. The two surfaces must not diverge.

---

## 2. Canonical Figma file architecture (mandatory)

The canvas docs occupy a **fixed, ordered set of top-level pages** in the Figma file. This structure is **canonical and mandatory** — `/setup` always emits exactly these pages in this order, `/extend` appends into them, and `/audit` locates every piece by page and name. Components are **never** scattered one-per-page; foundations are **never** left unbuilt. The ordered pages are:

```
①  Cover                   (entry / index page, §8.3)
②  Foundations             (one doc frame per foundation, §3–§6 — Color, Type, Spacing, Radius, Elevation, Icons, Motion)
③  Components              (ALL component doc cards, together, in one auto-layout grid, §7)
④  Patterns                (composed multi-component patterns / recipes — optional, present when the system defines any)
    Changelog              (optional — generated version history of the doc layer)
```

| Page | Required | What lives here |
|---|---|---|
| **① Cover** | always | System name, version, and the linked index of every foundation frame and every component card (§8.3). The single entry surface. |
| **② Foundations** | always | One doc frame per foundation — Color, Typography, Spacing, Radius, Elevation, **Icons (Lucide)**, Motion — laid out on the shared grid (§2.1). Split into clearly-labelled sections on one page, or (only if the file is large) one page per foundation under a `②a Color`, `②b Type`… numbering that preserves order. |
| **③ Components** | always | **Every** component doc card, together, in a single **auto-layout grid** (§7). One page for the whole library; split to one page **per category** only when the library is large enough to warrant it — never one page per component. |
| **④ Patterns** | conditional | Composed, multi-component patterns/recipes (form layouts, page shells, empty states). Present when the manifest defines any pattern; each pattern is one doc card on the shared grid. |
| **Changelog** | optional | A generated record of doc-layer versions/regenerations. |

The page numbering prefix (`①②③④`) is part of the name so the pages sort in canonical order and `/audit` can assert their presence and sequence. Every leaf frame on these pages is a **documentation frame / doc card** built from the reusable template in §8, and every one conforms to Craft & Measurement (auto-layout, token-bound spacing, no overlaps).

### 2.1 One consistent layout grid (all doc pages and frames)

Every doc page and every doc frame shares **one layout system** — the docs must read as a single, evenly-gridded document, not a pile of loose frames:

- **Fixed outer margins:** a consistent outer padding on every page/frame — default **`space/1600` (64px)** — identical on all four sides. No frame hugs the edge; no frame floats at an arbitrary offset.
- **Column grid:** a shared column grid (e.g. 12 columns with a token-bound gutter) that all doc frames align to. Foundation specimens and component cards snap to column boundaries; nothing is nudged by eye.
- **Consistent section spacing:** the vertical rhythm between frames and between sections is a single spacing token (default **`space/1200` (48px)** between doc cards, **`space/800` (32px)** between sections within a frame) — never ad-hoc gaps.
- **Auto-layout only, no overlaps:** every page is organized with auto-layout containers and every frame is auto-layout (§8). **Nothing is absolute-positioned and no two frames overlap** — this is the Craft & Measurement rule, enforced here at the page level. Overlapping or free-floating frames are a **hard failure** (§11, C-STRUCT-5).

All of the above are driven from the system's own **spacing tokens** (4px base) — there are no magic numbers in the doc layout itself, per §8.1.

---

## 3. Foundation documentation pages

Each foundation gets a **documentation frame** using the §8 template. The set below is normative. Foundations marked **conditional** are required only when the system actually defines that axis (recorded in `ds-manifest.json`); the others are always required.

| Foundation | Required | Doc-frame content (summary) |
|---|---|---|
| **Color** | always | Swatch grid; primitives vs semantic separated; light + dark; swatches token-bound (§4). |
| **Typography** | always | One specimen per real Figma **text style**; a type-scale overview (§5). |
| **Spacing** | always | Ruler/box specimens per space token + name + value (§6.1). |
| **Radius / Shape** | always | Corner specimens per radius token + name + value (§6.2). |
| **Elevation / Shadow** | conditional* | Shadow cards per elevation token + name + value (§6.3). |
| **Icons** | always | **Lucide** icon grid — real icon instances at the system's icon sizes/stroke, with sizing and stroke rules and the icon component's usage (§6.4). |
| **Motion** | conditional | Duration/easing specimens; named motion tokens + values (§6.4). |

\* Elevation is **always required if the system defines any elevation/shadow tokens**; conditional only in flat systems that define none.

Each foundation doc frame follows the §8 skeleton (header → sections → footer) and derives every value from the manifest — never from a designer typing a hex or a pixel count into a label by hand.

---

## 4. The Color page (swatch grid)

The Color doc frame is a **swatch grid**. It is the single strictest foundation frame because its swatches must stay **live**.

### 4.1 Live-binding rule (normative)

Every swatch's rendered color fill **MUST be bound to the actual Figma variable** it documents (via `figc bind`), never a hardcoded paint. A swatch is a *view of the variable*, so when the variable's value changes the swatch updates and cannot silently lie. A hardcoded swatch fill is a **hard failure** in the checklist (§11, C-COLOR-2).

### 4.2 Structure

- **Two clearly separated regions:** **Primitives** (the raw scale — `blue/50…900`) above/apart from **Semantic** (intent aliases — `color/action/bg`, `color/text/muted`). The separation is visible and labelled, mirroring the two-tier token rule.
- **Light and dark shown.** Each semantic swatch shows its value in **both modes** — either as a split swatch (light half / dark half) or as paired light/dark grids driven by the collection's two modes. Mode is never inferred; it is rendered.
- **Grouped by family/intent** (action, surface, text, border, feedback…) with a group label per row band.

### 4.3 Swatch anatomy (per swatch)

```
┌──────────────────────────────┐
│  ▓▓▓▓▓▓▓▓▓▓  ← color chip (fill BOUND to the variable via figc bind)
│  ▓▓▓▓▓▓▓▓▓▓     for semantic tokens, split light │ dark
├──────────────────────────────┤
│  color/action/bg          ← token name  (canonicalName / Figma var path)
│  #1f6feb  ·  light        ← value(s), per mode
│  #4c8dff  ·  dark
│  Primary action surface   ← semantic usage (one line, from the token's intent)
│  aliases → blue/600       ← alias target (semantic swatches only)
└──────────────────────────────┘
```

Required fields per swatch: **rendered chip (bound)**, **token name**, **value(s) per mode**, **semantic usage**, and — for semantic tokens — the **alias target**. Primitive swatches omit usage/alias and are grouped as the raw scale.

---

## 5. The Typography page (specimen per text style)

Typography in this system is built as **real Figma text styles**. The Typography doc frame therefore renders a **specimen per text style**, and each specimen text **MUST use the actual Figma text style** it documents — not a manually-set family/size/weight. A specimen with detached, hand-set text is a **hard failure** (§11, C-TYPE-2).

### 5.1 Structure

- **A type-scale overview** at the top: every style rendered at one glance, in scale order (display → heading levels → body → caption → code), so the ramp reads as a ramp.
- **One specimen block per text style**, in the same order.

### 5.2 Specimen anatomy (per text style)

```
┌────────────────────────────────────────────────────────┐
│  The quick brown fox jumps over the lazy dog    ← sample line
│     rendered using the ACTUAL Figma text style
├────────────────────────────────────────────────────────┤
│  Heading / H2                     ← style name (Figma text-style path)
│  Inter · 24px · 600 · 32px LH · -0.01em LS   ← specs, read from the style
│  Section titles within a page     ← usage (one line)
└────────────────────────────────────────────────────────┘
```

Required per specimen: the **sample rendered with the real text style**, the **style name**, and its **specs** — family, size, weight, line-height, letter-spacing — read from the style itself, plus a one-line usage. The specs label is derived from the style, so it cannot drift from what the sample actually renders.

---

## 6. Spacing, Radius, Elevation, and conditional foundations

Each renders a **visual specimen + token name + value**, one per token, in scale order.

### 6.1 Spacing
A **ruler/box specimen** per space token: a bar or box whose width/height equals the token's value (with that dimension **bound** to the space variable where the tool allows), labelled with the token name (`space/300`) and value (`12px`). A stacked overview shows the whole ramp for proportion.

### 6.2 Radius / Shape
A **corner specimen** per radius token: a card whose corner radius equals the token (bound to the radius variable), labelled name (`radius/control`) + value (`8px`). Includes the full-round / pill case if defined.

### 6.3 Elevation / Shadow
A **shadow card** per elevation token: an identical surface carrying only that elevation's shadow effect (from the effect style/variable), labelled name (`elevation/raised`) + value (offset/blur/spread/color). Cards sit on a neutral ground so the shadow reads; shown in light and dark if elevations are mode-varying.

### 6.4 Icons (always) and Motion (conditional)
- **Icons** (always required): an **icon grid** of real **Lucide** icon instances at the system's defined icon size(s) and stroke width, plus the sizing/stroke rules and the icon component's usage. Icons are placed as instances (§7.1), never pasted vectors. Lucide is the icon foundation per Craft & Measurement; the grid is laid out on the shared column grid (§2.1) with token-bound gaps.
- **Motion** (when motion tokens exist): **duration and easing specimens** — a labelled sample per duration token and per easing curve — with the named motion token and its value. Where the canvas cannot animate, the specimen shows the curve and duration as an annotated graph plus the token value.

---

## 7. Component documentation frames

### 7.0 Components page layout (organized, together, auto-layout grid)

All component doc cards live **together on the ③ Components page** (or one page **per category** only when the library is large), arranged in a **single auto-layout grid** on the shared layout grid (§2.1). Each component gets **its own frame** — a **component doc card** — and the cards are laid into rows/columns by an auto-layout wrapper with token-bound gaps (default `space/1200` between cards). Cards are uniform in width and snap to the column grid.

**Explicitly forbidden (each a hard failure, §11):**
- **One-component-per-page scattering** — a separate page per component instead of the organized grid.
- **Overlapping frames** — any two doc cards (or their contents) intersecting.
- **Absolute positioning / free-floating cards** — cards placed by raw x/y instead of flowing inside the auto-layout grid.
- **Inconsistent margins/gaps** — cards with differing outer margins or ad-hoc spacing instead of the shared grid's token-bound rhythm.

Each **component doc card** is a doc frame (§8 template) that mirrors the component's **Component Contract** on the canvas and contains, in order:

1. **Header** — component name, one-line description, status, APG pattern (from the Contract).
2. **Variants & states matrix** — the component rendered **across all its variants and states** (e.g. every `variant` × `size`, plus hover/focus/disabled/loading where they are distinct visual states), each as a **real component instance**.
3. **Anatomy diagram** — one instance with **numbered callouts** naming its parts (container, label, icon slot, focus ring…), matching the Contract's anatomy.
4. **Usage / props summary** — a compact table of the key props (name · type · default) drawn from the Contract's `propsTable`, plus the canonical import. This is a *summary view* of the Contract, not a re-authored API.
5. **Do / Don't** — at least **one Do and one Don't**, each a **correct instance placed next to an incorrect one**, with a short caption and rationale (sourced from the Contract's `guidance` node). The correct example uses bound tokens and real instances; the Don't visibly shows the anti-pattern.

### 7.1 Instance rules (normative)

- Components are shown as **real instances placed via figc** (`figc place`) — **never detached**, **never resized** (`figc place` avoids the instance-resize anti-pattern), and never redrawn by hand. A detached or resized "component" in a doc frame is a **hard failure** (§11, C-CMP-2).
- Every variant/state combination that the Contract enumerates MUST appear in the matrix. Missing combinations fail the coverage check (§11, C-CMP-3).
- All fills/strokes visible in the frame are **token-bound**; the doc frame itself obeys the no-hardcoded-color rule.

### 7.2 Component doc-frame skeleton

```
Frame: "Components / Button"   (auto-layout, vertical, gap = space/500)
├─ Header
│    Button · "Triggers an action or event" · stable · APG: Button
├─ § Variants & states
│    matrix of real <Button> instances:
│    primary·secondary·ghost·destructive  ×  sm·md·lg
│    + hover / focus / disabled / loading states
├─ § Anatomy
│    one instance + numbered callouts (1 container · 2 label · 3 icon slot · 4 focus ring)
├─ § Usage & props
│    import line + compact props table (variant, size, isLoading, isDisabled…)  ← from Contract
└─ § Do / Don't
     Do:   ✓ one primary button per view      (correct instance)
     Don't: ✗ two primaries side by side       (incorrect instance)
     Do:   ✓ label on icon-only button
     Don't: ✗ icon-only button with no label
```

---

## 8. Layout & template conventions ("clean and comprehensive")

The canvas docs must **exemplify the design system they document** — dogfooding. They are built from the system's own tokens and components, so a broken token is visible in its own documentation.

### 8.1 The reusable doc-frame template

Every foundation and component doc frame is an instance of one **doc-frame template**:

- **Auto-layout**, vertical, top-to-bottom flow.
- **Spacing driven by the system's own spacing tokens** — the gap between sections, the padding, and the internal rhythm all reference `space/*` tokens (bound where the tool allows). No magic numbers.
- **Type from the system's own text styles**; **color from the system's own semantic tokens** (bound). The template contains **zero hardcoded colors or fonts**.
- A fixed **section order** (below), with a header block and a footer.

### 8.2 Frame skeleton (sections in order)

```
Frame: "<Area> / <Name>"          (auto-layout ▮ vertical ▮ gap = space token)
├─ Header block
│    ├─ Title            (H1 / display text style)
│    ├─ Description      (body text style — one or two lines)
│    └─ Meta row         (status · source version · last generated)
├─ Section 1 … N          (foundation- or component-specific, in the fixed order per §4–§7)
│    each: section heading (H2 style) + specimen content
└─ Footer
     ├─ "Generated by figma-doc-builder from ds-manifest <version>"
     └─ sourceHash (short) — the freshness stamp (§1.1)
```

The **section order is fixed per doc type** (Color's regions, Typography's overview-then-specimens, a component's Header → Variants → Anatomy → Usage → Do/Don't). `/setup` emits frames in this order; `/audit` checks presence and order; `/extend` fills the same skeleton for each new component.

### 8.3 Cover / index

The **① Cover** page (§2) is the entry surface: system name, version, and a linked index of every foundation frame and every component doc card. It is regenerated whenever a page or card is added or removed, so the index never lists a page that does not exist (or omits one that does).

---

## 9. How the canvas docs are built

### 9.1 Roles

- **`figma-doc-builder`** — a dedicated agent role that **owns rendering the canvas docs from the manifest**. It reads a source slice (a foundation's tokens, or a component's Contract), and emits the doc pages/frames, swatches, specimens, matrices, and Do/Don't pairs. It is invoked by `figc-operator`, or runs as its sibling, on `/setup` (all foundations + components) and on `/extend` (one component doc frame).
- **`figc-operator`** — the **sole** Figma interface (per the architecture roster). All of `figma-doc-builder`'s Figma writes go through it: creating pages/frames/auto-layout/text, placing component instances, binding token swatches, and screenshotting to verify.

### 9.2 figc mechanics

Canvas docs are built with figc:

- `figc exec '<plugin api code>'` — create the docs pages, auto-layout frames, text nodes, swatch/specimen nodes, and callouts; read text-style specs to label typography specimens; read token values to label swatches.
- `figc place <componentKey> [--parent id]` — place **real component instances** into component doc frames (never detach, never `resize()`).
- `figc bind <id> <fill|stroke> <Collection/token>` — bind each color swatch's fill to its variable, and bind the frame's own colors/spacing to semantic tokens.
- `figc shot <id>` — **screenshot-verify every frame after building it** and inspect the result before moving on.

### 9.3 The figc conventions that apply

The plugin-wide figc conventions are **normative for canvas docs**:

1. **Bind tokens, never hardcode** — swatches bind to variables; the doc frames' own colors/spacing/type reference semantic tokens.
2. **Never resize an instance** — components are placed with `figc place` and shown at their real sizes.
3. **Shot-verify** — every frame is `figc shot`-checked after it is rendered.

### 9.4 Regeneration

On any source change, `figma-doc-builder` **regenerates the affected frame(s)** rather than patching by hand, then updates the frame's `sourceHash` and node id in the manifest's `canvasDocs` index (§10). Regeneration is idempotent: rebuilding an unchanged source yields the same frame.

---

## 10. Manifest index — `canvasDocs`

The canvas docs are indexed in `ds-manifest.json` so `/audit` can locate each frame and check its freshness without crawling the Figma file. This is the machine anchor for §1.1 and §11.

```jsonc
"canvasDocs": {
  "coverPageId": "12:0",
  "foundations": [
    { "foundation": "color",      "required": true,  "figmaNodeId": "12:8",
      "sourceHash": "sha256:4a1c…", "generatedFrom": "acme-ds@4.3.0" },
    { "foundation": "typography", "required": true,  "figmaNodeId": "12:24", "sourceHash": "sha256:9f2e…" },
    { "foundation": "spacing",    "required": true,  "figmaNodeId": "12:40", "sourceHash": "sha256:71b0…" },
    { "foundation": "radius",     "required": true,  "figmaNodeId": "12:56", "sourceHash": "sha256:c3d1…" },
    { "foundation": "elevation",  "required": true,  "figmaNodeId": "12:72", "sourceHash": "sha256:0e88…" },
    { "foundation": "iconography","required": false, "figmaNodeId": null,     "sourceHash": null },
    { "foundation": "motion",     "required": false, "figmaNodeId": null,     "sourceHash": null }
  ],
  "components": [
    { "id": "cmp.button", "figmaNodeId": "34:12", "sourceHash": "sha256:aa10…",
      "variantsShown": 12, "variantsExpected": 12, "doCount": 2, "dontCount": 2,
      "contractRef": "contracts/button.json" }
  ]
}
```

- `sourceHash` is the hash of the exact source slice the frame was generated from (a foundation's token set; a component's Contract record). Freshness = recorded `sourceHash` equals the recomputed hash of the live source.
- `figmaNodeId` lets `/audit` (via `figc-operator`) fetch and inspect the actual frame — e.g. to confirm swatches are bound and specimens use text styles.

---

## 11. Evaluation — the Canvas Documentation Checklist

`/audit` runs this rubric against the canvas docs. Every check is **objective** — binary (PASS/FAIL) or graded (a `0.0–1.0` coverage ratio) — and verified against the `canvasDocs` index (§10) plus, where noted, direct inspection of the Figma node via `figc-operator`. Checks marked **[blocker]** must pass for the canvas docs to be considered conformant at all.

### Presence & structure
| ID | Type | Check |
|---|---|---|
| C-STRUCT-0 | **[blocker]** binary | The **canonical pages exist in order**: `① Cover`, `② Foundations`, `③ Components` (plus `④ Patterns` iff the system defines patterns). The doc layer was actually built — not skipped (§0, §2). |
| C-STRUCT-1 | **[blocker]** binary | The **① Cover / index** page exists and lists exactly the foundation frames and component doc cards that exist (no missing, no phantom entries). |
| C-STRUCT-2 | **[blocker]** graded | **Every required foundation** (§3, incl. **Icons**) has a doc frame present (`figmaNodeId` non-null): fraction of required foundations present. |
| C-STRUCT-3 | binary | Every **conditional** foundation that the system actually defines (motion/elevation tokens exist) has a doc frame present. |
| C-STRUCT-4 | graded | Fraction of foundation/component frames whose sections appear **in the fixed order** for their doc type (§8.2). |
| C-STRUCT-5 | **[blocker]** binary | **Components are together on the ③ Components page in one auto-layout grid** — not scattered one-per-page (§7.0). Fails if any component has its own dedicated page instead of a card in the grid. |
| C-STRUCT-6 | **[blocker]** binary | **No overlapping frames** anywhere in the doc layer — no two doc cards/frames (or their contents) intersect, and none are absolute-positioned/free-floating outside the auto-layout flow (§2.1, §7.0). |
| C-STRUCT-7 | graded | **Consistent margins & grid:** fraction of doc frames whose outer margins equal the standard (`space/1600`) and that align to the shared column grid with token-bound section spacing (§2.1). |
| C-STRUCT-8 | **[blocker]** graded | **Auto-layout everywhere:** fraction of doc pages/frames organized with auto-layout (nothing absolute-positioned), per Craft & Measurement. |
| C-STRUCT-9 | binary | The **Icons foundation is present and uses Lucide** — real Lucide icon instances in a grid, not pasted vectors (§6.4). |

### Freshness (sync rule)
| ID | Type | Check |
|---|---|---|
| C-FRESH-1 | **[blocker]** graded | Fraction of doc frames whose recorded `sourceHash` **equals** the current hash of their source slice. A present-but-**stale** frame **fails** (§1.1). |
| C-FRESH-2 | binary | The cover/index `generatedFrom` version matches the manifest's current `system.version`. |

### Color page
| ID | Type | Check |
|---|---|---|
| C-COLOR-1 | binary | The Color frame separates **primitive** and **semantic** regions, both labelled. |
| C-COLOR-2 | **[blocker]** graded | Fraction of swatches whose color chip fill is **bound to a variable** (via `figc bind`), not a hardcoded paint. A hardcoded swatch fails. |
| C-COLOR-3 | graded | Fraction of semantic swatches carrying **both** light and dark values and their **usage** + **alias target**. |

### Typography page
| ID | Type | Check |
|---|---|---|
| C-TYPE-1 | **[blocker]** graded | **One specimen per Figma text style** exists (fraction of text styles with a specimen). |
| C-TYPE-2 | **[blocker]** graded | Fraction of specimens whose sample text is rendered **using the actual text style** (not detached/hand-set type). |
| C-TYPE-3 | binary | A **type-scale overview** is present, in scale order. |
| C-TYPE-4 | graded | Fraction of specimens labelled with all specs: family, size, weight, line-height, letter-spacing. |

### Spacing / Radius / Elevation
| ID | Type | Check |
|---|---|---|
| C-FND-1 | graded | Fraction of space/radius/elevation tokens with a **visual specimen + token name + value** present. |
| C-FND-2 | binary | Elevation frame present **iff** the system defines elevation/shadow tokens. |

### Component doc frames
| ID | Type | Check |
|---|---|---|
| C-CMP-1 | **[blocker]** graded | Fraction of components in the inventory that have a **doc frame** (`figmaNodeId` non-null). |
| C-CMP-2 | **[blocker]** graded | Fraction of component doc frames whose examples are **real instances** — none detached, none resized off their component. |
| C-CMP-3 | graded | Per component: `variantsShown / variantsExpected` — the frame shows **all** variants/states the Contract enumerates. |
| C-CMP-4 | **[blocker]** binary | Every component doc frame has **≥1 Do and ≥1 Don't**, each a correct instance beside an incorrect one. |
| C-CMP-5 | binary | Every component doc frame carries an **anatomy diagram with callouts** and a **usage/props summary** sourced from the Contract. |

### Dogfooding & hygiene
| ID | Type | Check |
|---|---|---|
| C-DOG-1 | graded | Fraction of doc frames whose **own** colors are semantic-token-bound and whose spacing references space tokens (no hardcoded color/magic numbers in the template). |
| C-DOG-2 | binary | Every doc frame carries the footer **freshness stamp** (`generatedFrom` + short `sourceHash`). |

### Score & gate
The canvas docs are **Canvas-conformant** when every **[blocker]** check passes; the **Canvas Documentation Score** is the fraction of all checks passed (graded checks contribute their ratio). `/setup` emits canvas docs that pass all checks; `/extend` must not lower the score (every new component ships its conformant doc frame); `/audit` reports failing check IDs with the offending Figma node id and, for stale frames, the diverging `sourceHash`.

**Gate rule:** a canvas doc set is *non-conformant* — regardless of other scores — if any of these hold: the doc layer is **missing / was skipped** (C-STRUCT-0), a required foundation or component frame is absent (C-STRUCT-2, C-CMP-1), components are **scattered** rather than in the auto-layout grid (C-STRUCT-5), any frames **overlap or are absolute-positioned** (C-STRUCT-6, C-STRUCT-8), or any frame is **stale** (C-FRESH-1). Missing, structurally-wrong, and stale doc layers all actively mislead a designer reading the file — which is exactly what this standard exists to prevent.

---

## 12. Relationship to sibling conventions

- **Component Contract** is the *source* for every component doc frame (§7). This standard does not re-define props, a11y, or Do/Don't — it **projects the Contract onto the canvas**. A component's Contract and its canvas doc frame are two renderings of one record; C-FRESH-1 keeps them in sync.
- **Agent-Readability** owns `ds-manifest.json`; this standard **adds the `canvasDocs` index** (§10) to it, so the canvas docs are discoverable and their freshness is machine-checkable — the same manifest-as-spine principle.
- **`ds-manifest.json`** is the shared source both surfaces render from. The Canvas Documentation Score measures how completely and how *freshly* the Figma canvas mirrors that source — the design-side twin of the Component Contract's web-side checklist.
