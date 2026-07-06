# Avatar — build spec

Foundation: shadcn/Radix + Tailwind, 4px grid. Backing: Radix Avatar (`Root/Image/Fallback`). APG: none (img/presentation; wrap in Button/Link with an accessible name if interactive). Skeleton default — override rules in `README.md`. **Radius intrinsic full.**

Anatomy: root (relative flex, overflow-hidden, rounded-full, shrink-0) · image (aspect-square, size-full, object-cover) · fallback (initials/icon, centered, bg-muted) · optional status ring/border · group wrapper (overlap stack).

## Variants

| variant | bg | text | border | notes |
|---|---|---|---|---|
| image | muted | muted-foreground | none | image object-cover; muted only shows while loading/errored |
| fallback-initials | muted | muted-foreground | none | 1–2 uppercase initials, centered |
| fallback-icon | muted | muted-foreground | none | Lucide User currentColor stroke 1.5–2, ~55% of box |
| ringed (group/status) | muted | muted-foreground | background | `ring-2` ring-color = background to punch a gap between adjacent avatars |

## Measurements (px)

| token | dims |
|---|---|
| sm | box 24 · radius full · image cover · fallback-font 12/500/16 · icon 14 |
| **default** | box 32 · radius full · image cover · fallback-font 14/500/20 · icon 16 |
| lg | box 40 · radius full · image cover · fallback-font 16/500/24 · icon 20 |
| image (part) | aspect-square · size 100% · object-cover · inherits root radius via overflow-hidden |
| fallback (part) | size 100% · flex center · bg muted · radius full |
| group-overlap (part) | offset -8 (margin-left -8/sibling) · ring 2 · ring-color background · later sibling on top |
| status-ring (part) | ring 2 · offset 0 · ring-color background |

## States

- default: root `rounded-full overflow-hidden shrink-0`; image `object-cover` fills; else fallback `bg-muted text-muted-foreground`
- loading: Radix delays fallback (`delayMs`) then shows it until image decodes; no layout shift (box fixed)
- image-error: AvatarImage unmounts → AvatarFallback shown automatically
- hover: non-interactive none; group → optional z-index raise so hovered avatar overlaps siblings
- focus-visible: only when interactive wrapper present → `outline-none, ring 3px ring/50, offset 0, border→ring`; bare avatar not focusable
- disabled: opacity 50 · pointer-events-none (interactive wrapper only)

## Rules

- root `relative flex size-{6|8|10} shrink-0 overflow-hidden rounded-full`; image `aspect-square size-full object-cover`; fallback `flex size-full items-center justify-center rounded-full bg-muted`.
- radius intrinsic full (9999) — do NOT apply the 8px skeleton radius.
- initials uppercase, max 2 chars, centered, no wrap; icon fallback Lucide (User) currentColor stroke 1.5–2, ~55% of box.
- `shrink-0` always so the avatar never squashes in flex rows.
- group: wrapper `flex -space-x-2` (-8) and `*:data-[slot=avatar]:ring-2 *:data-[slot=avatar]:ring-background`.
- `overflow-hidden` on root clips image to circle — never set radius on the image itself.
- if clickable, wrap in button/link with ≥44×44 hit area + accessible name (alt/aria-label); transition `all 150ms ease`.
- set `data-slot=avatar / avatar-image / avatar-fallback` for group ring selectors.

## Tokens consumed (semantic only)

`muted` `muted-foreground` `background` `foreground` `border` `ring`

## Overrides (skeleton → change when)

- radius full → 8 (`rounded-md`, squircle) for an org/brand/entity instead of a person.
- box sizes → 20/28/36 for a compact table or comment-thread.
- ring-color background → primary/destructive/chart token when encoding presence/status/selection.
- overlap offset (`-space-x`) → pair magnitude with ring width so the gap stays visible.
- add `border 1px border` when a light avatar on a light surface needs an edge without the group ring.

## Figc binding

Root + fallback; bind size → `size/*`, radius full, fills → semantic tokens; place image via component. Verify `figc bound`.
