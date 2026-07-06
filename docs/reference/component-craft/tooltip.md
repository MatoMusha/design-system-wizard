# Tooltip — build spec

Foundation: shadcn/Radix + Tailwind, 4px grid. Backing: Radix Tooltip (`Provider/Root/Trigger/Portal/Content/Arrow`). APG: Tooltip (`role=tooltip`, non-focusable, hover/focus triggered, Escape dismisses). Skeleton default — override rules in `README.md`.

## Variants

| variant | bg | text | border | notes |
|---|---|---|---|---|
| default (inverse) | primary | primary-foreground | none | shadcn ships this only; high-contrast chip, arrow fill-primary |
| surface (on-canvas) | popover | popover-foreground | border | opt-in for light-on-light; 1px border, arrow fill-popover + border edge |

## Measurements (px)

| token | dims |
|---|---|
| **content** | pad-x 12 · pad-y 6 · font 12/400/16 · radius 8 · max-w none (w-fit) · side-offset 0 · z 50 |
| arrow | size 10 · rotate 45 · corner-radius 2 · translate-y -2 · fill primary |
| content (multiline opt) | max-w 280 · text-balance · lh 16 · pad-y 6 |

## States

- closed: unmounted via Portal; opens on trigger hover/focus after delay
- open: `animate-in fade-in-0 zoom-in-95 + slide-in-from-{side}-2`; origin `--radix-tooltip-content-transform-origin`; 150ms
- closing: `data-[state=closed]:animate-out fade-out-0 zoom-out-95`; 150ms
- trigger-focus-visible: `ring 3px ring/50 offset 0 border→ring` on the TRIGGER; tooltip opens immediately (no hover delay)
- trigger-disabled: opacity 50 pointer-events-none on trigger; wrap in span if tooltip must still fire
- dismiss: Escape or pointer-leave closes; collision auto-flips side

## Rules

- text-only content — no interactive/focusable children (use HoverCard/Popover instead). `role=tooltip`; never the sole source of critical info.
- single concise line, sentence case, no terminal period; `w-fit`; wrap via `text-balance` only past max-w cap.
- `delayDuration 700ms` open, `skipDelayDuration 300ms` (Radix TooltipProvider defaults).
- render in Portal at z-50; transform-origin bound to `--radix-tooltip-content-transform-origin`.
- arrow = `TooltipPrimitive.Arrow`, `size-2.5 rotate-45 fill-primary translate-y-[-2px]`; `sideOffset 0` (arrow bridges the gap); Radix handles collision side-flip.
- icons Lucide currentColor stroke 1.5–2, gap 4 if paired with text.
- trigger exposes ≥44×44 hit area even when visually smaller.

## Tokens consumed (semantic only)

`primary` `primary-foreground` `popover` `popover-foreground` `border` `radius`

## Overrides (skeleton → change when)

- variant → surface (popover + border) when the tooltip sits on a light/muted surface where the inverse chip lacks contrast.
- radius → `rounded-md` resolves to 6 under a compact/dense theme (`--radius 0.5rem`) instead of 8.
- max-width → cap 280 + `text-balance` for rich/multiline copy; else `w-fit` unbounded.
- font-weight → 500 when very small inverse text needs a legibility boost (default 400).
- sideOffset → 4–6 when the arrow is omitted, to restore the trigger gap.

## Figc binding

Content chip + arrow; bind pad → `space/*`, radius → `radii/*`, fills → semantic tokens. Verify `figc bound`.
