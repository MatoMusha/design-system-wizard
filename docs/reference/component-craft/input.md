# Input — build spec

Foundation: shadcn/Radix + Tailwind, 4px grid. Backing: native `<input>` (`data-slot="input"`, no Radix). APG: none — pair with `<label htmlFor>` + `aria-invalid`/`aria-describedby`. Skeleton default — override rules in `README.md`.

## Variants

| variant | bg | text | border | notes |
|---|---|---|---|---|
| default | transparent (dark input/30) | foreground | input | placeholder = muted-foreground; selection bg primary / text primary-foreground |
| invalid | transparent | foreground | destructive | `aria-invalid` → ring destructive/20 (dark /40) |
| disabled | transparent | foreground @50% | input @50% | opacity 50 · pointer-events none · cursor-not-allowed |
| file | transparent | foreground | input / 0 (file button) | file:// pseudo-button borderless, transparent, font-medium |

## Measurements (px)

| token | dims |
|---|---|
| sm | height 32 · pad-x 12 · font 14/400/20 · border 1 · radius 8 · icon 16 |
| **default** | height 36 · pad-x 12 · pad-y 4 · font 14/400/20 · border 1 · radius 8 · icon 16 · shadow xs |
| lg | height 44 · pad-x 14 · font 16/400/24 · border 1 · radius 8 · icon 20 |
| file-button | height 28 · border 0 · font 14/500/20 · bg transparent · text foreground |
| icon-slot (lead/trail) | icon 16 · inset-x 12 · gap 8 · pad-that-side 36 (pad 12 + icon 16 + gap 8) |

## States

- default: `border-input · bg-transparent · text-foreground · shadow-xs · placeholder:text-muted-foreground`
- hover: none (border unchanged); cursor text
- typing: caret foreground · `selection:bg-primary selection:text-primary-foreground`
- focus-visible: `border→ring · ring 3px ring/50 · offset 0 · outline-none`
- invalid: `aria-invalid → border-destructive · ring destructive/20 (dark /40)`
- disabled: `opacity-50 · pointer-events-none · cursor-not-allowed`

## Rules

- `w-full min-w-0`; constrain width via wrapper/`max-w`, never fixed inline width.
- radius = `rounded-md` = `--radius − 2` = 8; keep 8 across all sizes.
- border always 1px `input` (except file:// pseudo-button).
- placeholder = muted-foreground, never foreground.
- font `text-base` (16) mobile / `md:text-sm` (14) desktop to prevent iOS zoom; weight 400.
- invalid is state-driven via `aria-invalid`, not a prop; error text is a sibling linked by `aria-describedby`.
- lead/trail icons: Lucide, currentColor, stroke 1.75, absolute-positioned; add matching pad-x so text clears the icon. Decorative icons `aria-hidden`; a clickable adornment gets ≥44×44 hit area.
- transition `transition-[color,box-shadow] 150ms ease` (never height/layout).

## Tokens consumed (semantic only)

`input` `foreground` `muted-foreground` `ring` `destructive` `primary` `primary-foreground` `background` `radius`

## Overrides (skeleton → change when)

- height family → compact density drops a step (sm 28 / default 32 / lg 36), keep 4px grid.
- radius 8 → tracks `--radius` (`radius − 2`), so corner theme shifts input automatically.
- bg-transparent → filled/subtle variant = `bg-muted` + `border-transparent`, same heights/radius.
- pad-x 12 → the icon side becomes 36 when a lead/trail icon is present.
- `md:text-sm` → desktop-only product with no mobile-zoom concern → `text-sm` at all breakpoints.

## Figc binding

Auto-layout; bind `padding` → `space/*`, `radius` → `radii/*`, `height` → `size/element-*`, fills/borders → semantic tokens above. Verify `figc bound` — no raw numbers/hex.
