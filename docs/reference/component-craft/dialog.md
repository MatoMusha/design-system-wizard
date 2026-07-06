# Dialog (Modal) — build spec

Foundation: shadcn/Radix + Tailwind, 4px grid. Backing: Radix Dialog (`Root/Portal/Overlay/Content/Close/Title/Description`). APG: Dialog (Modal). Skeleton default — override rules in `README.md`.

## Variants

| variant | notes |
|---|---|
| default | dismissible; `showCloseButton=true`; Escape + overlay-click close; header + optional footer |
| alert | AlertDialog pattern; `showCloseButton=false`; non-dismissible (no Escape/overlay close); `role=alertdialog`; explicit Cancel + Action footer |
| destructive-confirm | alert-shaped; footer primary uses destructive bg + #fff text; Cancel = secondary/outline |

All: bg background · text foreground · border border.

## Measurements (px)

| token | dims |
|---|---|
| **content** | max-w 512 (`sm:max-w-lg`) · w `calc(100%−32)` · pad 24 · gap 16 · radius 8 · border 1 · shadow lg · centered (top/left 50% translate -50%) · z 50 |
| overlay | inset 0 · bg black/50 · blur 0 · z 50 |
| close | icon 16 · top 16 · right 16 · radius 2 · hit 44 · opacity 70→100 · stroke 1.75 |
| header | flex-col · gap 8 · text center → `sm:left` |
| footer | flex-col-reverse · gap 8 → `sm:flex-row` · justify-end |
| title | font 18/600/18 · foreground |
| description | font 14/400/20 · muted-foreground |

## States

- open: `animate-in fade-in-0 zoom-in-95`; overlay fade-in-0; 200ms
- closed: `animate-out fade-out-0 zoom-out-95`; overlay fade-out-0; 200ms
- hover: close button opacity 100
- focus-visible: `ring 3px ring/50 · offset 0 · border→ring` (close + focusable footer controls)
- disabled: opacity 50 · pointer-events none (footer controls)

## Rules

- render in a portal; lock body scroll while open (Radix). Center via `fixed top-50%/left-50% + translate -50%/-50%`.
- mobile gutter `max-w-[calc(100%-2rem)]` (16 each side); `sm:max-w-lg` (512) above.
- content radius 8 (`rounded-lg`); pad 24 (`p-6`); internal gap 16 (`gap-4`).
- close = Lucide X currentColor `size-4` (16) stroke 1.5–2 + `span.sr-only "Close"`; ≥44×44 hit area.
- DialogTitle REQUIRED for `aria-labelledby` (Radix warns if absent); DialogDescription drives `aria-describedby`.
- footer stacks reversed on mobile (primary on top), right-aligns in a row on sm+.
- title wraps freely, `font 18/600 leading-none`.
- alert/destructive-confirm: `showCloseButton=false` + disable Escape/outside-click close.
- never blur overlay by default; solid `bg-black/50` only.

## Tokens consumed (semantic only)

`background` `foreground` `border` `muted-foreground` `accent` `accent-foreground` `ring` `primary` `primary-foreground` `secondary` `destructive`

## Overrides (skeleton → change when)

- overlay-blur → `backdrop-blur-sm` for a frosted scrim (default 0).
- content max-width → `sm:max-w-2xl`/`max-w-xl` for a form/wide dialog; full-bleed sheet on mobile.
- close button + dismiss → off for alert/confirm (`preventDefault` on escape/pointer-down-outside).
- radius 8 → 0 (sharp) or 12/28 (soft/M3).
- title font → `text-xl` (20/600) lg density / 16/600 compact.
- focus-ring → shadcn ships ring-2/offset-2 on close; replace with universal 3px ring/50 offset 0.
- shadow → `shadow-xl` for a high-elevation surface.

## Figc binding

Overlay + content frames; bind pad/gap → `space/*`, radius → `radii/*`, fills/border → semantic tokens; place footer buttons via `figc place`. Verify `figc bound`.
