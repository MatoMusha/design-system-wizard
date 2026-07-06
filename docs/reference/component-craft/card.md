# Card — build spec

Foundation: shadcn/Radix + Tailwind, 4px grid. Backing: native `div` (`data-slot="card"`, no Radix). APG: none intrinsic — adopt Link/Button semantics only when the whole card is clickable. Skeleton default — override rules in `README.md`.

## Variants

| variant | bg | text | border | notes |
|---|---|---|---|---|
| default (elevated) | card | card-foreground | border | 1px border + shadow-sm |
| outline (flat) | card | card-foreground | border | shadow-none; border carries the edge |
| muted (informational) | muted | muted-foreground | transparent | filled, no border/shadow |
| interactive (clickable) | card | card-foreground | border | + hover bg-accent, focus-ring, cursor-pointer, ≥44px hit area |
| selected | card | card-foreground | ring | 1px ring-colored border |

## Measurements (px)

| token | dims |
|---|---|
| **container** | py 24 · px 24 (per-section) · gap 24 · radius 8 · border 1 · shadow sm |
| header | px 24 · gap 8 · pb 24 (when border-b) · grid rows auto/auto |
| title | font 16/600/16 · leading-none |
| description | font 14/400/20 · muted-foreground |
| content | px 24 · py 0 |
| footer | px 24 · pt 24 (when border-t) · flex items-center · gap 8 |
| action-slot | col-start 2 · row-span 2 · self-start · justify-self-end |

## States

- default: `bg-card · text-card-foreground · border 1px border · shadow-sm`
- hover (interactive): `bg-accent · text-accent-foreground · cursor-pointer`
- focus-visible (interactive): `ring 3px · ring/50 · offset 0 · border→ring · outline-none`
- selected: `border-color ring · aria-selected true`
- disabled: opacity 50 · pointer-events none

## Rules

- vertical padding on the container (py 24); horizontal padding on each section (px 24) so dividers span full width.
- gap 24 between header/content/footer; header gap 8 (title↔description).
- CardTitle: `leading-none + font-semibold` (600), inherits 16px base. CardDescription: `text-sm` (14/20) muted-foreground.
- header → 2-col grid `[1fr_auto]` when a CardAction is present (action `col-start-2 row-span-2 self-start justify-self-end`).
- `border-b` on header adds pb 24; `border-t` on footer adds pt 24.
- no fixed height — content-driven. Non-interactive by default: hover/focus-ring only when the whole card is a link/button, then ≥44×44 hit area.
- icons Lucide currentColor stroke 1.5–2; transition `all 150ms ease`.

## Tokens consumed (semantic only)

`card` `card-foreground` `border` `muted-foreground` `accent` `accent-foreground` `muted` `foreground` `ring` `radius`

## Overrides (skeleton → change when)

- radius 8 → skeleton pins 8 (shadcn base ships `rounded-xl` ~12–14; M3/Polaris 12) — raise per corner scale.
- padding 24 → 16 for compact density (Material/Polaris/Primer).
- shadow → `shadow-none` for outline/flat.
- section gap 24 → 16 for dense content.
- bg → `bg-muted` (drop border+shadow) for the muted variant.

## Figc binding

Frame with section sub-frames (header/content/footer); bind pad/gap → `space/*`, radius → `radii/*`, fills/border → semantic tokens. Verify `figc bound`.
