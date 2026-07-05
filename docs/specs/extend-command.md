# `/extend` — Grow an existing design system by one component (or a small batch)

> **Status:** spec
> **Command:** `/extend [component-name…]`
> **Contract:** Plan → (explicit human approval) → Execute
> **Sibling docs:** [`component-contract.md`](../conventions/component-contract.md) · [`agent-readability.md`](../conventions/agent-readability.md) · [`ds-manifest.json` schema](../conventions/ds-manifest.md)

## 1. Purpose & placement in the workflow

`/extend` adds a **new component** — or a small, coherent set of components — to an **existing** design system, built *to the system's own standard* rather than to a generic template. The whole point of the command is that the additions are indistinguishable, in structure and quality, from what the system already ships: same token discipline, same Component Contract, same parity guarantees, same agent-readable manifest.

It is defined by one thing the other commands do not do: **for every component it first runs a "context of use" interview with the human and dispatches UX/UI pattern research, and only then executes.** The interview grounds the component in *this* product; the research grounds it in established, accessible UI patterns. Execution is downstream of both.

### Relationship to the other commands

| Command | Role | Relationship to `/extend` |
|---|---|---|
| `/setup` | Bootstraps a fresh design system: token pipeline, first components, conventions, manifest. | Runs **once, before** `/extend`. Produces the conventions and `ds-manifest.json` that `/extend` reads and conforms to. |
| `/audit` | Inspects an existing system for drift (token violations, contract gaps, broken parity) and proposes fixes. | Runs **before** `/extend` on an inherited system. `/extend` assumes the system is already conformant — audit-then-fix is the gate that makes "add to the standard" meaningful. |
| `/infra` | Sets up / repairs the build pipeline (Style Dictionary → CSS vars → Tailwind → Storybook) and the agent toolchain. | A **peer**. `/extend` assumes infra is healthy; if the token export or Storybook build is broken, `/infra` runs first. |

`/extend` is the *steady-state* command: `/setup` and `/audit` establish the standard, `/extend` grows the system within it, one reviewed increment at a time.

### Preconditions (hard gate — checked before the interview starts)

`/extend` refuses to proceed unless all hold:

1. **An existing design system** with a populated **`ds-manifest.json`** (component inventory + two-tier token registry + parity map present and non-empty).
2. **Conventions in place** — `component-contract.md` and `agent-readability.md` exist and the manifest declares it follows them.
3. **A healthy token pipeline** — semantic tokens resolve end-to-end (design → Style Dictionary → CSS variables → Tailwind). If not, defer to `/infra`.
4. **A clean-enough tree** — code writes will land on a dedicated git branch (see §6). Uncommitted conflicting work blocks the run.
5. **figc reachable** — `figc status` confirms the daemon and the figc plugin are live on the target Figma file. Figma is the source of truth; without it the command cannot build or bind.

If a precondition fails, `/extend` stops and names the command that resolves it. It never "half-adds" a component.

---

## 2. PLAN PHASE

The plan phase has three stages and produces exactly one artifact: a **written extension plan** ending in a **STOP for approval**. Nothing is created, written, or bound during the plan phase — it is entirely read + reason + propose. Figma, code, tokens, and the manifest are untouched until §3.

```
┌─────────────────────────────────────────────────────────────┐
│ PLAN PHASE  (no writes)                                       │
│                                                               │
│  2a  Context-of-use interview   ── human, one batched round   │
│           │                                                   │
│           ▼                                                   │
│  2b  UX/UI pattern research      ── ux-pattern-researcher     │
│           │        (parallel per component in a batch)        │
│           ▼                                                   │
│  2c  Component proposal + plan   ── synthesize → write plan   │
│           │                                                   │
│           ▼                                                   │
│        ██ STOP — explicit human approval required ██          │
└─────────────────────────────────────────────────────────────┘
```

### 2a. Context-of-use interview

Run by the **`context-interviewer`** agent. Its job is to capture *why this component exists in this product* — the facts research and generic patterns cannot supply. Output is a structured **context-of-use brief** (see schema below).

**Adaptive behavior.** The interviewer classifies the requested component:

- **Well-known type** (Button, Dialog, Tooltip, Checkbox, Combobox, Tabs, …): pre-fill defaults from the established pattern and the system's existing conventions, then ask the human **only to confirm or override** the uncertain points. Do not ask what the pattern already answers.
- **Novel / product-specific type**: ask the full set.

**Batching.** Ask in **one focused round** of grouped questions (a single message the human answers inline), not a slow back-and-forth. A second round is allowed only if an answer materially changes which questions matter.

**Question set** (grouped; adaptive subset is asked):

1. **Purpose** — In one sentence, what is this component *for*? What user job does it do that no existing component covers?
2. **Surfaces & context** — Where does it appear (which pages/regions/layouts)? Marketing surface, dense data/admin surface, or both? Inside forms, toolbars, modals, tables?
3. **Density & frequency** — How often does a user meet it per session? Is it a primary control or an occasional one? Comfortable or compact density?
4. **Variants** — What visual/functional variants are needed (e.g. primary/secondary/ghost; single/multi)? Which is the default?
5. **Sizes** — Which sizes (sm/md/lg)? What is the default? Any surface that dictates a specific size?
6. **States** — Which of: default, hover, focus, active, disabled, loading, error, selected, read-only, empty? Any product-specific state?
7. **Content** — What does it hold (text only, icon+text, media, arbitrary children)? Min/max content? Localization / RTL / long-string behavior?
8. **Interaction model** — Click, type, drag, keyboard-first? Single vs multi-select? Async (loading/optimistic)? What happens on submit/confirm/dismiss?
9. **Accessibility needs** — Keyboard operability expectations, screen-reader announcements, focus management, contrast/target-size constraints, any regulatory bar (e.g. WCAG AA) the product commits to.
10. **Composition** — What existing components must it compose with (fields, forms, menus, popovers)? What must it **not** duplicate? Does an adjacent component already exist that should be extended instead of adding a new one?
11. **Constraints** — Brand rules, platform (web only? responsive breakpoints?), performance budgets, dependencies to avoid.

**Guardrail — de-duplication.** If Q10 surfaces that the system already has a component covering this job, the interviewer stops and recommends `/audit` or a variant addition instead of a new component. `/extend` adds; it does not duplicate.

**Context-of-use brief (captured artifact):**

```yaml
component: Combobox
purpose: "Let users filter a large option set by typing, then pick one."
surfaces: [forms, filter-bars, table-toolbars]        # dense/admin
density: compact
frequency: primary            # met many times per session
variants: [single-select]     # default: single-select
sizes: [sm, md]               # default: md
states: [default, hover, focus, disabled, loading, error, empty-results]
content: "icon(optional) + text label per option; async-loaded options"
interaction: "type to filter; ↑/↓ navigate; Enter select; Esc close; async results"
a11y:
  bar: WCAG-AA
  keyboard: full
  sr: "announce active option + result count"
composes_with: [FormField, Popover, Input]
not_duplicating: [Select]     # Select = short static list; Combobox = typeahead/large/async
constraints: { platform: web, rtl: true, no_new_runtime_deps: true }
open_questions: []
```

### 2b. UX/UI pattern research

Dispatched to the **`ux-pattern-researcher`** subagent (one per component; parallel across a batch). It does **not** invent a design — it retrieves and distills the established, accessible pattern for this component *type* so the proposal rests on convention, not guesswork.

**The accessibility authority is the WAI-ARIA Authoring Practices Guide (APG).** Keyboard interaction, roles, states, and focus behavior are taken from the APG pattern for this component and cited. Where the APG is silent, general WCAG guidance applies.

The researcher must return a concise, **cited** pattern brief containing:

- **Anatomy** — the named parts of the component and how they nest.
- **Standard variants** — the variants the pattern community treats as canonical, and when each is used.
- **Interaction & keyboard behavior** — the full keyboard map and pointer behavior, sourced from the **WAI-ARIA APG** pattern.
- **Accessibility** — required ARIA roles/states/properties, focus management, screen-reader expectations, target-size/contrast notes — **per the APG**, cited to the specific pattern page.
- **Common states** — the states the pattern is expected to express.
- **Do / Don't** — concise, sourced guidance (feeds directly into the Component Contract's Do/Don't blocks).
- **Backing primitive** — the **Radix** primitive that most likely provides the accessible behavior, and the **shadcn/ui** styled source that wraps it. Radix supplies the a11y-correct headless primitive; shadcn supplies the styled, ownable source. The researcher **confirms the right primitive** (name, whether it exists, any gaps) rather than assuming.

**Pattern brief (returned artifact, abridged for Combobox):**

```yaml
component: Combobox
apg_pattern: "Combobox (WAI-ARIA APG)"      # cited authority
anatomy: [trigger/input, listbox popup, option, empty-state, clear-button?]
variants:
  - name: single-select
    when: "one value chosen from a large/typeahead/async set"
keyboard:                                    # per APG Combobox pattern
  - "Down/Up: move active option within listbox"
  - "Enter: select active option, close"
  - "Esc: close listbox (or clear on second press)"
  - "Type: filter options; listbox opens"
  - "Home/End: first/last option"
a11y:
  roles: { input: combobox, popup: listbox, item: option }
  states: [aria-expanded, aria-activedescendant, aria-controls, aria-selected]
  focus: "focus stays on input; active option tracked via aria-activedescendant"
  sr: "announce result count on filter; announce active option"
  citation: "WAI-ARIA APG — Combobox pattern"
states: [default, focus, disabled, loading, error, empty-results]
do:   ["announce result count", "keep focus on the input", "support Esc to close"]
dont: ["move DOM focus into the listbox", "use for short static lists (use Select)"]
backing_primitive:
  radix: "no single Radix 'Combobox'; compose Radix Popover + a command/list primitive"
  shadcn: "shadcn Combobox recipe = Popover + Command (cmdk) + Button trigger"
  confirmed: true
  note: "combobox is a shadcn *composition*, not a one-to-one Radix primitive — flag in plan"
```

### 2c. Component proposal + plan (synthesis → written plan → STOP)

Synthesize the context brief (2a), the pattern brief (2b), and the system's conventions into a single written plan. This is where product reality, established pattern, and house standard meet. The plan contains:

1. **Proposed component API — a draft Component Contract.** Props table (`name / type / default / required / description`), variants, sizes, and states, with **accessibility documented inline per prop**, and structured **Do/Don't** seeded from research. This is the real Contract in draft form, so approval is approval of the actual documentation standard the component will ship with.
2. **Semantic tokens needed.** List every semantic token the component references. For each, state whether it **already exists** in the two-tier system or is **NEW**. Components reference only *semantic* tokens — never primitives, never hex. **New semantic tokens/aliases are flagged explicitly and prominently**, because adding them touches Figma and the token pipeline (see §3 step 0 and §6). No token is ever silently invented.
3. **Figma build plan.** The component and its variant matrix (variant properties, sizes, states), and exactly which **semantic** token binds to which property (fill/stroke/text). No hex, ever.
4. **Code plan.** The confirmed **Radix primitive** (or the shadcn *composition* if there's no 1:1 primitive) + **shadcn** styled source, and how Tailwind classes reference the **semantic** layer (CSS variables), not primitives.
5. **Storybook stories.** Which variants/states become stories, authored as **examples-as-data** so they're both rendered and machine-readable.
6. **Manifest & parity updates.** The new entry in the component inventory and the new rows in the **parity map** (design name ↔ token name ↔ CSS var ↔ Tailwind class ↔ code prop — identity across every hop).
7. **Agent-readability updates.** Which agent-discoverable artifacts get the new component so agents can find and use it.

The plan ends with an explicit **STOP**: a plain-language summary of what will change (files, Figma nodes, and — highlighted — any new tokens), and a request for approval. **No execution occurs until the human approves.**

---

## 3. EXECUTE PHASE (only after explicit approval)

Steps run in this order. Each names the responsible agent. The ordering is load-bearing: **tokens exist before they're bound, Figma is built and verified before code, code exists before it's documented and storied, and parity is verified last.**

**Step 0 — New semantic tokens (only if the plan flagged any).** This is the **only** path in `/extend` that touches tokens. The **`figc-operator`** creates the new **semantic** variables in Figma as **aliases of existing primitives** (never new raw values), binds where appropriate, then triggers a token **re-export / rebuild** (Style Dictionary → CSS variables) so the code layer actually has the new semantic names before any component references them. If the plan flagged no new tokens, skip this step entirely.

**Step 1 — Build the component in Figma.** `figc-operator` creates the component and its variant matrix, and **binds semantic tokens** to every color property — never hardcoded colors, never `resize()` on instances. After building, it runs **`figc shot`** on the component and inspects the screenshot to verify the variants render correctly. Figma is the source of truth; it is built and verified first.

**Step 2 — Generate code.** **`component-generator`** adds the component from the confirmed **Radix primitive / shadcn source**, wired so Tailwind classes reference the **semantic** token layer (CSS variables), matching the Figma variant matrix one-to-one. Writes land on the git branch only (§6).

**Step 3 — Document.** Write the **Component Contract** doc for the component, conforming to the standard: canonical section skeleton (Overview → Usage → Best Practices → Props), full props table with inline per-prop accessibility, structured Do/Don't (from research), examples-as-data. This upgrades the 2c draft Contract to the shipped doc.

**Step 4 — Storybook.** **`storybook-builder`** adds stories for the variants/states as **examples-as-data**, so each documented example is both rendered in Storybook and machine-readable.

**Step 5 — Update the manifest.** Add the component to the **inventory** and add its **parity map** rows in `ds-manifest.json` (design ↔ token ↔ CSS var ↔ Tailwind ↔ code — name identity across every hop).

**Step 6 — Verify parity.** Run **`parity-verifier`** to enforce that every hop in the new parity rows resolves and names match end-to-end. A parity failure blocks completion.

**Step 7 — Update agent-readability artifacts.** Update the machine/agent-consumable artifacts (per `agent-readability.md`) so the new component is **discoverable** by agents — manifest + structured docs reflect it.

**Step 8 — Final verification checkpoint.** Confirm, with evidence, that: `parity-verifier` passes, the `figc shot` of the component looks correct, and the Storybook story renders. Only then is the component reported complete.

---

## 4. Agents dispatched

| Agent | Responsibility | Existing / new |
|---|---|---|
| **`context-interviewer`** | Runs the adaptive, batched context-of-use interview; produces the structured context-of-use brief. | **NEW** |
| **`ux-pattern-researcher`** | Researches the component type's pattern (anatomy, variants, keyboard, states, do/don't); accessibility per **WAI-ARIA APG** (cited); confirms the backing Radix/shadcn primitive; returns a cited pattern brief. | **NEW** |
| `figc-operator` | Creates semantic tokens (if any) + the Figma component/variants; binds semantic tokens; `figc shot` verify. | existing |
| `component-generator` | Generates the Radix/shadcn component wired to the semantic token layer via Tailwind. | existing |
| `storybook-builder` | Adds Storybook stories for variants/states as examples-as-data. | existing |
| `parity-verifier` | Enforces the parity map end-to-end across every hop; blocks on failure. | existing |

**Parallel vs sequential.** Within the plan phase, the **interview (2a) precedes research (2b)** for a single component (the brief may narrow what research must cover) — but across a **batch**, the `ux-pattern-researcher` runs **in parallel, one per component**. The **execute phase is strictly sequential** per component (Steps 0→8), because each step depends on the previous (tokens before bind, Figma before code, code before docs/stories, parity last).

---

## 5. Batch / multi-component note

`/extend Combobox Tabs Tooltip` is valid. Handling:

- **Research fans out in parallel** — one `ux-pattern-researcher` per component. Interviews are still batched with the human (grouped by component, ideally one combined round).
- **Plan and approve as one batch.** A single written plan covers all requested components, with all new tokens for the batch surfaced together. The human approves (or trims) the batch once — one approval gate, not one per component.
- **Execute sequentially, with a parity check between components.** Build/generate/document/story/manifest one component fully (Steps 0–8), run `parity-verifier`, and only then start the next. This keeps the manifest and parity map consistent at every step and localizes any failure to a single component rather than corrupting the batch.
- **Partial approval is fine** — the human may approve some components and defer others; deferred ones simply aren't executed.

---

## 6. Gates & safety

1. **Plan → approve → execute.** The single hard gate. The plan phase performs **no writes** to Figma, code, tokens, or the manifest. Execution begins only on **explicit human approval** of the written plan. This is the plugin-wide contract; `/extend` does not deviate.
2. **New tokens are never silent.** Any new semantic token/alias is **surfaced explicitly in the plan** (flagged, listed, and highlighted in the STOP summary) because it touches Figma and the token pipeline. New tokens are **semantic aliases of existing primitives** — `/extend` never introduces new raw/primitive values or hex. Token creation is the only token-touching path (Step 0) and it runs only if the approved plan called for it.
3. **figc shot-verify checkpoint.** Every Figma change is verified by screenshot (`figc shot` + inspect) before the workflow moves on — after the component build (Step 1) and again at the final checkpoint (Step 8).
4. **Git-branch-only for code writes.** All code generation, docs, and stories land on a **dedicated feature branch**, never directly on the default branch. The human reviews and merges. Figma (source of truth) is edited live via figc; code follows one-way from it.
5. **Token discipline enforced.** Components bind/reference **semantic tokens only** — never primitives, never hardcoded colors. `parity-verifier` (Step 6) is the machine gate that proves name identity across every hop; a failure blocks completion.
6. **No duplication.** If the interview reveals an existing component already covers the job, `/extend` stops and redirects (variant addition or `/audit`) rather than adding a redundant component.

---

## Appendix — Worked mini-example: `/extend Combobox`

**Command:** `/extend Combobox`

**2a — Interview (adaptive; Combobox is well-known → confirm/override only).**
`context-interviewer` pre-fills typeahead defaults and asks the human to confirm surfaces, select-mode, sizes, and the async-vs-static question. Human confirms: dense admin filter-bars and forms, single-select, sm/md (default md), options load async, must compose with `FormField`/`Popover`, must **not** duplicate the existing `Select` (short static lists). → produces the context brief in §2a.

**2b — Research (`ux-pattern-researcher`).** Returns the pattern brief in §2b: APG **Combobox** pattern is the a11y authority (roles `combobox`/`listbox`/`option`; `aria-expanded`, `aria-activedescendant`; focus stays on the input; announce result count; ↑/↓/Enter/Esc/Home/End keymap). **Confirms** there is **no single Radix `Combobox` primitive** — the accessible pattern is a **composition of Radix Popover + a command/list primitive (shadcn's Combobox recipe = Popover + Command/cmdk + Button trigger)**. This is flagged as a composition, not a 1:1 primitive.

**2c — Proposal + plan (STOP).**
- **Draft Contract:** props `options`, `value`, `onChange`, `placeholder`, `size` (`sm|md`, default `md`), `disabled`, `loading`, `emptyMessage` — each with inline a11y notes; Do/Don't seeded from research ("keep focus on the input"; "don't use for short static lists — use `Select`").
- **Semantic tokens:** reuses existing `surface`, `border`, `text`, `text-muted`, `focus-ring`. **NEW (flagged):** `--color-combobox-option-selected-bg` (semantic alias of an existing primitive) for the active/selected option row — highlighted in the STOP summary as a Figma-touching change.
- **Figma plan:** `Combobox` component; variant props `size ∈ {sm, md}` × `state ∈ {default, focus, disabled, loading, empty}`; bind `surface`/`border`/`text`/`focus-ring`/new selected-bg token. No hex.
- **Code plan:** shadcn Combobox composition (Radix Popover + Command); Tailwind classes reference the semantic CSS vars.
- **Stories:** one per state (default/focus/disabled/loading/empty-results) as examples-as-data.
- **Manifest/parity:** new inventory entry + parity rows for each token (design name ↔ CSS var ↔ Tailwind ↔ code prop).
- **STOP** — summary highlights the one new token and requests approval.

**Execute (on approval):**
0. `figc-operator` creates semantic var `combobox-option-selected-bg` (alias of existing primitive), binds it, re-exports tokens (Style Dictionary → CSS vars).
1. `figc-operator` builds the `Combobox` component + variant matrix, binds semantic tokens, `figc shot` → inspect (verifies focus ring, selected-row bg, empty state).
2. `component-generator` adds the shadcn/Radix composition wired to the semantic layer, on the feature branch.
3. Write the Component Contract doc (Overview → Usage → Best Practices → Props; inline a11y; Do/Don't).
4. `storybook-builder` adds the five state stories as examples-as-data.
5. Update `ds-manifest.json` inventory + parity rows.
6. `parity-verifier` runs → all hops resolve.
7. Update agent-readability artifacts so agents can discover `Combobox`.
8. Final checkpoint: parity passes, `figc shot` looks right, Storybook story renders → report complete.
