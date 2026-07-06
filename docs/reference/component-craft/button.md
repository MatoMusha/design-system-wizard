# Button — build spec

Foundation: shadcn/Radix + Tailwind, 4px grid. Backing primitive: native `<button>`
(Radix `Slot` for `asChild`). APG: Button. Skeleton default — see `README.md` for the
override rules.

## Variants (6)

| variant | bg | text | border | hover | notes |
|---|---|---|---|---|---|
| `primary` | `primary` | `primary-foreground` | none | bg 90% | default intent |
| `secondary` | `secondary` | `secondary-foreground` | none | bg 80% | neutral filled |
| `destructive` | `destructive` | `#fff` | none | bg 90% | **no `destructive-foreground` token — text is white** |
| `outline` | `background` | `foreground` | 1px `border` | bg `accent` / text `accent-foreground` | |
| `ghost` | transparent | `foreground` | none | bg `accent` / text `accent-foreground` | |
| `link` | transparent | `primary` | none | `underline` (offset 4) | no height/padding box; inline |

## Size table (px) — locked default: height 36, radius 8

| token | height | h-pad | h-pad w/icon | icon-only | gap | icon | font (size/wt/lh) | radius |
|---|---|---|---|---|---|---|---|---|
| `xs` (opt) | 24 | 8 | 6 | 24×24 | 4 | 12 | 12 / 500 / 16 | 8 |
| `sm` | 32 | 12 | 10 | 32×32 | 6 | 16 | 14 / 500 / 20 | 8 |
| **`default`** | **36** | 16 | 12 | 36×36 | 8 | 16 | 14 / 500 / 20 | 8 |
| `lg` | 44 | 24 | 20 | 44×44 | 8 | 20 | 16 / 500 / 24 | 8 |

`link` variant ignores the height/padding/radius columns (inline text).

## State recipe (key:value)

- `hover`: filled → `bg 90%` (secondary 80%); `outline`·`ghost` → `bg accent · text accent-foreground`
- `active`: `bg 80%`
- `focus-visible`: `ring 3px · ring/50 · offset 0 · border→ring`
- `disabled`: `opacity 50% · pointer-events none`
- `loading`: `spinner replaces leading icon · label dims · aria-busy=true · pointer-events none`
- `transition`: `all 150ms ease` (none on `:active`)

## Rules

- `whitespace-nowrap`, `shrink-0`, **no min-width** (width = content). Icon-only = square (= height).
- `block` prop → `width:100%`.
- **Touch target ≥44×44** via hit-area expander (`@media (pointer:coarse)`); `lg` meets it natively.
- Icon: Lucide, size per table, `currentColor` (inherits text token), stroke 1.5–2px, `shrink-0`.
- Label: sentence-case, verb-first. No uppercase transform. No wrap.

## Tokens consumed (semantic only)

`primary` `primary-foreground` `secondary` `secondary-foreground` `destructive`
`background` `foreground` `accent` `accent-foreground` `border` `ring` `radius`
· heights `size/element-{sm,md,lg}` · padding `space/*` · radii `radii/*`

## Overrides (skeleton → change when)

- **height family** → `size/element-*` per interview density (README table). Default `md`=36.
- **radius** → per interview corner style (README table). Default 8; `link` none.
- **variant set** → drop `link` for non-web/app-only systems; add brand intents (e.g. `success`) only as new **semantic** tokens surfaced in the plan.
- **xs size** → include only if the brief has a data-dense surface.
- **weight** → `500` default; a brand may lock `600` (Tailwind-style) system-wide.

## Figma binding (figc)

- Build the variant × size matrix as a component set, **auto-layout**, gap/padding bound to `space/*`.
- `figc bind <id> gap|padding|radius <token>` for every spacing/radius field; fills bound to the semantic color tokens above.
- Verify with `figc bound <id>` — no raw numbers/hex on any field.
