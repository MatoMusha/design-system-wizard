# Separator — build spec

Foundation: shadcn/Radix + Tailwind. Backing: Radix `SeparatorPrimitive.Root` → plain div. APG: Separator (`role="separator"` / `role="none"` when decorative). Skeleton default — override rules in `README.md`. **Thickness always 1px; non-interactive.**

## Variants

| variant | color | notes |
|---|---|---|
| default (standalone) | border | 1px line; thickness + color fixed — the only real style variant |
| in-menu (dropdown/command/popover) | border | same 1px line with -x bleed + vertical margin to span a padded container (some systems tint muted — keep border for skeleton consistency) |

## Measurements (px)

| token | dims |
|---|---|
| horizontal | thickness 1 (`h-px`) · width 100% (`w-full`) · margin 0 · shrink-0 |
| vertical | thickness 1 (`w-px`) · height 100% (`h-full`) · margin 0 · shrink-0 |
| in-menu (horizontal) | thickness 1 · margin-y 4 (`my-1`) · margin-x -4 bleed (`-mx-1`) · width `calc(100%+8)` |
| inset (list/leading-icon align) | thickness 1 · inset-left 16 (or 40 past leading icon) · width `100%−inset` |

## States

- default: `bg border · thickness 1px` · decorative, non-interactive
- hover/active/focus/disabled: none — decorative element, not focusable
- semantic (`decorative=false`): `role=separator` + `aria-orientation`; exposed to AT as a boundary; still no focus/hover

## Rules

- wrap Radix `SeparatorPrimitive.Root`; default `orientation=horizontal decorative=true`.
- exact class: `bg-border shrink-0 data-[orientation=horizontal]:h-px data-[orientation=horizontal]:w-full data-[orientation=vertical]:h-full data-[orientation=vertical]:w-px`.
- thickness always 1px — never scale with control-height family; render via `h-px`/`w-px`, not `border`.
- color always token `border` — never hardcode, never muted/foreground for a standalone rule.
- orientation drives axis via `data-[orientation]` variants, not conditional JS.
- `shrink-0` so flex parents never collapse the 1px axis; vertical needs a sized parent (`h-full`).
- no radius/shadow/transition — static line.
- `decorative=true` → `role=none` (hidden from AT); set `decorative=false` only when the divide is semantically meaningful.
- standalone carries zero margin — spacing is the parent's job; menu/command contexts add `my-1 + -mx-1` bleed. Never receives tabindex.

## Tokens consumed (semantic only)

`border`

## Overrides (skeleton → change when)

- color border → muted for a fainter divider inside popover/menu surfaces (keep border unless system-wide).
- margin 0 → `my-1` (4) + `-mx-1` bleed inside a `p-1` padded menu/command/select group.
- inset-left 0 → 16/40 to align to the text column when list rows have leading icons/avatars.
- thickness 1 → 2px+ / gradient / dashed for a heavier section break (departs from skeleton — treat as a distinct token).
- `decorative:true → false + role=separator` when the boundary conveys structure to AT.

## Figc binding

1px line; bind fill → `border` token; margins via `space/*` in menu context. Verify `figc bound`.
