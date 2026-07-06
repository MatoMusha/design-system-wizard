# Radio Group — build spec

Foundation: shadcn/Radix + Tailwind, 4px grid. Backing: Radix RadioGroup (`Root/Item/Indicator`) + Lucide Circle dot. APG: Radio Group. Skeleton default — override rules in `README.md`. **Intrinsic circle; radius always full.**

Anatomy: Root (`role=radiogroup`, vertical grid) · Item (outer circle) · Indicator (centered wrapper) · Dot (Circle, fill-primary) · Label (`htmlFor`).

## Variants

| variant | bg | border | notes |
|---|---|---|---|
| default | transparent (dark input/30) | input | unchecked = empty ring; checked reveals 8px primary dot |
| invalid | transparent | destructive | `aria-invalid` → border-destructive + ring destructive/20 (dark /40) |

## Measurements (px)

| token | dims |
|---|---|
| **control (outer circle)** | box 16 · border 1 · radius full · shadow xs · hit-area 44 · shrink-0 |
| dot (inner) | box 8 · fill primary · radius full · centered abs-translate |
| group | grid · item-gap 12 (vertical) |
| label | gap-to-control 8 · font 14/500/20 · align center |
| focus-ring | ring 3 · offset 0 · color ring/50 |

## States

- default: `border-input · bg transparent (dark bg-input/30) · dot hidden`
- checked: dot 8px `fill-primary` visible; border-input retained
- focus-visible: `border→ring · ring 3px ring/50 · offset 0`
- disabled: opacity 50 · pointer-events none · cursor-not-allowed
- invalid: `aria-invalid → border-destructive · ring destructive/20 (dark /40)`

## Rules

- `role=radiogroup`; Radix roving tabindex + arrow-key selection, one dot per group.
- wrap each Item+Label in `flex items-center gap-2`; Label `htmlFor`→Item `id`.
- control 16px but expose ≥44×44 hit area via label/padding wrapper.
- dot = Lucide Circle with `fill-primary` (filled, not stroked); center with `absolute top/left 1/2 -translate-x/y-1/2`.
- `shrink-0` on control so flex never squashes the circle.
- radius always full (intrinsic) — never the 8px corner.
- transition `transition-[color,box-shadow] 150ms ease`; label sentence case, single line, cursor pointer.

## Tokens consumed (semantic only)

`primary` `input` `ring` `destructive` `foreground` `background`

## Overrides (skeleton → change when)

- item-gap 12 → 8 (`gap-2`) for compact density.
- control 16 / dot 8 → 20 / 10 (2px stroke) for Material-3 parity.
- border → destructive + ring destructive/20 when `aria-invalid` on Item.
- label gap 8 → grows for card/list-row radio where the full row is the hit target.

## Figc binding

Circle + dot components; bind size → `size/*`, gap → `space/*`, fills/border → semantic tokens. Verify `figc bound`.
