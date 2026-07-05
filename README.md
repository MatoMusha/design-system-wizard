# design-system-wizard

A Claude Code plugin that takes a design system across its whole lifecycle —
**create → audit → extend → generate code** — with **Figma as the source of
truth** (driven through [`figc`](https://github.com/MatoMusha/figc)) flowing
one-way into code: **Style Dictionary → CSS variables → shadcn/ui + Radix +
Tailwind → Storybook**.

Every command follows the same contract: **plan → approve → execute**. Nothing is
generated or written until you approve the plan.

## Status

**Design/spec phase.** The framework is agreed; implementation is not started.
Everything below the line is specified, not built.

## Commands

| Command | Does |
|---|---|
| `/setup` | Create a design system from scratch (brand interview → tokens → components + docs). |
| `/audit` | Score an existing system against the conventions; return a report + prioritized fix plan. |
| `/extend` | Add a component: context-of-use interview → UX/UI pattern research → build + wire + document. |
| `/infra` | Stand up the design-to-code pipeline with design↔code name-parity as a hard requirement. |

## Conventions (the shared "definition of good")

- **[Component Contract](docs/conventions/component-contract.md)** — typed,
  human + machine documentation standard per component, with a 24-check
  evaluation checklist.
- **[Agent-Readability](docs/conventions/agent-readability.md)** — makes the
  system reliably consumable by AI agents; scored 0–100 by `/audit`.
- **[Canvas Documentation Standard](docs/conventions/canvas-documentation.md)** —
  documentation rendered **on the Figma canvas**: foundation pages (Color,
  Typography specimens from real text styles, …) + a doc frame per component,
  kept in sync with the code docs and checked by `/audit`.

## Key specs

- **[Architecture](docs/ARCHITECTURE.md)** — how the pieces fit: commands,
  agents, state, gates, the token spine.
- **[Figma → DTCG export](docs/specs/figma-dtcg-export.prompt.md)** — the deep
  build-prompt for `figc tokens export`.
- **[`/extend` command](docs/specs/extend-command.md)** — the interview →
  research → execute flow.

## Prerequisites (for `/infra`)

`figc` extended with a `figc tokens export` subcommand (read-only DTCG export of
Figma variables). See the export spec above. Build order:
**figc extension → plugin scaffold → `/setup` → `/audit` → `/extend` → `/infra`.**
