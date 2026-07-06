# Checkbox — build spec

Foundation: shadcn/Radix + Tailwind, 4px grid. Backing: Radix Checkbox (`Root` + `Indicator`). APG: Checkbox (tri-state). Skeleton default — override rules in `README.md`. **Intrinsic 16px box; not the control-height family.**

## Variants

| variant | bg | border | notes |
|---|---|---|---|
| unchecked (default) | background (dark input/30) | input | empty box, no indicator |
| checked | primary | primary | Lucide Check, text primary-foreground |
| indeterminate | primary | primary | Lucide Minus via `data-[state=indeterminate]` |
| invalid | background | destructive | ring destructive/20 (dark /40); overlays any check state |

## Measurements (px)

| token | dims |
|---|---|
| **box** | size 16 · radius 4 · border 1 · shadow-xs · hit-area 44 |
| check-icon | icon 14 · stroke 2 · Lucide Check |
| indeterminate-icon | icon 14 · stroke 2 · Lucide Minus |
| label | font 14/500/20 · gap 8 |
| focus-ring | ring 3 · offset 0 |

## States

- default: `border-input · bg-background (dark bg-input/30) · shadow-xs`
- focus-visible: `border-ring · ring-ring/50 · ring-[3px] · ring-offset-0 · outline-none`
- checked: `bg-primary · text-primary-foreground · border-primary · show Check 14`
- indeterminate: `bg-primary · text-primary-foreground · border-primary · show Minus 14`
- invalid: `border-destructive · ring-destructive/20 (dark /40)`
- disabled: `opacity-50 · cursor-not-allowed · pointer-events-none`

## Rules

- box `size-4` (16), `shrink-0`, `rounded-[4px]`, 1px border.
- checked/indeterminate bg = primary; indicator = `text-current` inheriting primary-foreground.
- indicator icons Lucide Check / Minus at `size-3.5` (14), stroke 2; toggle via `data-[state=...]`.
- focus recipe: `focus-visible:border-ring + ring-ring/50 + ring-[3px]`, offset 0, `outline-none`.
- wrap in label `flex items-center gap-2` (8) for ≥44×44 hit area + click-to-toggle.
- label `text-sm/500`, sentence case, single line (`items-start` if wrapping).
- invalid via `aria-invalid`; check color stays primary-foreground (#fff) — no destructive-foreground token.
- transition `all 150ms ease`; disabled sets opacity-50 + pointer-events-none on box and label.

## Tokens consumed (semantic only)

`background` `foreground` `primary` `primary-foreground` `border` `input` `ring` `destructive`

## Overrides (skeleton → change when)

- gap 8 → `items-start gap-3` (12) for a multiline label.
- radius 4 → keep at `rounded-[4px]` (sub-input scale) even when global radius rises — never full 8.
- box 16 → keep visual 16 but enforce 44 hit-area via label padding; or scale to `size-5` (20) for Material parity (icon → 16).
- invalid ring → drop for validation-off forms (keep border only).

## Figc binding

Box component; bind size → `size/*`, radius → `radii/*`, gap → `space/*`, fills/border → semantic tokens. Verify `figc bound`.
