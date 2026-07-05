---
name: design-system-conventions
description: The shared "definition of good" for design-system-wizard — load before building, auditing, extending, or generating code for a design system. Points at the three conventions (Component Contract, Agent-Readability, Canvas Documentation Standard), the two-tier token + parity model, the DTCG export contract, and the figc conventions. Every command and worker agent conforms to these.
---

# Design System Conventions

This skill is the single source of the standards every design-system-wizard
command builds to, scores against, and verifies. Load it whenever you create,
audit, extend, or generate code for a design system. Read the relevant document
below rather than reproducing its contents from memory — the docs are canonical.

## The three conventions (the shared "definition of good")

1. **Component Contract** — `${CLAUDE_PLUGIN_ROOT}/docs/conventions/component-contract.md`
   Typed, dual (human + machine) documentation standard per component: props
   table schema (`name/type/default/required/description/slotElements/a11y`),
   accessibility documented **inline per prop** (against WAI-ARIA APG),
   structured Do/Don't, fixed section spine (Overview → When to Use → Usage →
   Best Practices → Props → Related), examples-as-data, semantic-tokens-only.
   Evaluated by the **Component Contract Evaluation Checklist** (24 binary checks).

2. **Agent-Readability** — `${CLAUDE_PLUGIN_ROOT}/docs/conventions/agent-readability.md`
   Makes the system reliably consumable by AI agents across six weighted
   dimensions (Discoverability, Structured docs, Token identity & semantics,
   Agent guidance artifacts, Machine interfaces, Determinism & self-description).
   Scored 0–100 with four bands and a manifest gate; `/audit` emits an
   `agent-readability-report`.

3. **Canvas Documentation Standard** — `${CLAUDE_PLUGIN_ROOT}/docs/conventions/canvas-documentation.md`
   Documentation rendered **on the Figma canvas**: foundation pages (Color,
   Typography specimens from real text styles, Spacing, Radius, Elevation, +
   conditional Iconography/Motion) and a doc frame per component. Canvas and
   code docs are two projections of one source, kept in sync by a `sourceHash`
   freshness rule. Evaluated by the **Canvas Documentation Checklist** (with a
   staleness gate).

## The token spine (two-tier + parity)

- **Two-tier tokens:** a `primitives` collection (raw scales) and a `semantic`
  collection that aliases primitives; semantic names are engineered to emit the
  exact identifiers shadcn hard-codes (`--background`, `--primary`,
  `--primary-foreground`, `--border`, `--ring`, `--radius`, …).
- **Typography = Figma text styles** exported as DTCG `typography` composite
  tokens, with a **dual output** of atomic type primitives for Tailwind/shadcn
  parity. Full mechanics: `${CLAUDE_PLUGIN_ROOT}/docs/specs/figma-dtcg-export.prompt.md`.
- **Modes** (Light/Dark) are the theming axis → per-mode DTCG files → SD `:root`
  and `.dark` blocks.
- **Parity is a hard requirement:** a token's identifier must survive every hop
  (Figma name → DTCG path → CSS `--var` → Tailwind/React identifier → Storybook)
  unchanged. The **parity map** in `ds-manifest.json` records it; `parity-verifier`
  fails the build on drift.

## figc conventions (all Figma access)

- Preconditions first: `figc status` (daemon + plugin connected).
- **Never hardcode colors** — bind semantic tokens (`figc bind`).
- **Never resize instances** — use `figc place`.
- After any canvas change, **`figc shot`** the node and inspect it.
- All Figma reads/writes go through the `figc-operator` agent — no other agent
  touches Figma directly.

## The universal contract

Every command is **plan → approve → execute**. The plan phase is read-only and
ends in a written plan + STOP. Nothing is generated, written, or pushed until the
human explicitly approves. New semantic tokens are always surfaced in the plan,
never invented silently. Code writes go to a branch, never `main`.

## Shared state

`ds-manifest.json` (schema: `${CLAUDE_PLUGIN_ROOT}/schema/ds-manifest.schema.json`)
is the single artifact tying the commands together and the primary
machine-readable entry point for Agent-Readability. `/setup` writes it, `/audit`
annotates it, `/extend` appends to it, `/infra` consumes and extends it.
