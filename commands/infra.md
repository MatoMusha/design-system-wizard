---
description: Build the design-to-code pipeline — Figma → DTCG → Style Dictionary → CSS variables → shadcn/Radix + Tailwind → Storybook, with string-identity parity verified at every hop.
argument-hint: "[existing GitHub repo]"
allowed-tools: Agent, Read, Write, Edit, Bash
---

Build (or repair) the one-way design-to-code pipeline. `$ARGUMENTS` may be an existing GitHub repo to build into.

**Load `${CLAUDE_PLUGIN_ROOT}/skills/design-system-conventions/SKILL.md` first** — the two-tier token spine, the parity requirement, and the figc / plan→approve→execute rules govern this pipeline. The full export mechanics live in **`${CLAUDE_PLUGIN_ROOT}/docs/specs/figma-dtcg-export.prompt.md`**.

**Prerequisite:** the pipeline's first hop needs the **`figc tokens export`** subcommand. If figc does not have it, surface that in the plan (it is the build spec above) — `/infra` cannot export tokens without it.

This command is **plan → approve → execute**. The plan phase writes nothing and pushes nothing.

## Preflight — the Figma Readiness Gate (blocking, runs first)

**Figma has priority: code follows a solid Figma file, never precedes it.** Before the plan phase, assert the **Figma Readiness Gate** (defined in the conventions skill). Dispatch `@design-system-wizard:figc-operator` (`figc status` first) to gather the facts, and evaluate the fixed pass/fail criteria: `primitives` + `semantic` variable collections with every semantic role resolved in **every chosen mode**; typography as **real text styles**; the **Foundations** pages built; and the **core component inventory** built as real Figma components. This is deterministic, not a judgment call.

**If the gate fails, STOP** and report exactly which criteria are unmet, pointing the human back to `/setup` (or `/audit` to see the gaps). Do not export tokens, generate code, provision a repo, or plan the pipeline against an unsolid file — code is an outcome of a solid Figma foundation, not a substitute for one.

## Plan phase (read-only) — only if the gate passed

**First, establish the target repository** (ask up front, one question): **is there an existing repo to build into?**
- **No** → `@design-system-wizard:repo-strategist` plans a **fresh repo from scratch** (provisioning proposal + branching convention). Provision nothing yet — it is proposed in the plan for approval.
- **Yes** → ask the human for the repo (path or GitHub remote), then `@design-system-wizard:repo-strategist` **inspects it first-hand** and **suggests a plan that fits it** (where the pipeline slots into the existing structure, what already exists, the branching convention).

Then dispatch both plan-phase workers (parallel — independent):

- `@design-system-wizard:repo-strategist` — carries out the repo decision above: **assess the existing repo** (Yes) or **plan the new repo** (No), plus the **branching convention** (all code writes land on a branch, never `main`). Surface this as a real choice, don't decide silently.
- `@design-system-wizard:pipeline-designer` — design the **full chain** with the **parity contract explicit**: Figma variables/text styles → `figc tokens export` (DTCG, aliases preserved) → Style Dictionary v4 → CSS variables → shadcn/Radix + Tailwind → Storybook, and the per-token parity tuple (Figma name ↔ DTCG path ↔ CSS `--var` ↔ Tailwind/React identifier ↔ Storybook story) that will be asserted end-to-end.

Write the pipeline plan, including the `figc tokens export` prerequisite check and any real decisions (repo, branching, mode/theming strategy) as questions.

**STOP. Present the plan and WAIT for the human's explicit approval. Provision nothing, generate nothing, and push nothing until the human approves.**

## Execute phase (only after approval)

Run in order; code writes land on the agreed branch, never `main`.

1. `@design-system-wizard:token-exporter` — runs `figc tokens export` itself (the read-only export worker, under the figc conventions) to produce the DTCG JSON (`primitives.json`, `semantic.light.json`, `semantic.dark.json`), aliases preserved as references.
2. `@design-system-wizard:sd-builder` — configure Style Dictionary **v4**: `css/variables` format, `outputReferences: true` (so aliases emit `var(--…)` chains, not inlined values), `name/kebab` transform, and per-mode output → `:root` (light) and `.dark` (dark) blocks.
3. `@design-system-wizard:component-generator` — wire shadcn/Radix components to the **semantic** token layer via Tailwind (semantic CSS variables, never primitives, never hex).
4. `@design-system-wizard:storybook-builder` — build Storybook stories as examples-as-data.
5. `@design-system-wizard:parity-verifier` — assert **string-identity at every hop** across the parity map; **fail the build on any drift**. Wire it as a **deterministic parity lint**: a reproducible, CI-runnable check emitting an **exit code + structured JSON drift report** (not only an agent invocation), and add it as a repo lint/CI step so drift breaks the build wherever the pipeline runs.

Then update `ds-manifest.json` (parity map + pipeline metadata) and report the chain status, the parity lint result (exit code), and the branch to review.
