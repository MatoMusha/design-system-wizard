# Textarea — build spec

Foundation: shadcn/Radix + Tailwind, 4px grid. Backing: native `<textarea>` (no Radix). APG: native multiline field + `aria-invalid`/`aria-describedby`. Skeleton default — override rules in `README.md`. **Border/focus/invalid recipe must match Input exactly.**

## Variants

| variant | bg | text | border | notes |
|---|---|---|---|---|
| default | transparent | foreground | input | placeholder = muted-foreground |
| invalid | transparent | foreground | destructive | `aria-invalid` → ring destructive/20 (dark /40) |
| disabled | transparent | foreground | input | opacity 50 · cursor-not-allowed · pointer-events none |

## Measurements (px)

| token | dims |
|---|---|
| **box (default)** | min-height 64 · pad-x 12 · pad-y 8 · font 14/400/20 · radius 8 · border 1 · rows≈2 |
| sm (opt) | min-height 48 · pad-x 12 · pad-y 6 · font 14/400/20 · radius 8 · border 1 |
| lg (opt) | min-height 96 · pad-x 12 · pad-y 8 · font 14/400/20 · radius 8 · border 1 |
| focus ring | ring 3 · ring/50 · offset 0 · border→ring |
| error text | font 12/400/16 · gap 6 above · color destructive |
| label | font 14/500/14 · gap 8 below |

## States

- default: `border-input · bg-transparent · text-foreground · placeholder:text-muted-foreground · shadow-xs`
- typing: caret foreground; auto-grows via `field-sizing-content` until min-height
- focus-visible: `outline-none · border-ring · ring-[3px] ring-ring/50 · offset 0`
- invalid: `aria-invalid → border-destructive · ring destructive/20 (dark /40)`
- disabled: `opacity-50 · cursor-not-allowed · pointer-events-none`
- readonly: `border-input · bg-muted/40 (opt) · caret hidden`

## Rules

- min-height 64 (~2 rows) is the floor; never shorter.
- resize: prefer `field-sizing-content` (auto-grow) or `resize-y`; never `resize-x`.
- pad-y stays 8 regardless of content height.
- font 14 (`text-sm`); 16 base on mobile (`md:text-sm`).
- `w-full`; width governed by parent/grid.
- transition `transition-[color,box-shadow] 150ms ease` — never height.
- label above (block), 8px gap, sentence case, no colon; line-height 20 for readable wrapping.

## Tokens consumed (semantic only)

`background` `foreground` `input` `ring` `muted` `muted-foreground` `destructive` `border`

## Overrides (skeleton → change when)

- min-height 64 → 48/96 for `rows` prop or sm/lg density.
- radius 8 → keep in lockstep with Input when the corner token changes.
- `field-sizing-content` → `resize-y` when the browser lacks support / fixed-canvas layout.
- bg-transparent → `bg-muted/40` when readonly or on a colored card surface.

## Figc binding

Auto-layout; bind `padding` → `space/*`, `radius` → `radii/*`, `min-height` → `size/*`, fills/borders → semantic tokens. Verify `figc bound`.
