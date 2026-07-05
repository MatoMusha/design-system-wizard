# The Craft & Measurement Standard

**Status:** Normative standard for `design-system-wizard`
**Applies to:** every foundation and every component in a design system managed by this plugin
**Consumed by:** `/setup` (builds *to* it), `/extend` (conforms new work *to* it), `/audit` (scores *against* it via the **Craft Checklist**, §7)
**Related conventions:** [Component Contract](./component-contract.md) · [Canvas Documentation](./canvas-documentation.md) · [Architecture — token spine](../ARCHITECTURE.md)
**Owned by:** `figc-operator` (mechanics) and `craft-reviewer` (the verification loop, §6)

---

## 0. Why this standard exists

A system can pass every structural checklist — a conformant Component Contract, a complete set of canvas doc frames, string-identical token parity — and still be **bad**: bland, generic, off-grid, sloppy, components that overlap and spacing that never lands on a rhythm. Those other conventions verify that the documentation is *well-formed*. This one verifies that the design is *well-made*.

The core insight: **quality is not generated, it is inherited and then enforced.** Two mechanisms produce it, and both are mandatory.

1. **Reference-grounding (§1–§2)** — the system's foundations are *derived* from proven, battle-tested open primitives (Radix Colors, the Tailwind scale, shadcn's semantic contract, Lucide icons), never hand-invented. This makes the system professional *by construction*, before a single component is drawn.
2. **The craft-verification loop (§5–§6)** — every built node is screenshotted, inspected against a binary checklist, corrected, and re-shot until it passes. Taste emerges from critique-and-refine, not from one-shot generation.

Grounding sets the floor; the loop closes the gap between "looks like a design system" and "is a good one."

---

## 1. Inherit taste, don't invent it (reference-grounding)

**The single biggest lever for quality.** Every foundation axis is built on a specific, named, open technical foundation. The plugin does not hand-mix values and hope they cohere. It maps proven primitives onto the system's own two-tier token spine ([Architecture §4](../ARCHITECTURE.md)). Grounding a system in **Radix + Tailwind + shadcn + Lucide** makes it coherent and professional the moment it is created.

> These are named as the **technical foundations to build on**, not as a visual style to copy. The plugin derives structure (perceptual scales, grid steps, semantic slots) from them; the brand's own color, tone, and identity are layered on top during `/setup`.

### 1.1 Color — derive from Radix Colors (or shadcn defaults)

Color primitives are **never hand-mixed hex ramps.** Base them on one of two grounded sources:

- **Radix Colors** — the 12-step perceptual scales, with light **and** dark built in. The steps carry fixed semantics; the plugin maps them onto shadcn's semantic contract rather than eyeballing which step is a border:

  | Steps | Role | Maps toward |
  |---|---|---|
  | 1–2 | App background, subtle background | `--background`, subtle surfaces |
  | 3–5 | Component backgrounds (normal, hover, active) | `--card`, `--muted`, `--secondary` |
  | 6–8 | Borders and separators (subtle, UI, hovered) | `--border`, `--input` |
  | 9–10 | Solid accents (base, hover) | `--primary`, `--ring` |
  | 11–12 | Text (low-contrast, high-contrast) | `--muted-foreground`, `--foreground` |

- **shadcn's own default oklch theme values** — the ready-made `--background`/`--foreground`/`--primary`/… set, used verbatim as the primitive layer.

Either way the semantic tier aliases these primitives to the exact identifiers shadcn hard-codes (`--background`, `--primary`, `--primary-foreground`, `--secondary`, `--muted`, `--accent`, `--destructive`, `--border`, `--input`, `--ring`), per the token spine. **NEVER hand-mix a hex ramp.** A scale a designer typed out step by step is off-limits; the scale must trace to Radix (or the shadcn defaults).

### 1.2 Spacing — a strict 4px base grid, Tailwind-aligned

One base unit: **4px.** The spacing scale is exactly:

```
0 · 2 · 4 · 8 · 12 · 16 · 20 · 24 · 32 · 40 · 48 · 64 · 80 · 96   (px)
```

Every measurement in the entire system — every gap, every padding, every fixed size — is one of these tokens, i.e. a multiple of the base unit (the single sub-grid exception, `2px`, exists only for hairline insets and icon optical nudges). This scale mirrors Tailwind's spacing steps so the exported utilities line up. **No arbitrary pixel values, ever.**

### 1.3 Radius — a small, consistent scale

```
2 · 4 · 6 · 8 · 12 · 16 · full   (px; "full" = pill/round)
```

A single radius token is assigned per component role (§3), so every control of a given role shares a corner. shadcn's `--radius` maps onto this scale.

### 1.4 Elevation / shadow — a 5-step subtle, layered scale

Five elevation tokens, each a **subtle, layered** shadow (a soft ambient layer plus a tighter directional layer), rising in the order:

```
elevation/0 (none) · elevation/1 (raised) · elevation/2 (overlay) ·
elevation/3 (popover) · elevation/4 (modal)
```

Shadows are low-opacity and layered — never one hard, heavy drop shadow. In dark mode, elevation reads through surface lightening as well as shadow.

### 1.5 Type — a modular scale on a professional typeface

Typeface is **Inter** or **Geist**, built as **real Figma text styles** (not raw variables — per the token spine). Sizes are Tailwind-aligned:

```
12 · 14 · 16 · 18 · 20 · 24 · 30 · 36 · 48   (px)
```

with **correct** line-height and letter-spacing per role — the two things a hand-generated scale always gets wrong:

| Role | Size (px) | Line-height | Letter-spacing | Weight |
|---|---|---|---|---|
| Display | 48 | 1.1 | −0.02em | 600–700 |
| H1 | 36 | 1.15 | −0.02em | 600–700 |
| H2 | 30 | 1.2 | −0.01em | 600 |
| H3 | 24 | 1.25 | −0.01em | 600 |
| H4 | 20 | 1.3 | 0 | 600 |
| Body large | 18 | 1.5 | 0 | 400 |
| Body | 16 | 1.5 | 0 | 400 |
| Body small | 14 | 1.5 | 0 | 400 |
| Caption | 12 | 1.4 | 0 | 400–500 |

Rules baked in: **tight line-heights (~1.1–1.25) on large headings, ~1.5 on body**; **slightly negative letter-spacing on large display, 0 on body**; weights drawn only from **400 / 500 / 600 / 700**. Every specimen in the Typography canvas doc renders from the actual text style ([Canvas Documentation §5](./canvas-documentation.md)), so the scale on the page is the scale in the file.

---

## 2. Icon system — Lucide (mandatory)

Icons are **Lucide**. No hand-drawn icons, ever — a bespoke vector in an icon slot is a hard failure (§7, CR-ICON-1).

- **Default size 24px**, with **16px** and **20px** as the smaller on-grid options. Icon sizes are spacing tokens like everything else.
- **Consistent stroke ~1.5–2px** across the whole set (Lucide's native stroke), never mixed weights.
- Icons are **sized on the 4px grid** and use **`currentColor`** so they inherit the token color of their surrounding text/context — never a hardcoded fill.
- **In Figma:** import Lucide icons as SVG — `figc-operator` runs `figma.createNodeFromSvg` on the Lucide SVG source — into an **Icons** foundation. Placed as instances, never pasted-and-detached vectors ([Canvas Documentation §6.4](./canvas-documentation.md)).
- **In code:** use `lucide-react`.

---

## 3. Pixel-perfect spatial discipline (mechanical, enforceable)

This section is the direct fix for the sloppy, overlapping, off-grid output. It is deliberately mechanical — every rule below is binary and machine- or visually-checkable (§7).

### 3.1 Everything on the grid

- **Base unit 4px.** Every gap, padding, and fixed dimension **equals a spacing token** (§1.2). No arbitrary px. **No manual nudging** — a designer dragging a node two pixels to make it "look right" is forbidden; fix the token or the layout, not the position.

### 3.2 Auto-layout only — nothing overlaps

- **Every frame and every component uses Figma auto-layout.** Gap and padding are **token-bound** (`figc bind` to `space/*`). Layout sizing is **hug** or **fill** — never a hand-set width/height that fights the content.
- **Nothing is absolute-positioned. Nothing overlaps.** Absolute positioning inside a component (the usual source of overlapping nodes and drift) is banned. If two elements need to stack, they stack in an auto-layout parent, not by coordinate. This single rule eliminates the overlap class of defects entirely.

### 3.3 Consistency per role

- **Snap to grid** — every node's position and size resolve to grid multiples.
- **Consistent internal padding per component role** — all controls of a role (button, input, card, menu item) share the same padding tokens; a button is not `12/16` in one place and `10/14` in another.
- **Consistent border widths** — one hairline token (typically 1px) used system-wide, bound to `--border`.
- **Consistent corner radius per role** — each role picks one radius token from §1.3 and every instance of that role uses it.

### 3.4 Optical alignment

- **Icon + text pairs are optically aligned** — centered on the shared axis (or baseline-aligned where the pattern calls for it), not merely box-aligned, so the icon reads as sitting *with* the label.
- **Icon-to-label gaps are consistent** — one gap token (commonly `space/8`) for every icon+text pairing across the system.

---

## 4. Restraint heuristics (taste)

Grounding gives good raw materials; restraint keeps them coherent. The defaults:

- **Neutral base + exactly ONE accent.** The bulk of the UI is the neutral scale; a single accent (`--primary`) carries emphasis. Not three competing brand colors.
- **Limited type sizes.** Use a handful of the scale's steps, not all nine at once. Hierarchy comes from a few well-separated sizes plus weight, not from many near-identical sizes.
- **Generous whitespace.** Prefer the larger spacing token when unsure; crowding reads as unfinished.
- **Consistent radii and subtle elevation.** One radius per role; shadows stay in the subtle 5-step scale.
- **Strong typographic hierarchy with few sizes** — clear jumps between levels beat a smooth ramp of many.

> **Rule of thumb:** *when in doubt — remove, align, or replace a value with a token.* Never "add a bit more" or "nudge until it looks right."

---

## 5. The craft-verification loop

**Building is not done at first render.** A one-shot generation is a draft, not a deliverable. Every build passes through this loop before it is considered complete:

```
build ──► figc shot the node ──► inspect the screenshot ──► fix ──► re-shot ──► …repeat until it passes §7
```

Inspect each screenshot for, at minimum:

- **overlaps** — any two nodes intersecting;
- **misalignment** — edges/centers that don't line up, icon+text off-axis;
- **off-grid spacing** — gaps/paddings that aren't spacing tokens;
- **inconsistent radii** — a role using more than one corner value;
- **weak hierarchy** — levels that don't read as distinct;
- **poor contrast** — text/background pairs below WCAG AA.

Fix what you find, re-shoot, and look again. **Taste emerges from this critique-and-refine cycle, not from the first output.** The loop is mandatory on `/setup` (foundations + base components), on `/extend` (each new component), and during `/audit` fixes.

The loop reuses the plugin-wide figc convention — *shot-verify every Figma write* — and turns it into an explicit, checklist-gated iteration rather than a single glance.

---

## 6. `craft-reviewer` — the agent that owns the loop

The **`craft-reviewer`** agent owns §5. It does not build; it critiques. Given a built node it:

1. drives `figc-operator` to `figc shot` the node (and its states/variants);
2. runs the **Craft Checklist** (§7) against the screenshot **and** the node tree (auto-layout flags, bound tokens, radii, text styles — read via figc);
3. returns a **PASS/FAIL per check** with the offending node id and a concrete correction;
4. loops with the builder until every blocker passes.

`craft-reviewer` is invoked by `/setup` and `/extend` after every build step, and by `/audit` (alongside `rubric-evaluator`) to score existing work. It is the craft twin of `rubric-evaluator`: where `rubric-evaluator` scores documentation conformance, `craft-reviewer` scores *how well-made the pixels are*.

---

## 7. The Craft Checklist

`/audit` and `craft-reviewer` enforce these. **Each is binary (PASS / FAIL)** and checkable either visually from a `figc shot` or mechanically from the node tree / token bindings. Checks marked **[blocker]** must pass for a node to be considered craft-conformant at all.

### Layout & positioning
| ID | Check |
|---|---|
| CR-LAYOUT-1 | **[blocker]** No two sibling nodes overlap (no intersecting bounding boxes). |
| CR-LAYOUT-2 | **[blocker]** Every frame and component uses **auto-layout**; **no node is absolute-positioned** within a component. |
| CR-LAYOUT-3 | Layout sizing is **hug** or **fill** — no hand-set fixed dimension fighting content. |
| CR-LAYOUT-4 | Every node's position and size resolve to **4px grid multiples** (no off-grid coordinates; no manual nudge). |

### Spacing
| ID | Check |
|---|---|
| CR-SPACE-1 | **[blocker]** Every gap and padding value is a **spacing token** from the §1.2 scale (a grid multiple) — no arbitrary px. |
| CR-SPACE-2 | Auto-layout gap/padding are **token-bound** (`figc bind` to `space/*`), not typed literals. |
| CR-SPACE-3 | Internal padding is **consistent per component role**; icon-to-label gap is one shared token. |

### Color & tokens
| ID | Check |
|---|---|
| CR-COLOR-1 | **[blocker]** Every fill and stroke is **token-bound** — no hardcoded hex/`rgb()`/`hsl()`/named color anywhere. |
| CR-COLOR-2 | **[blocker]** Color primitives **derive from Radix Colors** (12-step scales) **or the shadcn default oklch values** — not a hand-mixed hex ramp. |
| CR-COLOR-3 | Semantic tokens map to shadcn's fixed identifiers (`--background`, `--primary`, `--border`, `--ring`, …). |

### Radius & elevation
| ID | Check |
|---|---|
| CR-RADIUS-1 | **[blocker]** Every corner radius is a value from the §1.3 radius scale; each component **role uses exactly one** radius token. |
| CR-ELEV-1 | Every shadow is one of the 5 elevation tokens (subtle, layered) — no ad-hoc heavy drop shadows. |

### Typography
| ID | Check |
|---|---|
| CR-TYPE-1 | **[blocker]** All text uses a **real Figma text style** from the §1.5 scale — no detached/hand-set family, size, or weight. |
| CR-TYPE-2 | Sizes come only from the §1.5 set; weights only from 400/500/600/700. |
| CR-TYPE-3 | Line-height matches role (~1.1–1.25 for large headings, ~1.5 for body); large display carries slightly negative letter-spacing. |
| CR-TYPE-4 | Typographic hierarchy reads as distinct levels (few, well-separated sizes). |

### Icons
| ID | Check |
|---|---|
| CR-ICON-1 | **[blocker]** Every icon is a **Lucide** icon (no hand-drawn vectors). |
| CR-ICON-2 | Icons are sized on-grid (24 default; 16/20), consistent ~1.5–2px stroke, and use `currentColor` (inherit the token color). |

### Alignment & optics
| ID | Check |
|---|---|
| CR-ALIGN-1 | **[blocker]** No misaligned edges/centers; icon+text pairs are **optically aligned** on a shared axis/baseline. |

### Restraint
| ID | Check |
|---|---|
| CR-REST-1 | Neutral base + **one** accent (no competing brand colors). |
| CR-REST-2 | Whitespace is generous; layout is not crowded. |

### Contrast
| ID | Check |
|---|---|
| CR-A11Y-1 | **[blocker]** Every text/background and essential UI pairing meets **WCAG AA** contrast (4.5:1 body text, 3:1 large text and UI components), in **both** light and dark modes. |

### Score & gate
A node is **craft-conformant** when every **[blocker]** check passes; its **Craft Score** is the fraction of all checks passed. `/setup` builds nodes that pass all checks; `/extend` must not lower a system's Craft Score (every new component ships craft-conformant); `/audit` reports failing check IDs with the offending Figma node id and a concrete fix. Because the blockers include *no overlaps*, *auto-layout only*, *on-grid spacing*, and *token-bound color*, a node that reproduces the current sloppy output cannot score above **non-conformant** — which is exactly what this standard exists to prevent.

---

## 8. Relationship to sibling conventions

- **Component Contract** governs whether a component's *documentation* is well-formed (props, a11y, examples-as-data). This standard governs whether the component itself is *well-made*. A component must pass both: a perfect Contract on an overlapping, off-grid component still fails `/audit`.
- **Canvas Documentation** renders foundations and components onto the Figma canvas; those doc frames are themselves subject to this standard (they dogfood the system's own grid, tokens, and text styles). The Color/Typography/Spacing/Radius/Elevation/Icon foundations this standard grounds are exactly the foundations that convention documents.
- **Token spine ([Architecture §4](../ARCHITECTURE.md))** is where the grounded primitives (Radix/shadcn color, Tailwind spacing, the type styles) live and flow one-way into code. This standard decides *what those primitives are*; the spine carries them, string-identical, to CSS variables and shadcn.
