# Alert — build spec

Foundation: shadcn/Radix + Tailwind, 4px grid. Backing: native `div role="alert"` (no Radix). APG: Alert (`role=alert`, `aria-live=assertive`, atomic). Skeleton default — override rules in `README.md`. **Static, non-interactive — no hover/focus/disabled.**

## Variants

| variant | bg | text | border | notes |
|---|---|---|---|---|
| default | card | card-foreground | border | leading icon `text-current`; description muted-foreground |
| destructive | card | destructive | border | title+icon text-destructive; description destructive/90; NO destructive-foreground |

## Measurements (px)

| token | dims |
|---|---|
| **container** | pad-x 16 · pad-y 12 · radius 8 · border 1 · font 14/400/20 · width 100% |
| grid | cols `16px_1fr` (has-svg) / `0_1fr` (no-svg) · gap-x 12 · gap-y 2 · items-start |
| icon | size 16 · stroke 1.75 · translate-y 2 · currentColor · col-start 1 |
| title | col-start 2 · font 14/500/20 · min-h 16 · tracking-tight · line-clamp 1 |
| description | col-start 2 · font 14/400/20 · gap-y 4 · `[&_p]` leading relaxed · muted-foreground |

## States

- default: `bg-card text-card-foreground border-border · role=alert`
- destructive: `text-destructive bg-card · icon+title currentColor · desc text-destructive/90`
- no-icon: `grid-cols-[0_1fr] gap-x-0`; title/desc stay col-start-2
- with-icon: `has-[>svg]:grid-cols-[16px_1fr] gap-x-3`; svg col-start-1 translate-y-0.5
- focus/disabled: n/a (static container)

## Rules

- static `div role="alert"`; not focusable; no hover/active/disabled.
- layout is CSS **grid**, not flex: `has-[>svg]:grid-cols-[calc(var(--spacing)*4)_1fr]` else `grid-cols-[0_1fr]`.
- leading icon = direct first-child `<svg>`; color always currentColor (never a separate token). Lucide `size-4` (16), stroke 1.5–2, `translate-y-0.5` to optically center with title cap-height.
- title + description both `col-start-2` so multi-line text never wraps under the icon.
- title single line `line-clamp-1 min-h-4 font-medium tracking-tight`; description muted-foreground, `gap-1` between block children, `[&_p]:leading-relaxed`.
- border always 1px `border-border` (destructive does not recolor it).
- destructive text = the destructive token (text sits on card bg; destructive-foreground is NOT used).
- radius fixed 8 (`rounded-lg`), never pill; width 100%.

## Tokens consumed (semantic only)

`card` `card-foreground` `muted-foreground` `destructive` `border`

## Overrides (skeleton → change when)

- pad 16/12 → `px-3 py-2` (12/8) for compact density.
- radius 8 → 0 (flat) or 12 (soft).
- variant set → add success/warning/info (new semantic status tokens; keep `bg-card` + colored text/icon, description /90).
- icon col 16 → 20 (+ gap-x 16) for a lg icon.
- `border-border` → tinted style `variant/15` bg + `variant/30` border.

## Figc binding

Grid frame; bind pad/gap → `space/*`, radius → `radii/*`, icon/text/border → semantic tokens. Verify `figc bound`.
