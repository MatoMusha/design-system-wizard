# The Component Contract

**Status:** Normative standard for `design-system-wizard`
**Applies to:** every component in a design system managed by this plugin
**Consumed by:** `/setup` (emits), `/extend` (conforms), `/audit` (scores)

---

## 1. What the Component Contract is

The **Component Contract** is a single, typed, structured content model that documents one component. It is the atomic unit of design-system documentation in this plugin. Each component has exactly one Contract.

A Contract is **dual-surface** by construction:

- **Human surface** — the Contract renders to a readable documentation page (Markdown/MDX, a docs site, or a Storybook doc block). Designers and engineers read it.
- **Machine surface** — the same Contract is a parseable data structure (YAML front-of-file + typed body, with a canonical JSON projection). Agents (this plugin's `/extend`, `/audit`, code generators, and downstream AI assistants) read and emit it without scraping prose.

There is no second, separate "AI copy" of the docs. **One source, two projections.** The prose a human reads is generated from the same nodes an agent parses. This is the core rule: if information exists for a human but not as structured data (or vice versa), the Contract is malformed.

A Contract is **not freeform prose.** It is a tree of *typed content nodes*. Prose exists only inside `prose` nodes, which are themselves addressable and typed. Everything load-bearing — props, tokens, do/don't guidance, examples, a11y — lives in first-class typed structures, never buried in paragraphs.

### 1.1 Design goals

1. **Every fact is addressable.** A prop's default, an example's code, a Don't item — each is a field an agent can read by path, not a sentence it must parse.
2. **Accessibility is not a section, it is a property.** A11y lives *inside* each prop and interaction, so it cannot drift out of sync with the API it describes.
3. **Deterministic to audit.** Every requirement in this document maps to a binary check in §10. A Contract either has a props table with all required fields or it does not.
4. **One-way from Figma.** The Contract's token references and component identity trace back to Figma as the source of truth; the Contract never invents tokens or values.

---

## 2. The typed node model

A Contract body is an ordered list of **content nodes**. Six node types exist. All are both renderable and machine-readable.

| Node type | Purpose | Renders as |
|-----------|---------|------------|
| `prose` | Explanatory text. The only place free text is allowed. | Paragraph(s) |
| `code` | A code sample. Carries `lang` and a human `label`. | Fenced, labelled code block |
| `list` | An ordered or unordered list of strings. | Bulleted / numbered list |
| `table` | Generic tabular data (`columns` + `rows`). | Table |
| `propsTable` | **First-class** typed component API table. Not a generic table. | Props reference table |
| `guidance` | **First-class** structured Do / Don't block. | Do/Don't callout pair |

### 2.1 Node schemas

```yaml
# prose
- kind: prose
  text: string            # Markdown-inline allowed (bold, code spans, links). No block structure.

# code
- kind: code
  lang: string            # "tsx" | "ts" | "css" | "bash" | "json" | ...
  label: string           # human caption, e.g. "Import" or "Loading state"
  code: string            # verbatim source

# list
- kind: list
  ordered: boolean        # default false
  items: [string]

# table
- kind: table
  columns: [string]
  rows: [[string]]        # each row length == columns length

# propsTable  (see §3)
- kind: propsTable
  props: [Prop]

# guidance  (see §5)
- kind: guidance
  do:   [GuidanceItem]
  dont: [GuidanceItem]
```

Nodes are placed inside named **sections** (§6). The section skeleton is fixed; the nodes inside each section are flexible except where a section *requires* a particular node type (e.g. **Props** must contain a `propsTable`).

---

## 3. The props table (first-class)

The props table is the heart of the Contract. It is **not** a generic table — it is a typed structure with a fixed field set. Each row is a `Prop`.

### 3.1 `Prop` field set (all fields normative)

| Field | Type | Required | Meaning |
|-------|------|----------|---------|
| `name` | string | yes | Prop name as written in code (e.g. `variant`, `isLoading`). |
| `type` | string | yes | **Precise** type. Union props MUST enumerate literals: `'primary' \| 'secondary' \| 'ghost' \| 'destructive'`. Never `string` when a union is meant. |
| `description` | string | yes | What the prop does **and its accessibility behavior** (see §4). A11y is written *into* this field, not appended elsewhere. |
| `required` | boolean | yes | Whether the prop must be supplied. |
| `default` | string \| null | yes | Default value as source text (`'secondary'`, `false`, `'md'`) or `null` if none. |
| `slotElements` | [string] \| null | yes | For slot/render-prop/composition props: the element(s) or component(s) this slot accepts (e.g. `['ReactNode']`, `['<a>', '<button>']`). `null` if not a slot. |
| `a11y` | A11y \| null | conditional | **Required for every interactive or state-bearing prop** (§4). `null` only for purely presentational props with no assistive-tech surface. |

### 3.2 Type precision rule

- Enumerable props MUST list every literal, joined by ` | `.
- Boolean props use `boolean`.
- The `default` for a union prop MUST be one of that prop's literals.
- No prop may be typed `any`. `unknown` is permitted only for genuinely opaque pass-through props and must be justified in `description`.

---

## 4. Accessibility, baked into props (mandatory)

Accessibility is documented **per prop and per interaction**, inline, never as a bolted-on afterthought section. The external authority is the **[WAI-ARIA Authoring Practices Guide (APG)](https://www.w3.org/WAI/ARIA/apg/)** — Contracts cite APG pattern names where a component maps to one (e.g. "Button", "Disclosure", "Dialog (Modal)").

### 4.1 The `A11y` shape (per prop)

Every interactive prop and every prop that changes assistive-tech-visible state MUST carry an `a11y` object:

```yaml
a11y:
  ariaMapping: string        # how this prop maps to ARIA. e.g. "sets aria-disabled; does NOT set native disabled"
  keyboard: string | null    # keyboard interaction this prop introduces or alters. e.g. "Space/Enter activate"
  focus: string | null       # focus behavior. e.g. "remains focusable while disabled so tooltip is reachable"
  announcement: string | null # live-region / screen-reader announcement. e.g. "announces 'Loading' via aria-live=polite"
```

A field is `null` only when genuinely not applicable; it may not be omitted.

### 4.2 What "documents its a11y" means (the bar)

For each interactive prop, the `description` and/or `a11y` object MUST make explicit, where applicable:

- **ARIA mapping** — which `aria-*` attribute(s) or `role` the prop drives, and any deliberate divergence from native semantics (e.g. `aria-disabled` instead of native `disabled` so the element stays in the tab order and keeps a reachable tooltip).
- **Keyboard interaction** — keys that operate the control (e.g. `Space`/`Enter` activate; `Esc` dismisses), per the relevant APG pattern.
- **Focus behavior** — whether/when the element is focusable, focus movement it triggers, and focus-visible expectations.
- **Live-region announcements** — what a screen reader announces on state change (e.g. an `isLoading` prop that "announces *Loading* via a polite live region").

### 4.3 Reference examples (illustrative prop descriptions)

- `label` — "Visible text. **Used as the `aria-label` when `isIconOnly` is true**, so an icon-only button is still named for screen readers."
- `isLoading` — "Shows a spinner and disables activation. **Announces *Loading* via an `aria-live=\"polite\"` region**; restores prior label on completion."
- `isDisabled` — "Prevents activation. **Applies `aria-disabled` rather than the native `disabled` attribute** so the element stays focusable and its tooltip remains reachable per APG."

### 4.4 Component-level a11y

Beyond props, the **Best Practices** section (§6) MUST include the component's overall APG pattern reference and any roving-tabindex / focus-trap / labelling requirements that are not expressible on a single prop.

---

## 5. Structured Do / Don't guidance

Guidance is data, not prose. Every Contract carries at least one `guidance` node with **at least one Do and at least one Don't**.

### 5.1 Schema

```yaml
- kind: guidance
  do:
    - text: string          # the recommended practice
      rationale: string | null  # why. optional but strongly encouraged
  dont:
    - text: string
      rationale: string | null
```

`GuidanceItem = { text: string, rationale: string | null }`.

### 5.2 Examples

```yaml
- kind: guidance
  do:
    - text: "Use semantic tokens such as --color-text-primary for all colors."
      rationale: "They adapt across themes and modes automatically."
    - text: "Give icon-only buttons a label prop."
      rationale: "It becomes the accessible name."
  dont:
    - text: "Hardcode hex values like #1A73E8."
      rationale: "They won't adapt to dark mode or rebrands."
    - text: "Nest a Button inside an anchor or another button."
      rationale: "Invalid HTML and unpredictable for assistive tech."
```

---

## 6. Canonical section skeleton (ordered)

Every Contract has the **same ordered sections**. `/setup` emits them in this order; `/audit` checks presence and order; `/extend` fills them.

| # | Section | Required | Contains |
|---|---------|----------|----------|
| 1 | **Overview** | yes | `prose` describing what the component is and the problem it solves. Includes the APG pattern name if one applies. |
| 2 | **When to Use / When Not to Use** | yes | Two `list` nodes (or a `guidance`-style pair): when to reach for this component, and when to reach for a different one. |
| 3 | **Usage** | yes | The canonical **import** statement (`code`, label "Import") + at least one live example (§7). Shows the minimal correct usage. |
| 4 | **Best Practices** | yes | At least one `guidance` node (Do/Don't). Component-level a11y notes (§4.4). Token usage notes (§8). |
| 5 | **Props** | yes | Exactly one `propsTable` node (§3). This is the API reference for the component. |
| 6 | **Related Components** | yes | `list` of cross-links to sibling Contracts (e.g. Button → IconButton, Link, ButtonGroup). May be empty only for genuinely standalone components. |

Sections 1–6 are the fixed spine. Additional `prose`/`code` nodes may appear within a section but no new top-level sections may be introduced without extending this standard.

---

## 7. Live preview / example-as-data

Examples must **both render as real React and feed agents.** They are stored as structured `example` records, never as opaque prose blobs.

### 7.1 `Example` schema

```yaml
examples:
  - id: string             # stable slug, e.g. "button-loading"
    title: string          # human caption
    description: string | null
    code: string           # the JSX/TSX that renders the preview — verbatim, runnable
    showcase: boolean       # true => rendered in the large "--showcase" visual demo strip
    renders: boolean        # true => this example is mounted as a live React preview
```

### 7.2 Rules

- Every component's **Usage** section MUST contain the canonical **import** as a `code` node labelled `"Import"`.
- At least **one** example MUST have `renders: true` (a real, mounted React preview — not a screenshot).
- At least **one** example SHOULD carry `showcase: true` to populate the visual demo strip.
- Example `code` is the single source: the docs site mounts it live, and agents read the same string. No separate "for humans" screenshot substitutes for a renderable example.
- Examples MUST use only **semantic tokens** (§8) — an example containing a hardcoded hex color fails audit.

---

## 8. Two-tier token rule

The design system has two token tiers:

- **Primitive tokens** — raw values (`--blue-600`, `--space-4`, `#1A73E8`). The palette. Not for direct component use.
- **Semantic tokens** — intent-named, theme-aware aliases of primitives (`--color-text-primary`, `--color-bg-danger`, `--space-inline-sm`). These are what components consume.

**Rule (normative):** every value a Contract references — in props, in Best Practices, and in every example — MUST be a **semantic token**. A Contract MUST NOT reference a primitive token, and MUST NOT contain a hardcoded literal color (hex, `rgb()`, `hsl()`, named CSS color) anywhere in its examples or guidance.

This ties into a separate **token parity requirement** (documented in the token-parity convention): the set of semantic tokens a Contract references must exist in the Figma source and survive the one-way Style Dictionary → CSS-variable export unchanged. The Contract is downstream of Figma; it names tokens, it never defines them. `/audit` cross-checks referenced semantic tokens against the exported CSS-variable set and flags any that do not resolve.

---

## 9. Reference-vs-guide split

Two kinds of documentation exist and MUST be kept in separate spaces:

- **Reference** — the per-component Contract. Precise, exhaustive, API-shaped: props, types, a11y, examples. This is what this standard governs. It lives in the component's own reference space (one Contract per component).
- **Guides** — conceptual, cross-cutting narrative: "theming", "getting started", "accessibility philosophy", "migrating from v1". Guides are freeform and live in a separate guides space.

**Rule:** A Contract is reference. It MUST NOT absorb conceptual guide material, and a guide MUST NOT redefine a component's API. Cross-links are the join: a guide links to Contracts; a Contract's **Related Components** links to sibling Contracts, not to guides. `/audit` operates only on Contracts (reference); it does not score guides.

---

## 10. Machine-readable Contract shape (canonical)

A Contract serializes to this canonical JSON/YAML. This is what `/setup` emits and `/extend` produces; the Markdown page is a projection of it.

```yaml
component:
  name: string                 # "Button"
  slug: string                 # "button"
  status: draft | stable | deprecated
  apgPattern: string | null    # "Button" — WAI-ARIA APG pattern, or null
  import: string               # "import { Button } from '@acme/ui'"
  tokensUsed: [string]         # semantic tokens referenced, e.g. ["--color-bg-primary"]

sections:                      # ordered, matches §6 spine
  overview:      { nodes: [Node] }
  whenToUse:     { use: [string], dont: [string] }
  usage:         { nodes: [Node], examples: [Example] }
  bestPractices: { nodes: [Node] }   # MUST include >=1 guidance node
  props:         { table: { props: [Prop] } }
  related:       { links: [{ name: string, slug: string }] }

# Node, Prop, A11y, Example, GuidanceItem as defined in §2–§7
```

A valid Contract round-trips: JSON → Markdown page → parsed back to equivalent JSON. Anything that survives only in the rendered Markdown but not the JSON is non-conformant.

---

## 11. Worked example — `Button`

Below is a complete, conformant Contract for a neutral `Button`, shown first as the machine surface (YAML), then as the rendered human surface.

### 11.1 Machine surface

```yaml
component:
  name: Button
  slug: button
  status: stable
  apgPattern: Button
  import: "import { Button } from '@acme/ui'"
  tokensUsed:
    - --color-bg-primary
    - --color-text-on-primary
    - --color-bg-secondary
    - --color-text-primary
    - --color-bg-danger
    - --space-inline-md
    - --radius-control

sections:
  overview:
    nodes:
      - kind: prose
        text: >
          Button triggers an action or event — submitting a form, opening a dialog,
          confirming a choice. It maps to the WAI-ARIA APG **Button** pattern and renders
          a native `<button>` unless composed as a link. Use it for actions; use **Link**
          for navigation.

  whenToUse:
    use:
      - "Triggering an action in place (submit, confirm, open, delete)."
      - "The primary or secondary call-to-action on a view."
      - "Icon-only affordances in toolbars (with a label for the accessible name)."
    dont:
      - "Navigating to another page or route — use Link, which renders an anchor."
      - "Toggling a persistent on/off state — use Switch or ToggleButton."
      - "Long-running background tasks with no immediate feedback — pair with a status region."

  usage:
    nodes:
      - kind: code
        lang: tsx
        label: Import
        code: "import { Button } from '@acme/ui'"
    examples:
      - id: button-basic
        title: Basic
        description: Default secondary button.
        code: "<Button onClick={save}>Save changes</Button>"
        showcase: true
        renders: true
      - id: button-variants
        title: Variants
        description: The four visual variants.
        code: |
          <>
            <Button variant="primary">Primary</Button>
            <Button variant="secondary">Secondary</Button>
            <Button variant="ghost">Ghost</Button>
            <Button variant="destructive">Delete</Button>
          </>
        showcase: true
        renders: true
      - id: button-loading
        title: Loading
        description: Announces its loading state to assistive tech.
        code: "<Button isLoading>Saving…</Button>"
        showcase: false
        renders: true
      - id: button-icon-only
        title: Icon only
        description: Requires a label for the accessible name.
        code: '<Button isIconOnly label="Add item"><PlusIcon /></Button>'
        showcase: false
        renders: true

  bestPractices:
    nodes:
      - kind: prose
        text: >
          Follows the APG **Button** pattern: operable with **Space** and **Enter**,
          focusable, and named by its text content or `label`. Keep one primary button
          per view so the main action stays unambiguous.
      - kind: guidance
        do:
          - text: "Use semantic tokens (e.g. --color-bg-primary) for all styling."
            rationale: "They adapt across themes and modes automatically."
          - text: "Give every icon-only button a label."
            rationale: "It becomes the accessible name for screen readers."
          - text: "Reserve the destructive variant for irreversible actions."
            rationale: "It signals danger; overuse dilutes the signal."
        dont:
          - text: "Hardcode hex values such as #1A73E8."
            rationale: "They won't adapt to dark mode or a rebrand."
          - text: "Use a Button to navigate between pages."
            rationale: "Screen-reader users expect a link; use Link."
          - text: "Place more than one primary button in the same view."
            rationale: "It removes the clear primary action."

  props:
    table:
      props:
        - name: variant
          type: "'primary' | 'secondary' | 'ghost' | 'destructive'"
          description: "Visual emphasis of the button. destructive also sets the danger token set."
          required: false
          default: "'secondary'"
          slotElements: null
          a11y:
            ariaMapping: "Purely visual; no ARIA impact. destructive does not by itself convey danger to AT — pair with a clear label."
            keyboard: null
            focus: null
            announcement: null
        - name: size
          type: "'sm' | 'md' | 'lg'"
          description: "Control height and padding. Hit target stays >=24px CSS px at sm per WCAG 2.2 target size."
          required: false
          default: "'md'"
          slotElements: null
          a11y:
            ariaMapping: "No ARIA impact."
            keyboard: null
            focus: null
            announcement: null
        - name: children
          type: "ReactNode"
          description: "Visible button content. Provides the accessible name unless isIconOnly is set."
          required: false
          default: "null"
          slotElements: ["ReactNode"]
          a11y:
            ariaMapping: "Text content becomes the accessible name."
            keyboard: null
            focus: null
            announcement: null
        - name: label
          type: "string"
          description: "Accessible name. Used as the aria-label when isIconOnly is true, so an icon-only button is still named."
          required: false
          default: "null"
          slotElements: null
          a11y:
            ariaMapping: "Applied as aria-label when isIconOnly is true."
            keyboard: null
            focus: null
            announcement: null
        - name: isIconOnly
          type: "boolean"
          description: "Renders only an icon. Requires label to supply the accessible name."
          required: false
          default: "false"
          slotElements: null
          a11y:
            ariaMapping: "When true, the accessible name comes from label (aria-label), not visible text."
            keyboard: null
            focus: null
            announcement: null
        - name: isLoading
          type: "boolean"
          description: "Shows a spinner and blocks activation while a task runs. Announces 'Loading' via a polite live region and restores the label on completion."
          required: false
          default: "false"
          slotElements: null
          a11y:
            ariaMapping: "Sets aria-busy=\"true\" on the button while loading."
            keyboard: "Activation (Space/Enter) is suppressed while loading."
            focus: "Button remains focusable during loading."
            announcement: "Announces 'Loading' via an aria-live=\"polite\" region."
        - name: isDisabled
          type: "boolean"
          description: "Prevents activation. Uses aria-disabled instead of the native disabled attribute so the element stays focusable and any tooltip remains reachable."
          required: false
          default: "false"
          slotElements: null
          a11y:
            ariaMapping: "Applies aria-disabled=\"true\"; does NOT set native disabled."
            keyboard: "Activation via Space/Enter is prevented."
            focus: "Stays in the tab order so the tooltip and reason stay reachable, per APG."
            announcement: "Screen readers announce the button as dimmed/unavailable."
        - name: onClick
          type: "(e: MouseEvent) => void"
          description: "Fires on activation by pointer or keyboard. Not called when isDisabled or isLoading."
          required: false
          default: "null"
          slotElements: null
          a11y:
            ariaMapping: "No ARIA impact."
            keyboard: "Invoked by Space and Enter, matching the APG Button pattern."
            focus: "No focus movement unless the handler navigates."
            announcement: null
        - name: type
          type: "'button' | 'submit' | 'reset'"
          description: "Native button type. Use submit only inside a form."
          required: false
          default: "'button'"
          slotElements: null
          a11y:
            ariaMapping: "Native semantics; submit associates with the form."
            keyboard: "submit is also triggered by Enter within the form."
            focus: null
            announcement: null

  related:
    links:
      - { name: IconButton, slug: icon-button }
      - { name: ButtonGroup, slug: button-group }
      - { name: Link, slug: link }
```

### 11.2 Human surface (rendered projection)

> ## Button
> `import { Button } from '@acme/ui'` · Pattern: [APG Button](https://www.w3.org/WAI/ARIA/apg/patterns/button/) · Status: stable
>
> ### Overview
> Button triggers an action or event — submitting a form, opening a dialog, confirming a choice. It maps to the APG **Button** pattern and renders a native `<button>` unless composed as a link. Use it for actions; use **Link** for navigation.
>
> ### When to Use / When Not to Use
> **Use it when** — triggering an action in place · it's the primary/secondary CTA · icon-only toolbar affordances (with a label).
> **Don't use it when** — navigating to a page (use **Link**) · toggling persistent state (use **Switch**) · long background tasks with no feedback.
>
> ### Usage
> ```tsx
> // Import
> import { Button } from '@acme/ui'
> ```
> *[live preview: Basic · Variants · Loading · Icon only]*
>
> ### Best Practices
> Follows the APG Button pattern: Space/Enter activate, focusable, named by content or `label`. Keep one primary button per view.
>
> **Do** — use semantic tokens · label every icon-only button · reserve destructive for irreversible actions.
> **Don't** — hardcode hex values · navigate with a Button · place two primaries in one view.
>
> ### Props
> *(rendered from the `propsTable`: name · type · description-with-a11y · required · default · slot)*
>
> ### Related Components
> IconButton · ButtonGroup · Link

---

## 12. Component Contract Evaluation Checklist

`/audit` runs these checks against an existing component's Contract. **Each is binary (PASS / FAIL) and machine-checkable** against the canonical JSON (§10). A Contract's audit score is `passed / total`. Checks marked **[blocker]** must pass for a Contract to be considered conformant at all.

### Structure
1. **[blocker]** All six spine sections present, in order: Overview → When to Use → Usage → Best Practices → Props → Related. `sections` has all six keys.
2. `component.name`, `component.slug`, `component.status`, `component.import` are all non-empty.
3. `component.apgPattern` is present (a string or explicit `null`), not missing.

### Props
4. **[blocker]** Props section contains exactly one `propsTable` node.
5. **[blocker]** Every `Prop` has all seven required fields present: `name`, `type`, `description`, `required`, `default`, `slotElements`, `a11y`.
6. No prop's `type` is `any`. `unknown` appears only with a justification in `description`.
7. Every union-typed prop enumerates ≥2 literals joined by ` | `, and its `default` (if non-null) is one of those literals.
8. Every prop's `default` field is present (a value or `null`) — never missing.

### Accessibility
9. **[blocker]** Every interactive or state-bearing prop (event handlers, boolean state props, props that set ARIA) has a non-null `a11y` object.
10. **[blocker]** Each such `a11y` object has all four keys present (`ariaMapping`, `keyboard`, `focus`, `announcement`), each a string or explicit `null`.
11. Any prop whose name or description implies a loading/busy state documents a live-region `announcement`.
12. Any prop whose name or description implies a disabled state documents its `ariaMapping` and `focus` behavior.
13. The Best Practices section references the component's APG pattern (matches `component.apgPattern` when non-null).

### Guidance
14. **[blocker]** At least one `guidance` node exists with **≥1 Do item and ≥1 Don't item**.
15. Every `GuidanceItem` has a non-empty `text` field.

### Examples & live preview
16. **[blocker]** Usage contains a `code` node labelled `"Import"` with non-empty code matching `component.import`.
17. At least one `Example` has `renders: true`.
18. Every `Example` has non-empty `code`, a `title`, and boolean `showcase` and `renders` fields.

### Tokens (two-tier)
19. **[blocker]** No hardcoded literal color (hex, `rgb()`, `rgba()`, `hsl()`, or named CSS color) appears in any example `code`, guidance item, or prose node.
20. No primitive token (matching the primitive namespace, e.g. `--blue-*`, `--space-<n>` raw scale) is referenced anywhere; only semantic tokens appear.
21. **[blocker]** Every token in `component.tokensUsed` resolves against the exported CSS-variable set (token-parity cross-check).

### Reference/guide hygiene
22. Related Components links resolve to existing Contract slugs (or the list is explicitly empty for a standalone component).
23. The Contract contains no conceptual guide material misplaced as a top-level section (only the six spine sections exist).

### Round-trip
24. **[blocker]** The Contract round-trips: parsing the rendered page reproduces the canonical JSON with no human-only or machine-only divergence.

---

*A component is **Contract-conformant** when every `[blocker]` check passes; its **Contract score** is the fraction of all 24 checks passed. `/setup` emits Contracts that pass all 24; `/extend` must not lower a component's score; `/audit` reports the failing check numbers with the offending path.*
