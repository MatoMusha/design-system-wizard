# Component Craft Catalog

Per-component **build specs** — exact measurements, variant matrices, state recipes.
The grounded default (`skeleton`) that `/setup` and `/extend` build to. Cross-system
research reconciled to a shadcn/Radix + Tailwind + 4px-grid foundation.

## Standing rules

1. **Spec-only.** Every entry is numbers, tokens, and key:value rules. No prose,
   no rationale except one-clause build rules.
2. **Skeleton, not law.** These are defaults. The `/setup` brand interview and
   `/extend` context-of-use interview **override** any value per project need.
   Each spec carries an `Overrides` block naming what changes and when.
3. **Semantic tokens only.** Every value is a semantic token or a number bound to
   one — never a raw hex, never a primitive reference, never an ad-hoc pixel in code.
4. **Parity.** Token names in a spec are the exact identifiers that survive
   Figma → DTCG → CSS → Tailwind/React (see `../shadcn-skeleton.md`).

## Shared primitives (referenced by every spec)

- **Grid:** 4px. Every gap/padding/size is a multiple, bound to a `space/*` token.
- **Control-height family** (`size/element-*`, overridable — see below):
  `sm 32 · default 36 · lg 44 · xs 24`.
- **Focus-visible recipe (universal):** `ring 3px · ring/50 · offset 0 · border→ring`.
- **Touch target:** interactive controls expose a **≥44×44px hit area**
  (`::before` expander under `@media (pointer:coarse)`) regardless of visual height.
- **Disabled:** `opacity 50% · pointer-events none`. **Transition:** `all 150ms ease`
  (none on `:active`).

### Height-family & radius overrides (interview-driven)

| interview answer | height family (sm/default/lg) | default radius |
|---|---|---|
| density = **comfortable** (default) | 32 / 36 / 44 | 8 |
| density = **compact** | 28 / 32 / 40 | 6 |
| density = **spacious** | 36 / 40 / 48 | 8 |
| corner = **sharp** | — | 4 |
| corner = **subtle** (default) | — | 8 |
| corner = **rounded/pill** | — | full (height-based) |

## Catalog

**Action** · [`button.md`](./button.md)
**Data input** · [`input.md`](./input.md) · [`textarea.md`](./textarea.md) · [`label.md`](./label.md) · [`select.md`](./select.md) · [`checkbox.md`](./checkbox.md) · [`radio-group.md`](./radio-group.md) · [`switch.md`](./switch.md)
**Container** · [`card.md`](./card.md) · [`separator.md`](./separator.md)
**Feedback** · [`badge.md`](./badge.md) · [`alert.md`](./alert.md) · [`tooltip.md`](./tooltip.md)
**Navigation** · [`tabs.md`](./tabs.md)
**Overlay** · [`dialog.md`](./dialog.md)
**Media** · [`avatar.md`](./avatar.md)

16 core components. All specs anchored to the locked skeleton (control-height family
sm 32 / default 36 / lg 44, radius 8, universal 3px focus ring, 4px grid, Lucide
icons, ≥44px touch target) and cross-checked against shadcn/Radix, Material 3,
Polaris, Primer, Apple HIG, Tailwind, and Ant.
