# Select — build spec

Foundation: shadcn/Radix + Tailwind, 4px grid. Backing: Radix Select (`Root/Trigger/Value/Icon/Portal/Content/Viewport/Item/ItemIndicator/ItemText/Label/Separator/Scroll*`). APG: Combobox/Listbox (single-select, roving focus, type-ahead). Skeleton default — override rules in `README.md`. **Trigger tracks the Input height family.**

## Variants

| variant | bg | text | border | notes |
|---|---|---|---|---|
| default (outline) | transparent (dark input/30) | foreground; placeholder muted-foreground; chevron muted-foreground/50 | input | border 1px, shadow-xs, w-fit |
| invalid | transparent | foreground | destructive | + ring destructive/20 (dark /40) |
| ghost (toolbar/inline) | transparent → hover accent | foreground | none | drop border+shadow; keep px/gap/height |

## Measurements (px)

| token | dims |
|---|---|
| trigger-sm | height 32 · pad-x 12 · pad-y 6 · gap 8 · chevron 16 · font 14/400/20 · radius 8 · border 1 · shadow xs |
| **trigger-default** | height 36 · pad-x 12 · pad-y 8 · gap 8 · chevron 16 · font 14/400/20 · radius 8 · border 1 · shadow xs · w-fit |
| trigger-lg | height 44 · pad-x 16 · gap 8 · chevron 20 · font 16/400/24 · radius 8 · border 1 · shadow xs |
| content (popover) | min-w 128 · pad 4 · radius 8 · border 1 · shadow md · z 50 · max-h `--radix-select-content-available-height` |
| option (item) | height 32 · pad-y 6 · pad-l 8 · pad-r 32 · gap 8 · radius 6 · font 14/400/20 |
| selected-check | box 14 · icon 16 · inset-right 8 · v-center |
| group-label | pad-x 8 · pad-y 6 · font 12/500/16 · muted-foreground |
| separator | height 1 · margin-y 4 · margin-x -4 · bg border |
| scroll-button | pad-y 4 · icon 16 · full-width · center |

## States

- default: `border input · bg transparent · shadow-xs · placeholder muted-foreground`
- open: content `data-[state=open]` animate-in fade-in-0 zoom-in-95 (closed → fade-out/zoom-out-95, 150ms)
- focus-visible: `border→ring · ring 3px · ring-ring/50 · offset 0 · outline none`
- placeholder: `data-[placeholder]:text-muted-foreground`
- item-highlighted (focus): `bg accent · text accent-foreground`
- item-selected: ItemIndicator Check 16 in 14px box at inset-right 8 (currentColor)
- item-disabled: opacity 50 · pointer-events none
- disabled: opacity 50 · cursor-not-allowed
- invalid: `border destructive · ring destructive/20 (dark /40)`

## Rules

- trigger value `nowrap` + `line-clamp-1`; trigger `w-fit` (`min-w-0`) — set `w-full` at call site for field-width.
- chevron = Lucide ChevronDown 16, opacity-50, currentColor.
- trigger height data-driven (`data-[size=default]:h-9 / sm:h-8`; add `lg:h-11` for 44).
- content `min-w-[8rem]`; `position=popper` matches trigger width via `--radix-select-trigger-width`, offset 4/side.
- option reserves pad-r 32 for the check (absolutely `right-2`, never reflows text).
- row highlight uses **focus** (Radix roving), not hover — style `focus:bg-accent`.
- group label `text-xs muted-foreground` non-interactive; separator 1px `bg-border` with `-mx-1` bleed.
- transition `transition-[color,box-shadow] 150ms`; content zoom+fade on enter/exit.
- whole trigger is the hit target — ensure ≥44px effective touch area at sm/default via wrapper padding on touch.

## Tokens consumed (semantic only)

`input` `border` `ring` `background` `foreground` `muted-foreground` `popover` `popover-foreground` `accent` `accent-foreground` `destructive` `radius`

## Overrides (skeleton → change when)

- trigger-lg (h44/pad16/icon20/font16) → comfortable density / primary form fields (not shipped by shadcn — extend size union).
- trigger `w-full` → labeled form field (not an inline/toolbar picker).
- option radius `rounded-none` + content `rounded-md` → square/low-corner brand.
- remove border+shadow, add `hover:bg-accent` → ghost/toolbar variant.
- content `position=popper` + offset → trigger-width-matching dropdown.
- dark `bg-input/30` + `hover:bg-input/50` → dark theme fill.

## Figc binding

Trigger + content + option as components; bind all pad/gap/radius/height → tokens, fills/borders → semantic tokens. Verify `figc bound`.
