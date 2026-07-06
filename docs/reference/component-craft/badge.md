# Badge — build spec

Foundation: shadcn/Radix + Tailwind, 4px grid. Backing: native `span` (`data-slot="badge"`), polymorphic to `<a>` via Radix Slot when `asChild`. APG: none (non-interactive label). Skeleton default — override rules in `README.md`.

## Variants

| variant | bg | text | border | notes |
|---|---|---|---|---|
| default | primary | primary-foreground | transparent | `[a&]:hover:bg-primary/90` |
| secondary | secondary | secondary-foreground | transparent | `[a&]:hover:bg-secondary/90` |
| destructive | destructive | #fff (NOT destructive-foreground) | transparent | hover /90; focus ring destructive/20 (dark /40); dark bg-destructive/60 |
| outline | transparent | foreground | border | `[a&]:hover:bg-accent [a&]:hover:text-accent-foreground` |

## Measurements (px)

| token | dims |
|---|---|
| **box** | height ~22 intrinsic (line 16 + pad-y 2·2 + border 1·2) · pad-x 8 · pad-y 2 · gap 4 · icon 12 · font 12/500/16 · radius 6 · border 1 |
| icon | size 12 (`size-3`) · stroke 1.5–2 · currentColor · gap 4 |

## States

- default: variant bg + text per table; border 1px transparent (outline = border token)
- hover: only when polymorphic `[a&]` → bg /90 (outline → bg accent, text accent-foreground); static span has none
- focus-visible: `border→ring · ring ring/50 3px · offset 0 · outline-none` (destructive: ring destructive/20, dark /40)
- disabled: n/a on static badge; on link/button host → opacity 50 · pointer-events-none
- invalid: `aria-invalid` → ring destructive/20 (dark /40) · border-destructive

## Rules

- `inline-flex items-center justify-center; w-fit shrink-0; whitespace-nowrap; overflow-hidden` (never wrap/grow).
- radius 6 (`rounded-md`) — pick 6 over full to match the skeleton corner family; pill only if the variant-set opts in.
- icon Lucide currentColor `size-3` (12), stroke 1.5–2; `[&>svg]:pointer-events-none`, gap 4.
- destructive text is literal `#fff` (`text-white`), never destructive-foreground.
- transition `transition-[color,box-shadow] 150ms ease`; label short, no forced uppercase.
- non-interactive by default; add `role=status` for live status, or `asChild`→`<a>` for links.

## Tokens consumed (semantic only)

`primary` `primary-foreground` `secondary` `secondary-foreground` `destructive` `foreground` `border` `ring` `accent` `accent-foreground` `radius`

## Overrides (skeleton → change when)

- radius → full (pill) for count/notification badges or a pill tag-set.
- height/font → 24–28 / `text-sm` when it sits inline with default-36 controls and looks undersized.
- pad-x → 10 (`px-2.5`) when an icon+text combo needs breathing room.
- add success/warning variants → extend cva with the same `border-transparent + bg/text` pair pattern (new semantic tokens).
- dismissible (trailing X, ≥44×44 hit area) when the badge becomes a removable filter/tag chip.

## Figc binding

Chip component; bind pad/gap → `space/*`, radius → `radii/*`, fills → semantic tokens. Verify `figc bound`.
