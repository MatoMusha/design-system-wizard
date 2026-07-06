# design-system-wizard — Architecture

A Claude Code plugin that takes a design system across its whole lifecycle:
**create → audit → extend → generate code**. Figma is the source of truth,
driven through **figc** (a CLI + Plugin-API bridge — no REST token, no cloud),
flowing **one-way** into code (Style Dictionary → CSS variables → shadcn/ui +
Radix + Tailwind → Storybook).

> **Status:** design/spec phase. Nothing is built yet. The four command specs
> and two conventions in this repo are the agreed framework; implementation is
> gated on approval.

---

## 1. Principles (non-negotiable)

1. **Plan → approve → execute.** Every command has a read-only *plan phase* that
   ends in a written plan and a **STOP**. Nothing is generated, written, or
   pushed until the human explicitly approves. This holds for `/setup`,
   `/audit`, `/extend`, and `/infra` equally.
2. **No arbitrary decisions.** Where a real choice exists (naming, tooling, repo
   strategy, token structure), it is surfaced in the plan and asked — never
   decided silently. Clarifying questions are batched into one focused round.
3. **Figma is the source of truth.** Tokens flow design → code, one way. Code is
   generated; the pipeline never writes tokens back into Figma.
4. **Design ↔ code name-parity is a hard requirement.** A token's identifier
   must survive every hop unchanged. This is verified, not hoped for.
5. **The design system must be agent-readable.** Authored to a machine-consumable
   standard and scored for it (see the Agent-Readability convention).

---

## 2. The four commands

Each is a plan→approve→execute skill that dispatches worker agents.

### `/setup` — create a design system from scratch
- **Plan:** `brand-interviewer` (context interview: existing product, brand/main
  color, typography, tone) → `token-architect` (proposes the two-tier taxonomy:
  primitives + semantic scales) → `component-planner` (component inventory + the
  Component Contract doc structure). → written DS plan, STOP.
- **Execute (on approval):** `figc-operator` builds variable collections/modes +
  base components (auto-layout only, token-bound spacing, Lucide icons — never
  overlapping), and builds **typography as real Figma text styles** (not raw
  variables); `token-writer` emits DTCG token JSON; `doc-writer` scaffolds
  Component Contract docs; `figma-doc-builder` renders the **canvas documentation**
  into the canonical file structure; `craft-reviewer` runs the **craft-QA loop**
  until the Craft Checklist passes. Writes `ds-manifest.json`.

### `/audit` — audit an existing design system
- **Plan:** `ds-scanner` inventories what exists (Figma vars via `figc-operator`
  + code tokens/components) → `rubric-evaluator` scores against the shared
  conventions, producing the **Component Contract Evaluation Checklist** results,
  the **Agent-Readability Score**, and the **Canvas Documentation Checklist**
  (including the staleness gate) → `fix-planner` returns a report + prioritized
  fix plan. → STOP.
- **Execute (on approval):** `fix-applier` agents run per fix category, reusing
  `figc-operator` / `token-writer`. Annotates `ds-manifest.json`.

### `/extend` — grow an existing system by a component *(→ [spec](specs/extend-command.md))*
The distinguishing flow: **context-of-use interview → UX/UI pattern research →
execute**, conforming to the system's standard.
- **Plan:** `context-interviewer` (adaptive, batched interview → context brief) →
  `ux-pattern-researcher` (common patterns, anatomy, keyboard/a11y per WAI-ARIA
  APG, confirms backing Radix/shadcn primitive) → synthesized component proposal
  (draft Component Contract + tokens + Figma/code/Storybook/manifest plans). →
  STOP. New semantic tokens are surfaced explicitly, never invented silently.
- **Execute (on approval):** optional new semantic-token step → `figc-operator`
  builds the component + variants (bound tokens, `figc shot`-verify) →
  `component-generator` wires the Radix/shadcn component to semantic tokens
  (Lucide icons) → Component Contract doc → `figma-doc-builder` renders the
  component's **canvas doc card** → `storybook-builder` stories → `craft-reviewer`
  QA loop → manifest + parity update → `parity-verifier`.

### `/infra` — build the design-to-code pipeline *(→ [export spec](specs/figma-dtcg-export.prompt.md))*
- **Plan:** `repo-strategist` (existing GitHub repo vs provision new; branching)
  → `pipeline-designer` (the full chain with the parity contract explicit). →
  STOP.
- **Execute (on approval):** `token-exporter` (figc → DTCG JSON) → `sd-builder`
  (Style Dictionary v4: `css/variables`, `outputReferences: true`, `name/kebab`)
  → `component-generator` (shadcn/Radix wired to semantic tokens) →
  `storybook-builder` → `parity-verifier` asserts string-identity at every hop.

---

## 3. The conventions (the shared "definition of good")

Referenced by all four commands: `/setup` builds *to* them, `/audit` scores
*against* them, `/extend` conforms new work *to* them, `/infra` verifies *them*.

- **[Component Contract](conventions/component-contract.md)** — a typed, dual
  (human + machine) content model documenting each component: props table schema
  (`name/type/default/required/description/slotElements/a11y`), accessibility
  documented **inline per prop** (mandated against WAI-ARIA APG), structured
  Do/Don't blocks, a fixed section spine (Overview → When to Use → Usage → Best
  Practices → Props → Related), examples-as-data (render live *and* feed agents),
  semantic-tokens-only rule. Ships with the **Component Contract Evaluation
  Checklist** — 24 binary, machine-checkable PASS/FAIL checks `/audit` runs.

- **[Agent-Readability](conventions/agent-readability.md)** — makes the system
  reliably consumable by AI agents, across six weighted dimensions:
  Discoverability (one `ds-manifest.json`), Structured docs (Component Contract
  conformance), Token identity & semantics (parity, semantic tiers, light/dark),
  Agent guidance artifacts (generated CLAUDE.md/AGENTS.md/editor rules),
  Machine interfaces (`--json` + stable error codes, examples-as-data, optional
  dense/MCP), Determinism & self-description. Scored 0–100 with four bands and a
  manifest gate; `/audit` emits an `agent-readability-report` with `topFixes`.

- **[Canvas Documentation Standard](conventions/canvas-documentation.md)** — the
  system has **two documentation surfaces**: the *code/web docs* (Component
  Contract → Markdown/Storybook) and the *canvas docs* — documentation frames
  rendered **on the Figma canvas**. Defines a Foundations area with a doc frame
  per foundation — **Color** (token-bound swatch grid), **Typography** (a
  specimen per real Figma text style), Spacing, Radius/Shape, Elevation, and
  conditional Iconography/Motion — plus a component doc frame per component
  (real placed instances across variants, anatomy callouts, Do/Don't pairs).
  Both surfaces are projections of one source (`ds-manifest.json` + Contract),
  kept in sync by a `sourceHash` freshness rule. Ships the **Canvas
  Documentation Checklist** (with a staleness gate) that `/audit` runs.

- **[Craft & Measurement Standard](conventions/craft-and-measurement.md)** — how
  the system becomes *actually good*, not just good-looking. **Inherit taste,
  don't invent it:** ground color in **Radix Colors** (or shadcn defaults),
  spacing on a **4px grid** (Tailwind scale), a modular **type scale**
  (Inter/Geist), a radius + 5-step elevation scale, and **icons = Lucide**.
  **Pixel-perfect / auto-layout only:** every measurement is a spacing token;
  nothing is absolute-positioned or overlapping. **Craft-verification loop:**
  `figc shot` → inspect → fix → re-shoot until the **Craft Checklist** passes.
  Owned by the `craft-reviewer` agent; enforced by `/setup`, `/extend`, `/audit`.

> These distill widely-used best practices into an original, self-owned
> standard. No third-party design system is cited as a *style* source; Radix,
> Tailwind, shadcn, and Lucide are named only as the technical foundations the
> system is built on.

---

## 4. The token spine (two-tier + parity)

```
Figma  primitives collection (raw scales)  ─┐
Figma  semantic collection (aliases,        │  figc tokens export
       names engineered to shadcn's         │  (read-only, DTCG,
       fixed identifiers)                    ▼   aliases preserved)
                              tokens/*.dtcg.json   (primitives.json,
                              semantic.light.json, semantic.dark.json)
                                             ▼  Style Dictionary v4
                              css: --background --primary --primary-foreground …
                                             ▼  Tailwind @theme / config
                              shadcn/ui + Radix components  ──►  Storybook
                                             ▲
                              parity-verifier: string-identity at every hop,
                              fail the build on drift
```

- **Two-tier:** designers think in a `semantic` layer that aliases `primitives`.
  Semantic names are engineered to emit exactly the identifiers shadcn hard-codes
  (`--background`, `--primary`, `--primary-foreground`, `--border`, `--ring`,
  `--radius`, …) — because shadcn will not adapt to your names; your names must
  produce its.
- **Modes = theming axis:** a collection's `Light`/`Dark` modes export to
  per-mode files → Style Dictionary builds `:root` and `.dark` blocks.
- **Typography = Figma text styles.** Type is authored as real Figma **text
  styles**, exported as DTCG `typography` **composite** tokens (via
  `getLocalTextStylesAsync()`), with any variable-bound fields preserved as
  references. The export is **dual**: it also emits the atomic type primitives
  (font-family/size/line-height/letter-spacing/weight) as ordinary tokens — the
  composite `font` shorthand can't carry every property (e.g. letter-spacing),
  so the atomics are what give Tailwind utilities + shadcn parity their flat
  `--font-*` custom properties.
- **Parity map** (in `ds-manifest.json`): per token, the tuple
  `Figma var name ↔ DTCG path ↔ CSS --var ↔ Tailwind/React identifier ↔
  Storybook story`. The `parity-verifier` reads it as ground truth.

The full export mechanics (DTCG `$type` mapping, alias resolution, modes
strategy, the two-stage implementation, worked example, acceptance criteria)
live in **[figma-dtcg-export.prompt.md](specs/figma-dtcg-export.prompt.md)** —
this is also the build-prompt for extending figc with `figc tokens export`.

---

## 5. Shared state — `ds-manifest.json`

The single artifact that makes four commands one system. Carries the brand brief,
the two-tier token taxonomy, the component inventory (each entry pointing at its
Component Contract doc), and the parity map. `/setup` writes it, `/audit`
annotates it, `/extend` appends to it, `/infra` consumes and extends it. It is
also the primary entry point that satisfies Agent-Readability's Discoverability
dimension.

---

## 6. Agents roster

| Agent | Used by | Responsibility |
|---|---|---|
| `figc-operator` | all | **Canonical** Figma operator + owner of the figc conventions (preconditions `figc status`, bind semantic tokens, never resize, `figc shot`-verify). Two specialized workers also run `figc` directly under these conventions — `figma-doc-builder` and `token-exporter`; every other agent stays off Figma (its Figma work is dispatched to `figc-operator` by the command). |
| `brand-interviewer` | setup | Structured brand/context interview. |
| `token-architect` | setup | Proposes two-tier token taxonomy + scales. |
| `component-planner` | setup | Component inventory + doc plan. |
| `token-writer` | setup, audit | Emits/edits DTCG token JSON. |
| `doc-writer` | setup, extend | Writes Component Contract docs (code/web surface). |
| `figma-doc-builder` | setup, extend, audit | Renders **canvas documentation** (foundation pages + component doc frames) from the manifest, running `figc` itself under the figc conventions. |
| `ds-scanner` | audit | Inventories an existing DS (design + code). |
| `rubric-evaluator` | audit | Runs both convention checklists → scores. |
| `fix-planner` / `fix-applier` | audit | Prioritized fix plan and its application. |
| `context-interviewer` | extend | Adaptive context-of-use interview. |
| `ux-pattern-researcher` | extend | UX/a11y pattern research (WAI-ARIA APG), confirms primitive. |
| `token-exporter` | infra | Runs `figc tokens export` → DTCG. |
| `sd-builder` | infra | Style Dictionary config + build. |
| `component-generator` | extend, infra | shadcn/Radix component wired to semantic tokens. |
| `storybook-builder` | extend, infra | Storybook stories (examples-as-data). |
| `parity-verifier` | extend, infra, audit | Guardian of design↔code name parity. |
| `craft-reviewer` | setup, extend, audit | Visual-QA loop: `figc shot` → inspect against the Craft Checklist (overlaps, off-grid, hierarchy, contrast) → loop fixes through `figc-operator` until it passes. |

---

## 7. Plugin layout (as scaffolded, v0.1.0)

```
design-system-wizard/
  .claude-plugin/plugin.json                                 ← manifest (name auto-namespaces components)
  commands/   setup.md · audit.md · extend.md · infra.md     ← what you type; each holds the
                                                                full plan→approve→execute procedure
  agents/     20 workers (figc-operator · … · parity-verifier)
  skills/     design-system-conventions/SKILL.md             ← the shared "definition of good"
                                                                (loads the 3 convention docs + figc rules)
  schema/     ds-manifest.schema.json                        ← shared-state contract
  docs/       ARCHITECTURE.md · conventions/ · specs/
```

Refinement from the earlier proposal: the plan→approve→execute procedure lives in
each **command** (the thing you type), and the shared standards live in a single
**`design-system-conventions` skill** — rather than a parallel `skills/ds-*` per
command, which would have duplicated the command bodies. Commands reference worker
agents as `@design-system-wizard:<agent>`; the model dispatches them. Conventions
are the `docs/conventions/` files, surfaced through the skill (no separate
`rubric/` copy).

External dependency: extending **figc** with `figc tokens export` is a
prerequisite for `/infra`. Build order:
**figc extension → plugin scaffold → `/setup` → `/audit` → `/extend` → `/infra`.**

---

## 8. Gates & safety

- Two-phase (plan → STOP → execute) on every command.
- `figc shot`-verify checkpoint after every Figma write.
- Git-branch-only for any code write — never commit to `main` directly.
- New semantic tokens are always surfaced in the plan; never invented silently.
