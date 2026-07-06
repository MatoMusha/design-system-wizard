# Label — build spec

Foundation: shadcn/Radix + Tailwind. Backing: Radix `LabelPrimitive.Root` → native `<label>` (consumed by shadcn FormLabel/FieldLabel). APG: none — accessible name via `htmlFor`↔`id`. Skeleton default — override rules in `README.md`. **Not part of the control-height family.**

Anatomy: label text · leading/trailing icon slot (gap 8, optional) · required marker (optional) · associated control (external, `htmlFor`→`id`) · helper/description (sibling, not part of label).

## Variants

| variant | text | notes |
|---|---|---|
| default | foreground (inherited) | `text-sm/500 leading-none` |
| invalid (`data-[error=true]`) | destructive | FormLabel flips to destructive on field error |
| disabled / peer-disabled | foreground | opacity 50 + pointer-events-none / cursor-not-allowed |
| required | foreground; marker destructive | trailing asterisk `text-destructive` |
| legend | foreground | `text-base/500` section caption; mb 12 |
| select-card | foreground; checked bg primary/5 | wraps control in bordered `rounded-md` card; checked → border-primary |

## Measurements (px)

| token | dims |
|---|---|
| **label (base)** | font 14/500/14 (leading-none) · internal-gap 8 · icon 16 · height auto |
| gap-to-control (FormItem) | 8 |
| field container (vertical) | gap 12 · horizontal → flex-row items-center gap 12 |
| field-content (label+desc) | gap 6 · lh ~20 |
| legend (variant=legend) | font 16/500/24 · mb 12 |
| description sibling | font 14/400/20 · muted-foreground |
| required marker | ml 2 · destructive |
| select-card wrapper | pad 16 · radius 8 · border 1 |

## States

- default: text foreground (currentColor); `font 14/500 leading-none`; `select-none`
- click: delegates focus to control via `htmlFor` (label non-interactive)
- focus-visible: none on label — the associated control shows the universal ring
- disabled (`group-data-[disabled=true]`) / peer-disabled: opacity 50 · pointer-events-none / cursor-not-allowed
- invalid (`data-[error=true]`): `text-destructive`
- checked (select-card): `border-primary · bg-primary/5` (dark /10)

## Rules

- render as native `<label>` via Radix `LabelPrimitive.Root`; `htmlFor` = control's `id`.
- layout `flex items-center gap-2`; `w-fit`; type `text-sm/500 leading-none`; `select-none`.
- icons: Lucide, currentColor, stroke 1.5–2, 16px, in the gap-2 slot.
- no own radius/height (not in control-height family). Sentence case, no uppercase.
- disabled only via peer/group data-attr — never style label disabled independent of its control.
- required marker opt-in: `after:content-['*'] after:ml-0.5 after:text-destructive` (shadcn ships none).
- transition `all 150ms ease`.

## Tokens consumed (semantic only)

`foreground` `destructive` `muted-foreground` `primary` `primary-foreground` `input` `border` `background` `ring`

## Overrides (skeleton → change when)

- orientation → `Field flex-row items-center`, label `flex-auto` for inline (checkbox/radio/switch rows).
- container gap 12 → 6 (FieldContent) or checkbox/radio group `gap-3`.
- `text-sm` → `text-base` (legend) when labelling a fieldset/section.
- `leading-none` → `leading-snug` when label wraps or pairs with inline description.
- add `rounded-md 8 + border input + pad 16 + checked border-primary/bg-primary/5` for a selectable-card label.
- append destructive asterisk when the form marks required fields visually.

## Figc binding

Text style bound to a type token; gap → `space/*`. Card variant: bind pad/radius/border tokens. Verify `figc bound`.
