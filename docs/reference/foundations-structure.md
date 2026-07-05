# Design System Foundations — Structural Reference

> **Purpose & scope.** This document reverse-engineers the *structural and scale patterns* of a
> mature, publicly-documented React design system for use as **functional inspiration** in our own
> Figma Foundations and token architecture. It records **facts** (scale values, token naming,
> organizational skeleton), not branding, prose, or identity. We adopt organizational patterns and
> scale philosophy; we do **not** reproduce brand assets, copy prose, or credit the source in shipped
> material. Values marked *[observed]* were read directly from the reference docs; *[inferred]* were
> deduced from patterns.

---

## 1. Foundations structure (the organizational skeleton)

The system separates **Guide** (conceptual/how-to), **Foundations** (design tokens + visual systems),
and **Libraries** (tooling) at the top level. The Foundations set is **one page per token family plus
an aggregate**: *[observed]*

| Foundation page | Covers |
|---|---|
| **All Tokens** (aggregate) | Every token family in one machine-readable index |
| **Color** | Semantic color roles, status, data-viz, syntax |
| **Typography** | Font families, sizes, weights, semantic type scale |
| **Spacing** | 4px-based spacing ladder |
| **Shape** | Radius scale (semantic hierarchy) |
| **Elevation** | Shadow / elevation levels + inset rings |
| **Motion** | Duration bands + easing |
| **Icons** | Semantic icon registry, sizing |
| **Illustrations** | Illustration assets |

**Pattern to adopt:** one Figma page per foundation family, **plus a single "All Tokens" overview
page** that indexes everything. This gives a clean 1:1 between token families and documentation pages.

---

## 2. Spacing scale *[observed]*

- **Base unit: 4px.** Naming convention: `--spacing-[step]`, where the step number roughly maps to
  `value / 4` (so `--spacing-4` = 16px). Half-steps use a hyphen (`--spacing-0-5` = 2px).

| Token | px | Token | px |
|---|---|---|---|
| `--spacing-0` | 0 | `--spacing-6` | 24 |
| `--spacing-0-5` | 2 | `--spacing-7` | 28 |
| `--spacing-1` | 4 | `--spacing-8` | 32 |
| `--spacing-1-5` | 6 | `--spacing-9` | 36 |
| `--spacing-2` | 8 | `--spacing-10` | 40 |
| `--spacing-3` | 12 | `--spacing-11` | 44 |
| `--spacing-4` | 16 | `--spacing-12` | 48 |
| `--spacing-5` | 20 | | |

**Note:** the step number is *not* strictly `value/4` past the low end (there is no `--spacing-2-5`
for 10px; the ladder skips to 12 at step 3). Steps 3→12 increment by a clean +4px each. The
half-steps (`0-5`, `1-5`) exist only at the fine-grained bottom of the scale.

---

## 3. Type scale *[observed]*

**Two layers:** a raw **font-size ladder** (geometric, base 14px, ratio 1.2) and a **semantic type
scale** that composes size + weight + line-height into named roles.

### Font sizes — `--font-size-[name]` (base 0.875rem/14px, ratio ≈ 1.2)
| Token | rem | Token | rem |
|---|---|---|---|
| `--font-size-4xs` | 0.375 | `--font-size-lg` | 1.0625 |
| `--font-size-3xs` | 0.4375 | `--font-size-xl` | 1.25 |
| `--font-size-2xs` | 0.5 | `--font-size-2xl` | 1.5 |
| `--font-size-xs` | 0.625 | `--font-size-3xl` | 1.8125 |
| `--font-size-sm` | 0.75 | `--font-size-4xl` | 2.1875 |
| `--font-size-base` | 0.875 | `--font-size-5xl` | 2.625 |

### Font weights — `--font-weight-[name]`
`normal` 400 · `medium` 500 · `semibold` 600 · `bold` 700

### Semantic type scale — `--text-[name]-size` / `-weight` / `-leading`
Line-heights are **unitless ratios snapped to a 4px vertical grid**.

| Role | Size token | Weight | Leading |
|---|---|---|---|
| Display 1 | 5xl | semibold | 1.2381 |
| Display 2 | 4xl | semibold | 1.2571 |
| Display 3 | 3xl | semibold | 1.2414 |
| H1 | 2xl | semibold | 1.3333 |
| H2 | xl | semibold | 1.4 |
| H3 | lg | semibold | 1.4118 |
| H4 | base | semibold | 1.4286 |
| H5 | sm | semibold | 1.6667 |
| H6 | xs | semibold | 1.6 |
| Large | lg | semibold | 1.4118 |
| Body | base | normal | 1.4286 |
| Label | base | medium | 1.4286 |
| Code | base | normal | 1.4286 |
| Supporting | sm | normal | 1.6667 |

**Pattern to adopt:** headings, display, and body/label/supporting are all **one flat semantic list**
that each reference the raw size ladder — not three separate scales. Weight is baked into the semantic
role (headings=semibold, body=normal, label=medium).

---

## 4. Shape / radius *[observed]*

Semantic hierarchy **inner → element → container → page**, with two fixed anchors (`none`, `full`)
that never scale with theme multipliers.

| Token | Value | Used for |
|---|---|---|
| `--radius-none` | 0 | fixed anchor |
| `--radius-inner` | 8px | nested elements inside a control |
| `--radius-element` | 12px | interactive controls (button, input, selector) |
| `--radius-container` | 16px | cards, panels, dialogs |
| `--radius-chat` | 28px | chat bubbles (domain-specific) |
| `--radius-page` | 32px | page-level surfaces |
| `--radius-full` | 9999px | pills, badges, avatar dots — fixed anchor |

**Concentric radius rule:** inner radius of a nested/padded element = `max(0, outerRadius − padding)`.

**Pattern to adopt:** name radii by **role in the nesting hierarchy**, not by t-shirt size. This makes
radius intent self-documenting and lets a single theme multiplier rescale the whole set while `none`
and `full` stay pinned.

---

## 5. Size tokens (control sizing as tokens) *[observed]*

Component/control heights are modeled as a small **element-size ladder**, separate from spacing:

| Token | Value |
|---|---|
| `--size-element-sm` | 28px |
| `--size-element-md` | 32px |
| `--size-element-lg` | 36px |

Plus `--border-width` = 1px (single token).

**Pattern to adopt:** treat the **height of interactive controls** (buttons, inputs, selectors) as a
first-class token family (`--size-element-*`) with sm/md/lg. Components then take a `size` prop that
maps to these tokens, rather than hardcoding heights. Note the tight 4px increments (28/32/36) — the
ladder is deliberately compact/dense.

---

## 6. Color *[observed]*

**Naming:** `--color-[semantic-role]-[variant]` (variant optional — e.g. `--color-text-primary`,
`--color-border-emphasized`, or bare `--color-success`). Tokens describe **purpose, not appearance**.

**Light/dark:** every color adapts automatically via CSS `light-dark()`; docs present values as
`light / dark` pairs (e.g. `#15110C / #DFE2E5`). One token, two resolved values.

**Semantic role groups:**
- **Surfaces** — accent, neutral, background (body / surface / card / popover), overlay, inverted
- **Text** — primary, secondary, disabled, accent
- **Icons** — primary, secondary, disabled, accent
- **Borders** — standard, emphasized
- **Status** — success, error, warning
- **Data visualization** — categorical + sequential scales (e.g. `--color-data-blue-5` … `-1`, 5 steps)
- **Syntax highlighting** — keyword, string, comment, number, …
- **Brand** — primary brand color

Distinction maintained between a **foundation palette** (raw blues/greens/oranges, mainly for data-viz)
and **semantic tokens** layered on top. Most semantic roles carry 2–3 variants
(primary / secondary / disabled-or-muted).

**Pattern to adopt:** semantic-first color naming with `light-dark()` value tuples; group by role
(text / icon / surface / border / status / data / syntax); keep raw palette separate from semantic.

---

## 7. Elevation & motion *[observed]*

### Elevation — `--shadow-*` (8 tokens, values adapt to dark mode via `light-dark()`)
- **3 elevation levels:** `--shadow-low`, `--shadow-med`, `--shadow-high` (each adds stronger offset+spread)
- **5 inset state rings:** `--shadow-inset-hover`, `-selected`, `-success`, `-warning`, `-error`
  (used for focus/selection/validation rings rather than drop shadows)

*Numeric shadow values are not published on the reference page — only the level structure.*

### Motion — duration + easing
Three speed **bands**, each with min / standard / max (9 duration tokens total):

| | min | standard | max |
|---|---|---|---|
| **fast** | 130ms | 175ms | 230ms |
| **medium** | 310ms | 410ms | 550ms |
| **slow** | 730ms | 975ms | 1300ms |

Easing: a **single** `--ease-standard` = `cubic-bezier(0.24, 1, 0.4, 1)`.

**Pattern to adopt:** motion as **named speed bands with min/std/max**, and one shared easing curve —
constrains motion choices to a small, consistent set. Elevation split into **stacking shadows** vs
**inset state rings**.

---

## 8. Component inventory & categories *[observed]*

~250 exported symbols (components + hooks) grouped into **~12 categories**. Import path convention:
components come from **per-category subpath entrypoints** (keeps bundles small, makes intent explicit).

| Category | Representative components |
|---|---|
| **Action** | Button, ButtonGroup, IconButton, ToggleButton, ToggleButtonGroup, DropdownMenu, MoreMenu, SegmentedControl, Toolbar, Link |
| **Container** | Card, ClickableCard, SelectableCard, Collapsible, Carousel |
| **Content** | Text, Heading, Avatar, AvatarGroup, Blockquote, Citation, Code, CodeBlock, Icon, Kbd, Markdown, Thumbnail, Timestamp, Token, EmptyState |
| **Data Input** | TextInput, TextArea, NumberInput, Checkbox, RadioList, Switch, Slider, Selector, MultiSelector, Typeahead, Field, FileInput, Calendar, Date/Time/DateRange inputs, PowerSearch, Tokenizer |
| **Feedback & Status** | Badge, Banner, ProgressBar, Skeleton, Spinner, StatusDot |
| **Layout** | AppShell, Layout, Grid, Section, FormLayout, Divider, AspectRatio, ResizeHandle, (Stacks: HStack/VStack/Center) |
| **Navigation** | Breadcrumbs, Pagination, SideNav, TopNav, TopNavMegaMenu, TabList, Outline |
| **Overlay** | Dialog, Popover, Tooltip, HoverCard, Toast, CommandPalette, Lightbox, Overlay |
| **Table & List** | Table, List, TreeList, MetadataList, OverflowList |
| **Chat** (domain) | ChatComposer, ChatLayout, ChatMessage, ChatSystemMessage, ChatToolCalls, ChatMessageMetadata |
| **Utility** | VisuallyHidden |

Each interactive component is paired with **hooks** (`useTooltip`, `usePopover`, `useTable*`, etc.) —
headless logic separated from presentation.

**Pattern to adopt:** group components by **functional role** (Action / Container / Content / Data
Input / Feedback / Layout / Navigation / Overlay / Table & List), and mirror these groups as sections
in the Figma component library. Domain clusters (e.g. Chat) can be their own category.

---

## 9. Token generation approach *[observed]*

Tokens are **generated from a scale config**, not hand-authored per token — a `defineTheme` config with
typed scale generators plus explicit overrides:

- **Typography scale** — from `base` (px) + `ratio` (e.g. 14 × 1.2).
- **Radius scale** — from `base` (px) + `multiplier` (0–2); generates inner/element/container/page/chat.
- **Motion scale** — from `fast` (ms) + `medium` (ms) + `ratio`.
- **Color scale** — from an `accent` hex + optional `neutralStyle` (warm/cool/neutral) + `contrast`;
  generates accent/background/text/border tokens.

**Overrides precedence:** *"Explicit token overrides always take precedence over scale-generated
values."* A `tokens` field sets CSS vars directly, e.g. `'--color-accent': ['#7B61FF', '#9B85FF']`
(the array = the light/dark `light-dark()` tuple).

**Theme inheritance:** an `extends` field allows a theme to inherit from a base. Merge behavior differs
by field — tokens are copied then child-overridden; components deep-merged; icons shallow-merged;
scale configs (typography/radius) replaced wholesale by the child.

**Packaging:** themes ship as `@{scope}/theme-{name}` packages exporting two subpaths — a runtime-
injection default (dev) and a pre-compiled `/built` CSS (production/SSR).

**Machine-readable angle:** the **All Tokens** page is effectively a structured index of every family
and value — the same data that feeds AI-assisted usage and the theme generator. Tokens are authored as
data (scale params + override maps), making the whole system introspectable.

**Pattern to adopt:** define our foundations as a **scale config (base + ratio/multiplier) with an
explicit-overrides layer**, so a theme is mostly a handful of parameters plus targeted overrides. Keep
an "All Tokens" export as the single machine-readable source of truth for both docs and Figma
variable sync.

---

## 10. Summary of patterns worth importing

1. **Foundations = one page per token family + an "All Tokens" aggregate.**
2. **Spacing:** 4px base, `--spacing-[step]` where step ≈ value/4, half-steps only at the low end (0-5, 1-5).
3. **Type:** raw geometric size ladder (base 14, ratio 1.2) + a flat semantic scale (display/H1–H6/body/label/supporting) with weight baked in and 4px-grid-snapped line-heights.
4. **Radius:** role-based nesting hierarchy (inner→element→container→page) + fixed `none`/`full` anchors + concentric formula.
5. **Size tokens:** control heights as a first-class `--size-element-sm/md/lg` (28/32/36px) ladder driving a `size` prop.
6. **Color:** semantic-first `--color-[role]-[variant]`, `light-dark()` value tuples, roles grouped by text/icon/surface/border/status/data/syntax, raw palette kept separate.
7. **Elevation/motion:** 3 shadow levels + inset state rings; motion as fast/medium/slow bands (min/std/max) with a single easing curve.
8. **Components:** grouped by functional role, per-category import paths, presentation split from headless hooks.
9. **Tokens generated from scale configs with an explicit-override layer; an "All Tokens" index as the machine-readable source of truth.**

*Source: publicly-published documentation of an established open-source React design system,
fetched 2026-07-05. Recorded as neutral structural/functional reference only (scale values and
organization) — no branding, identity, prose, or origin is reproduced.*
