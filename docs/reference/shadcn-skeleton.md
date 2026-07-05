# shadcn/ui Skeleton Spec — Token + Component Contract

> Reverse-engineered build reference for the Modulon design system. shadcn/ui is our **code target** (MIT-licensed), so the values below are the exact parity contract our Figma `semantic` collection and code must mirror.
>
> **Basis:** shadcn/ui current `new-york-v4` style, Tailwind v4, `cssVariables: true`, `baseColor: neutral`, color space **oklch**. Components use CSS `data-slot` attributes and are backed by `radix-ui`.
>
> **Sources:**
> - Theming / CSS variables: https://ui.shadcn.com/docs/theming
> - components.json config: https://ui.shadcn.com/docs/components-json
> - Component source (verbatim): `https://ui.shadcn.com/r/styles/new-york-v4/<name>.json`
> - Canonical globals.css: `https://raw.githubusercontent.com/shadcn-ui/ui/main/apps/v4/app/globals.css`

---

## 1. Token Contract — Full CSS Variable Set (light + dark, oklch)

This is the **semantic layer**. Every value is verbatim. Light values live in `:root`, dark in `.dark`. Dark mode is a **class strategy** (`.dark` on an ancestor), not a media query.

| Token | Light (`:root`) | Dark (`.dark`) |
|---|---|---|
| `--background` | `oklch(1 0 0)` | `oklch(0.145 0 0)` |
| `--foreground` | `oklch(0.145 0 0)` | `oklch(0.985 0 0)` |
| `--card` | `oklch(1 0 0)` | `oklch(0.205 0 0)` |
| `--card-foreground` | `oklch(0.145 0 0)` | `oklch(0.985 0 0)` |
| `--popover` | `oklch(1 0 0)` | `oklch(0.205 0 0)` |
| `--popover-foreground` | `oklch(0.145 0 0)` | `oklch(0.985 0 0)` |
| `--primary` | `oklch(0.205 0 0)` | `oklch(0.922 0 0)` |
| `--primary-foreground` | `oklch(0.985 0 0)` | `oklch(0.205 0 0)` |
| `--secondary` | `oklch(0.97 0 0)` | `oklch(0.269 0 0)` |
| `--secondary-foreground` | `oklch(0.205 0 0)` | `oklch(0.985 0 0)` |
| `--muted` | `oklch(0.97 0 0)` | `oklch(0.269 0 0)` |
| `--muted-foreground` | `oklch(0.556 0 0)` | `oklch(0.708 0 0)` |
| `--accent` | `oklch(0.97 0 0)` | `oklch(0.269 0 0)` |
| `--accent-foreground` | `oklch(0.205 0 0)` | `oklch(0.985 0 0)` |
| `--destructive` | `oklch(0.577 0.245 27.325)` | `oklch(0.704 0.191 22.216)` |
| `--border` | `oklch(0.922 0 0)` | `oklch(1 0 0 / 10%)` |
| `--input` | `oklch(0.922 0 0)` | `oklch(1 0 0 / 15%)` |
| `--ring` | `oklch(0.708 0 0)` | `oklch(0.556 0 0)` |
| `--chart-1` | `oklch(0.646 0.222 41.116)` | `oklch(0.488 0.243 264.376)` |
| `--chart-2` | `oklch(0.6 0.118 184.704)` | `oklch(0.696 0.17 162.48)` |
| `--chart-3` | `oklch(0.398 0.07 227.392)` | `oklch(0.769 0.188 70.08)` |
| `--chart-4` | `oklch(0.828 0.189 84.429)` | `oklch(0.627 0.265 303.9)` |
| `--chart-5` | `oklch(0.769 0.188 70.08)` | `oklch(0.645 0.246 16.439)` |
| `--sidebar` | `oklch(0.985 0 0)` | `oklch(0.205 0 0)` |
| `--sidebar-foreground` | `oklch(0.145 0 0)` | `oklch(0.985 0 0)` |
| `--sidebar-primary` | `oklch(0.205 0 0)` | `oklch(0.488 0.243 264.376)` |
| `--sidebar-primary-foreground` | `oklch(0.985 0 0)` | `oklch(0.985 0 0)` |
| `--sidebar-accent` | `oklch(0.97 0 0)` | `oklch(0.269 0 0)` |
| `--sidebar-accent-foreground` | `oklch(0.205 0 0)` | `oklch(0.985 0 0)` |
| `--sidebar-border` | `oklch(0.922 0 0)` | `oklch(1 0 0 / 10%)` |
| `--sidebar-ring` | `oklch(0.708 0 0)` | `oklch(0.556 0 0)` |
| `--radius` | `0.625rem` | (inherited from `:root`) |

### Important parity notes
- **`--destructive-foreground` is NOT shipped** in the current neutral theme. Destructive surfaces render their text with the literal utility `text-white` (see Button/Badge below), not a token. Do **not** invent a `--destructive-foreground` in Figma unless you also change the component contract — the code target hardcodes white.
- `--border` / `--input` in dark mode are **translucent white** (`oklch(1 0 0 / 10%)` and `/ 15%`), not opaque greys. Reproduce the alpha in Figma.
- Neutral base = fully desaturated (chroma `0`, hue `0`). Only `--destructive` and the chart colors carry chroma/hue.
- `--primary-foreground` in dark stays `0.205` (dark text on the near-white `--primary` `0.922`) — the primary pair **inverts** between themes.

---

## 2. Radius System

One authored token, `--radius: 0.625rem` (= **10px** at 16px root). All component radii derive from it via `calc()` inside the Tailwind v4 `@theme inline` block (multiplier-based in the current build):

```css
@theme inline {
  --radius-sm: calc(var(--radius) * 0.6);   /* 0.375rem  6px  */
  --radius-md: calc(var(--radius) * 0.8);   /* 0.5rem    8px  */
  --radius-lg: var(--radius);               /* 0.625rem  10px */
  --radius-xl: calc(var(--radius) * 1.4);   /* 0.875rem  14px */
  --radius-2xl: calc(var(--radius) * 1.8);  /* 1.125rem  18px */
  --radius-3xl: calc(var(--radius) * 2.2);  /* 1.375rem  22px */
  --radius-4xl: calc(var(--radius) * 2.6);  /* 1.625rem  26px */
}
```

Utility → value mapping used by components: `rounded-sm`→`--radius-sm`, `rounded-md`→`--radius-md`, `rounded-lg`→`--radius-lg`, `rounded-xl`→`--radius-xl`. `rounded-full` = pill (9999px).

> Legacy note: older shadcn builds derived these as `calc(var(--radius) - 4px)`, `- 2px`, `var(--radius)`, `+ 4px`. Current canonical globals.css is the **multiplier** form above — use it.

**Radius used per component:** Button/Input/Select-trigger/Tooltip `rounded-md`; Card/Dialog/Alert/Tabs-list `rounded-lg` (Card is `rounded-xl`); Badge/Avatar/Switch `rounded-full`; Checkbox `rounded-[4px]` (literal); Select-item `rounded-sm`.

---

## 3. Spacing & Sizing

shadcn does not ship a spacing scale — it consumes **Tailwind's default 4px grid** (`--spacing: 0.25rem`; utility `n` = `n × 0.25rem`). Steps actually used in the core components:

| Utility | Value | Seen in |
|---|---|---|
| `gap-1` / `gap-1.5` / `gap-2` | 4 / 6 / 8px | Badge, Button-sm, Button/Label/Select gaps |
| `gap-4` | 16px | Dialog content |
| `gap-6` | 24px | Card (root + between sections) |
| `px-2` / `px-3` / `px-4` / `px-6` | 8 / 12 / 16 / 24px | Badge / Input+Select+Button-sm / Button-default / Card+Dialog padding |
| `py-0.5` / `py-1` / `py-2` / `py-3` / `py-6` | 2 / 4 / 8 / 12 / 24px | Badge / Input / Button / Alert / Card |
| `p-1` / `p-[3px]` / `p-6` | 4 / 3 / 24px | Select viewport / Tabs-list / Dialog |
| `size-3` / `size-3.5` / `size-4` | 12 / 14 / 16px | icon sizes (default svg = `size-4`) |

### Control-height system (the sizing contract)

Heights are expressed as Tailwind `h-*` (`h-8`=32px, `h-9`=36px, `h-10`=40px). **Default control height = `h-9` (36px).**

| Component | sm | default | lg | icon |
|---|---|---|---|---|
| **Button** | `h-8` (32) | `h-9` (36) | `h-10` (40) | `size-9` (36²); also `icon-sm size-8`, `icon-lg size-10`, `icon-xs size-6` |
| **Button (xs)** | `h-6` (24) | — | — | — |
| **Input** | — | `h-9` (36) | — | — |
| **Select trigger** | `data-[size=sm] h-8` (32) | `data-[size=default] h-9` (36) | — | — |
| **Switch** | `h-3.5 w-6` (14×24) | `h-[1.15rem] w-8` (~18×32) | — | — |
| **Avatar** | `size-6` (24) | `size-8` (32) | `size-10` (40) | — |
| **Checkbox** | — | `size-4` (16) | — | — |
| **Tabs list** | — | `h-9` (36) | — | — |

Icon buttons are square (`size-*`). Button horizontal padding scales with size: `px-3` (sm) / `px-4` (default) / `px-6` (lg), with `has-[>svg]` reductions.

---

## 4. Core Component Inventory + Anatomy

All components set `data-slot="<name>"` on each part (styling/targeting hook) and `data-variant` / `data-size` where applicable. Base font size is `text-sm` for most controls (`text-xs` for Badge/Tooltip, `text-base md:text-sm` for Input). Focus convention is universal — see §5.

### Button — `radix-ui` `Slot` (via `asChild`)
- **Variants:** `default | secondary | destructive | outline | ghost | link`
- **Sizes:** `default | sm | lg | icon` (+ extended `xs`, `icon-xs`, `icon-sm`, `icon-lg`)
- **Base:** `inline-flex items-center justify-center gap-2 rounded-md text-sm font-medium whitespace-nowrap transition-all` + focus ring + `disabled:opacity-50 disabled:pointer-events-none` + `aria-invalid` ring + auto svg sizing (`[&_svg:not([class*='size-'])]:size-4`).
- **Variant anatomy:**
  - `default` → `bg-primary text-primary-foreground hover:bg-primary/90`
  - `secondary` → `bg-secondary text-secondary-foreground hover:bg-secondary/80`
  - `destructive` → `bg-destructive text-white hover:bg-destructive/90` + `dark:bg-destructive/60`
  - `outline` → `border bg-background shadow-xs hover:bg-accent hover:text-accent-foreground` + `dark:bg-input/30`
  - `ghost` → `hover:bg-accent hover:text-accent-foreground`
  - `link` → `text-primary underline-offset-4 hover:underline`

### Input — plain `<input>` (no Radix)
- **Variants/sizes:** none (single form). Height `h-9`.
- **Anatomy:** `w-full min-w-0 rounded-md border border-input bg-transparent px-3 py-1 text-base md:text-sm shadow-xs` + `dark:bg-input/30` + `placeholder:text-muted-foreground` + `selection:bg-primary selection:text-primary-foreground` + focus ring + `aria-invalid` ring + `file:*` styling + `disabled:opacity-50 disabled:cursor-not-allowed`.

### Label — `radix-ui` `Label`
- **Anatomy:** `flex items-center gap-2 text-sm leading-none font-medium select-none`; dims when peer/group disabled (`peer-disabled:opacity-50`, `group-data-[disabled=true]:opacity-50`).

### Card — plain `<div>`s (no Radix). Slots: `card`, `card-header`, `card-title`, `card-description`, `card-action`, `card-content`, `card-footer`.
- **Root:** `flex flex-col gap-6 rounded-xl border bg-card py-6 text-card-foreground shadow-sm`
- **Header:** container-query grid, `px-6 gap-2`, promotes `card-action` into a right-aligned column.
- **Title:** `leading-none font-semibold` · **Description:** `text-sm text-muted-foreground` · **Content:** `px-6` · **Footer:** `flex items-center px-6`.

### Badge — `radix-ui` `Slot` (via `asChild`)
- **Variants:** `default | secondary | destructive | outline | ghost | link` (no size axis)
- **Base:** `inline-flex w-fit items-center justify-center gap-1 overflow-hidden rounded-full border border-transparent px-2 py-0.5 text-xs font-medium` + focus/aria-invalid + `[&>svg]:size-3`.
- **Variant anatomy:** `default` `bg-primary text-primary-foreground`; `secondary` `bg-secondary text-secondary-foreground`; `destructive` `bg-destructive text-white`; `outline` `border-border text-foreground`; `ghost` hover-only accent; `link` `text-primary`. Hover states gated on `[a&]` (only when rendered as a link).

### Checkbox — `radix-ui` `Checkbox`
- **Anatomy:** `peer size-4 shrink-0 rounded-[4px] border border-input shadow-xs` + `dark:bg-input/30`; checked → `data-[state=checked]:bg-primary data-[state=checked]:text-primary-foreground data-[state=checked]:border-primary`; focus/aria-invalid ring; disabled dim. Indicator = `CheckIcon size-3.5`, `grid place-content-center`.

### Select — `radix-ui` `Select`. Slots: trigger, content, item, label, separator, group, value, scroll-up/down-button.
- **Trigger sizes:** `sm` (`h-8`) | `default` (`h-9`) via `data-size`. Anatomy: `flex w-fit items-center justify-between gap-2 rounded-md border border-input bg-transparent px-3 py-2 text-sm shadow-xs` + `data-[placeholder]:text-muted-foreground` + `dark:bg-input/30 dark:hover:bg-input/50` + focus ring; trailing `ChevronDownIcon size-4 opacity-50`.
- **Content:** `rounded-md border bg-popover text-popover-foreground shadow-md`, `z-50`, Radix open/close animations, viewport `p-1`.
- **Item:** `rounded-sm py-1.5 pr-8 pl-2 text-sm focus:bg-accent focus:text-accent-foreground`; check indicator right-aligned (`size-3.5`). **Label:** `text-xs text-muted-foreground`. **Separator:** `h-px bg-border`.

### Dialog — `radix-ui` `Dialog`. Slots: dialog, trigger, portal, close, overlay, content, header, footer, title, description.
- **Overlay:** `fixed inset-0 z-50 bg-black/50` + fade animations.
- **Content:** centered `fixed top-[50%] left-[50%] translate-[-50%] z-50 grid w-full max-w-[calc(100%-2rem)] sm:max-w-lg gap-4 rounded-lg border bg-background p-6 shadow-lg` + zoom/fade animations; built-in close button top-right (`size-4` X, `showCloseButton` prop).
- **Header:** `flex flex-col gap-2 text-center sm:text-left` · **Footer:** `flex flex-col-reverse gap-2 sm:flex-row sm:justify-end` · **Title:** `text-lg leading-none font-semibold` · **Description:** `text-sm text-muted-foreground`.

### Tabs — `radix-ui` `Tabs`. Slots: tabs, tabs-list, tabs-trigger, tabs-content.
- **List variants:** `default` (`bg-muted`) | `line` (`bg-transparent gap-1`). List: `inline-flex w-fit items-center justify-center rounded-lg p-[3px] text-muted-foreground h-9` (horizontal).
- **Trigger:** `rounded-md border border-transparent px-2 py-1 text-sm font-medium text-foreground/60`; active → `data-[state=active]:bg-background data-[state=active]:text-foreground` (+ `shadow-sm` in `default` variant; underline `::after` in `line` variant). Supports `horizontal`/`vertical` orientation.
- **Content:** `flex-1 outline-none`.

### Switch — `radix-ui` `Switch`
- **Sizes:** `sm` (`h-3.5 w-6`) | `default` (`h-[1.15rem] w-8`) via `data-size`.
- **Track:** `inline-flex shrink-0 items-center rounded-full border border-transparent shadow-xs`; `data-[state=checked]:bg-primary data-[state=unchecked]:bg-input` (+ `dark:data-[state=unchecked]:bg-input/80`); focus ring; disabled dim.
- **Thumb:** `rounded-full bg-background` (size `4`/`3`), `data-[state=checked]:translate-x-[calc(100%-2px)]`; dark checked thumb → `bg-primary-foreground`.

### Tooltip — `radix-ui` `Tooltip`. Slots: provider, tooltip, trigger, content. `Provider` default `delayDuration={0}`.
- **Content:** `z-50 w-fit rounded-md bg-foreground px-3 py-1.5 text-xs text-balance text-background` + open/side animations. **Inverted** — dark surface on light theme. Arrow = `size-2.5 rotate-45 rounded-[2px] bg-foreground fill-foreground`.

### Separator — `radix-ui` `Separator`
- **Orientation:** `horizontal` (default) | `vertical`. Anatomy: `shrink-0 bg-border`; `data-[orientation=horizontal]:h-px w-full` / `data-[orientation=vertical]:w-px h-full`. `decorative` default true.

### Avatar — `radix-ui` `Avatar`. Slots: avatar, avatar-image, avatar-fallback, avatar-badge, avatar-group, avatar-group-count.
- **Sizes:** `sm` (`size-6`) | `default` (`size-8`) | `lg` (`size-10`) via `data-size`.
- **Root:** `relative flex size-8 shrink-0 overflow-hidden rounded-full select-none`. **Image:** `aspect-square size-full`. **Fallback:** `flex size-full items-center justify-center rounded-full bg-muted text-sm text-muted-foreground`. **Badge:** absolute bottom-right dot `bg-primary text-primary-foreground ring-2 ring-background`. **Group:** `flex -space-x-2` with ring separators.

### Alert — plain `<div>` (no Radix). Slots: alert, alert-title, alert-description.
- **Variants:** `default` (`bg-card text-card-foreground`) | `destructive` (`bg-card text-destructive`, description `text-destructive/90`).
- **Base:** `relative grid w-full grid-cols-[0_1fr] items-start gap-y-0.5 rounded-lg border px-4 py-3 text-sm`; auto-adjusts to icon column with `has-[>svg]` (`grid-cols-[calc(var(--spacing)*4)_1fr] gap-x-3`), `[&>svg]:size-4`.
- **Title:** `col-start-2 line-clamp-1 min-h-4 font-medium tracking-tight` · **Description:** `col-start-2 text-sm text-muted-foreground`.

---

## 5. How Components Consume Tokens

### Utility → semantic-var mapping (Tailwind v4 `@theme inline`)
The `@theme inline` block maps a `--color-*` theme key to each semantic var, which generates the `bg-*`/`text-*`/`border-*`/`ring-*` utilities:

```css
@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-card: var(--card);
  --color-card-foreground: var(--card-foreground);
  --color-popover: var(--popover);
  --color-popover-foreground: var(--popover-foreground);
  --color-primary: var(--primary);
  --color-primary-foreground: var(--primary-foreground);
  --color-secondary: var(--secondary);
  --color-secondary-foreground: var(--secondary-foreground);
  --color-muted: var(--muted);
  --color-muted-foreground: var(--muted-foreground);
  --color-accent: var(--accent);
  --color-accent-foreground: var(--accent-foreground);
  --color-destructive: var(--destructive);
  --color-border: var(--border);
  --color-input: var(--input);
  --color-ring: var(--ring);
  --color-chart-1: var(--chart-1); /* …through --color-chart-5 */
  --color-sidebar: var(--sidebar);
  --color-sidebar-foreground: var(--sidebar-foreground);
  --color-sidebar-primary: var(--sidebar-primary);
  --color-sidebar-primary-foreground: var(--sidebar-primary-foreground);
  --color-sidebar-accent: var(--sidebar-accent);
  --color-sidebar-accent-foreground: var(--sidebar-accent-foreground);
  --color-sidebar-border: var(--sidebar-border);
  --color-sidebar-ring: var(--sidebar-ring);
  /* radius keys from §2 also live here */
}
```

So `bg-primary` → `--color-primary` → `var(--primary)`. Pattern: every surface pairs a `bg-X` with `text-X-foreground` (e.g. `bg-primary text-primary-foreground`, `bg-card text-card-foreground`, `bg-secondary text-secondary-foreground`). `border`/`border-input` → `--border`/`--input`; `ring-ring` → `--ring`. Alpha modifiers (`/90`, `/50`, `/30`) are applied at the utility level for hover/focus/dark states, e.g. `hover:bg-primary/90`, `ring-ring/50`, `dark:bg-input/30`.

### Focus-ring convention (universal)
Interactive controls share this exact recipe:
```
outline-none
focus-visible:border-ring
focus-visible:ring-ring/50
focus-visible:ring-[3px]
```
→ a 3px ring at 50% of `--ring` plus a solid `--ring` border. Invalid state parallels it: `aria-invalid:border-destructive aria-invalid:ring-destructive/20` (`dark:ring-destructive/40`). Dialog's inner close button uses the older `focus:ring-2 focus:ring-ring focus:ring-offset-2` form.

### Dark mode
Class strategy: `.dark { … }` overrides the same variable names (Tailwind v4 `@custom-variant dark (&:is(.dark *))`). Components add `dark:`-prefixed utilities for cases the token swap can't cover — e.g. translucent input fills (`dark:bg-input/30`), destructive intensity (`dark:bg-destructive/60`), and inverted switch thumbs (`dark:data-[state=checked]:bg-primary-foreground`).

---

## 6. components.json (config contract)

```jsonc
{
  "$schema": "https://ui.shadcn.com/schema.json",
  "style": "new-york",          // "default" is deprecated
  "rsc": true,                   // React Server Components
  "tsx": true,
  "tailwind": {
    "config": "",               // empty for Tailwind v4
    "css": "app/globals.css",
    "baseColor": "neutral",     // neutral | stone | zinc | gray | slate
    "cssVariables": true,        // semantic tokens (vs inline utilities)
    "prefix": ""
  },
  "aliases": {
    "components": "@/components",
    "utils": "@/lib/utils",
    "ui": "@/components/ui",
    "lib": "@/lib",
    "hooks": "@/hooks"
  },
  "iconLibrary": "lucide"        // new-york default; icons imported from lucide-react
}
```

- `cssVariables: true` is what makes the semantic layer in §1 the source of truth (set `false` to hardcode utilities instead — not our path).
- Utility merge helper is `cn()` from `@/lib/utils` (clsx + tailwind-merge). Icons come from `lucide-react` (CheckIcon, ChevronDownIcon, XIcon, etc.).
- Class variants are authored with `class-variance-authority` (`cva`) exposing `variants`, `defaultVariants`, and exported `*Variants` functions (`buttonVariants`, `badgeVariants`, `tabsListVariants`, `alertVariants`).
