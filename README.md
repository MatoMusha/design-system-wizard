# design-system-wizard

A **Claude Code plugin** that takes a design system across its whole lifecycle —
**create → audit → extend → generate code** — with **Figma as the source of
truth** (driven through [`figc`](https://github.com/MatoMusha/figc)) flowing
one-way into code: **Style Dictionary → CSS variables → shadcn/ui + Radix +
Tailwind → Storybook**.

Every command follows one contract: **plan → approve → execute**. Nothing is
generated, written, or pushed until you approve the plan.

| Command | What it does |
|---|---|
| `/setup` | Create a design system from scratch (brand interview → tokens → components + docs). |
| `/audit` | Score an existing system against the conventions; return a report + prioritized fix plan. |
| `/extend` | Add a component: context-of-use interview → UX/UI pattern research → build + wire + document. |
| `/infra` | Stand up the design-to-code pipeline with design↔code name-parity as a hard requirement. |

---

## Prerequisites

- **Claude Code** (the CLI, desktop, or IDE extension).
- **[figc](https://github.com/MatoMusha/figc)** — the Figma bridge all Figma work
  runs through. For any command that reads or writes Figma you need:
  - the figc daemon running: `figc serve` (or `node bin/figc.js serve`), and
  - the **figc** plugin open in your target Figma file (Plugins → Development → figc).
  - `figc status` confirms both are connected.
  - `/infra` additionally needs the `figc tokens export` subcommand (ships in figc
    on the `feat/tokens-export` branch onward).
- **Node.js ≥ 18** (for figc and the downstream token build).
- For `/infra`'s generated pipeline: a repo where **Style Dictionary v4**,
  **Tailwind**, **shadcn/ui + Radix**, and **Storybook** can be installed.

---

## Install

### From GitHub (recommended)

The repo is its own plugin marketplace, so two commands install it:

```
/plugin marketplace add MatoMusha/design-system-wizard
/plugin install design-system-wizard@design-system-wizard
```

(`design-system-wizard@design-system-wizard` is `plugin-name@marketplace-name` —
both happen to be the same here.) Then restart the session or run
`/reload-plugins`. Confirm with `/plugin` → **Installed**.

### From a local clone (development)

```bash
git clone https://github.com/MatoMusha/design-system-wizard.git ~/Code/design-system-wizard
```

Either load it directly for a session (no install, fastest for iteration):

```bash
claude --plugin-dir ~/Code/design-system-wizard
```

…or add the local path as a marketplace and install it:

```
/plugin marketplace add ~/Code/design-system-wizard
/plugin install design-system-wizard@design-system-wizard
```

After edits, `/reload-plugins` picks up changes. Validate the manifest with
`claude plugin validate ~/Code/design-system-wizard`.

Once installed, the commands are namespaced — type `/design-system-wizard:setup`
(Claude Code will also surface the short `/setup` form in the picker).

---

## Use

All four commands are two-phase. **Phase 1 (plan)** is read-only: the command
gathers context and presents a written plan, then **stops for your approval**.
**Phase 2 (execute)** runs only after you say go. New semantic tokens are always
surfaced in the plan (never invented silently); code writes go to a branch,
never `main`; every Figma change is verified with a `figc shot`.

### `/setup` — create a design system

```
/setup
```

Interviews you (existing product?, brand/main color, typography direction, tone),
then proposes a two-tier token taxonomy (primitives + a semantic layer named to
match shadcn's identifiers), a component inventory, and the documentation plan.
On approval it builds the Figma variable collections + **typography as real text
styles** + base components, emits DTCG tokens, writes the component docs, and
renders the on-canvas foundation pages (Color, Typography, …).

### `/audit` — audit an existing system

```
/audit [path-to-repo or Figma file]
```

Inventories the design + code, scores against the three conventions — the
**Component Contract Evaluation Checklist**, the **Agent-Readability Score**
(0–100), and the **Canvas Documentation Checklist** — and returns a report plus a
prioritized fix plan. On approval it applies the fixes.

### `/extend` — add a component

```
/extend <component name(s)>      # e.g. /extend Combobox
```

Runs a context-of-use interview, researches the UX/accessibility pattern (WAI-ARIA
APG) and confirms the backing Radix/shadcn primitive, then proposes the component
(API, tokens, Figma/code/Storybook/manifest plan). On approval it builds the Figma
component, generates the shadcn/Radix code wired to semantic tokens, writes the
Component Contract doc, renders the canvas doc frame, adds Storybook stories, and
verifies parity.

### `/infra` — build the design-to-code pipeline

```
/infra [existing-github-repo]
```

Proposes repo + branching strategy and the full pipeline, then stands it up:

```
Figma variables + text styles
  → figc tokens export (DTCG)
  → Style Dictionary v4  (css/variables, outputReferences, per-mode :root/.dark)
  → CSS variables / Tailwind @theme
  → shadcn/ui + Radix components
  → Storybook
```

A **parity-verifier** asserts each token's identifier is identical at every hop
and fails the build on drift.

---

## Conventions (the shared "definition of good")

- **[Component Contract](docs/conventions/component-contract.md)** — typed, human +
  machine documentation standard per component (+ a 24-check evaluation).
- **[Agent-Readability](docs/conventions/agent-readability.md)** — makes the system
  reliably consumable by AI agents; scored 0–100 by `/audit`.
- **[Canvas Documentation Standard](docs/conventions/canvas-documentation.md)** —
  documentation rendered on the Figma canvas, kept in sync with the code docs.

## Docs

- **[Architecture](docs/ARCHITECTURE.md)** — commands, the 20 worker agents,
  shared state, gates, the token spine.
- **[Figma → DTCG export](docs/specs/figma-dtcg-export.prompt.md)** — the
  `figc tokens export` spec.
- **[`/extend` command](docs/specs/extend-command.md)** — interview → research →
  execute.

## Status

**v0.1.0 — scaffolded.** Structure, conventions, specs, and the `figc tokens
export` capability are in place; the agents encode intent and will be iterated on
their first live runs against a real Figma file.

## Related

- **[figc](https://github.com/MatoMusha/figc)** — the Figma CLI bridge this plugin
  drives (kept as its own repo; this plugin points at it, and `figc tokens export`
  is the `/infra` prerequisite).
