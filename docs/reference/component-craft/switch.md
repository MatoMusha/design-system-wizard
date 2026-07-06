# Switch — build spec

Foundation: shadcn/Radix + Tailwind, 4px grid. Backing: Radix Switch (`Root` + `Thumb`). APG: Switch. Skeleton default — override rules in `README.md`. **Track radius always full (pill).**

## Variants

| variant | track | border | notes |
|---|---|---|---|
| default | unchecked input · checked primary | border-transparent 1px (focus → ring) | thumb bg background; single canonical variant |
| dark-tuned | unchecked input/80 · checked primary | border-transparent | thumb unchecked foreground, checked primary-foreground (dark only) |

## Measurements (px)

| token | dims |
|---|---|
| **root/default** | track-w 32 · track-h 18 · border 1 · radius full · gap-to-label 8 · transition 150 |
| track | w 32 · h 18 · inner 30×16 · radius full · bg input→primary |
| thumb | size 16 · radius full · offset-unchecked 0 · offset-checked 14 (`calc(100%-2px)`) · travel 14 |
| focus-ring | ring 3 · ring/50 · offset 0 · border→ring |
| hit-area | min 44×44 (visual box stays 32×18) |
| label | font 14/500/20 · gap 8 · color foreground |

## States

- unchecked: `track bg-input · thumb translate-x-0`
- checked: `track bg-primary · thumb translate-x-[calc(100%-2px)]` (14)
- focus-visible: `border→ring + ring-ring/50 ring-[3px] offset 0; outline-none`
- disabled: `opacity-50 · cursor-not-allowed · pointer-events-none`
- invalid (opt): `aria-invalid → border/ring destructive` (not stock shadcn)

## Rules

- Root `role=switch` with `aria-checked` (Radix).
- track radius = full (pill, intrinsic) — never 8.
- thumb 16, track 32×18, checked travel `calc(100% − 2px)` = 14 so thumb kisses inner edge.
- border 1px transparent reserves space so focus `border→ring` adds no layout shift.
- thumb bg = background (light); dark swaps to foreground / primary-foreground.
- label right of track, gap 8, `font 14/500`, foreground, sentence case.
- hit area ≥44×44 via the label row — never inflate the 32×18 visual.
- transition `all 150ms ease` on Root, `transition-transform` on Thumb; `shadow-xs` on track; no text/icon in thumb.

## Tokens consumed (semantic only)

`primary` `primary-foreground` `input` `background` `foreground` `ring` `border` `destructive`

## Overrides (skeleton → change when)

- track 32 / thumb 16 / travel 14 → scale to track 44×24, thumb 20, travel 20 when pairing with lg 44 controls.
- unchecked track input → input/80 (dark, for contrast).
- thumb bg background → foreground / primary-foreground (dark).
- checked track primary → a success token for a success-semantic toggle set.
- border-transparent → destructive border + ring on `aria-invalid`.

## Figc binding

Track + thumb; bind size → `size/*`, radius full, fills → semantic tokens; thumb travel via variant. Verify `figc bound`.
