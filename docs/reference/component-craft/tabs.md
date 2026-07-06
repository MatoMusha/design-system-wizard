# Tabs — build spec

Foundation: shadcn/Radix + Tailwind, 4px grid. Backing: Radix Tabs (`Root/List/Trigger/Content`). APG: Tabs (automatic activation). Skeleton default — override rules in `README.md`. **List tracks the control-height family.**

## Variants

| variant | list bg | active | inactive text | notes |
|---|---|---|---|---|
| default (segmented) | muted | trigger bg-background + shadow-sm | foreground/60 (dark muted-foreground) | triggers `flex-1` equal-width; radius list 8 / trigger 6 |
| line (underline) | transparent | 2px foreground underline, no bg/shadow | foreground/60 (dark muted-foreground) | list `rounded-none`, gap 4; triggers content-width |

## Measurements (px)

| token | dims |
|---|---|
| list / default | height 36 · pad 3 · gap 0 · radius 8 · bg muted |
| list / line | height 36 · pad 3 · gap 4 · radius 0 · bg transparent |
| list / sm | height 32 · pad 3 · radius 8 |
| list / lg | height 44 · pad 3 · radius 8 |
| **trigger** | height `calc(100%−1px)`≈35 · pad-x 8 · pad-y 4 · gap 6 · icon 16 · font 14/500/20 · radius 6 · border 1 |
| underline-indicator (line) | height 2 · offset-bottom -5 · inset-x 0 · bg foreground |
| content | margin-top 8 · pad 0 · flex-1 |
| root | gap 8 · flex-col (horizontal) |

## States

- default: `text foreground/60 (dark muted-foreground) · border transparent · bg transparent`
- hover: `text foreground` (no bg change)
- active / default: `bg background · text foreground · shadow-sm · border transparent` (dark: border input, bg input/30)
- active / line: `bg transparent · text foreground · shadow-none · underline 2px foreground`
- focus-visible: `border→ring · ring 3px · ring/50 · outline-0 (offset 0)`
- disabled: opacity 50 · pointer-events none

## Rules

- labels `whitespace-nowrap`; never wrap a trigger.
- default variant: triggers `flex-1` (equal-width segmentation); line variant: intrinsic width.
- icons Lucide currentColor `size-16 shrink-0 pointer-events-none`.
- nested radius: list `rounded-md` 8, trigger `rounded-sm` 6 (line variant both `rounded-none`).
- transition `transition-all 150ms ease`; content spacing via root `flex gap 8` (not trigger margin).
- coarse/touch pointer: promote list to lg (44) for the hit target.
- automatic activation: arrow keys move + select; roving tabindex. Sentence-case labels, no all-caps.

## Tokens consumed (semantic only)

`background` `foreground` `muted` `muted-foreground` `border` `input` `ring`

## Overrides (skeleton → change when)

- list height → density sm 32 / default 36 / lg 44.
- list+trigger radius → line variant `rounded-none` + 2px underline; default `rounded-md 8 / rounded-sm 6`.
- active treatment → default (bg-background + shadow-sm) vs line (foreground underline only).
- trigger width → default `flex-1`; line content-width.
- hit area → bump list to lg 44 on coarse pointer.

## Figc binding

List + trigger components; bind height → `size/element-*`, pad/gap → `space/*`, radius → `radii/*`, fills → semantic tokens. Verify `figc bound`.
