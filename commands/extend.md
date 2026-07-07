---
description: Add a component (or a small batch) to an existing design system, built to the system's own standard — context-of-use interview, WAI-ARIA pattern research, then Figma + code + docs + Storybook + parity.
argument-hint: "<component-name…>"
allowed-tools: Agent, Read, Write, Edit, Bash, AskUserQuestion
---

Add a new component — or a small, coherent batch — to an **existing** design system, indistinguishable in structure and quality from what the system already ships. `$ARGUMENTS` = the component(s) requested.

**Load `${CLAUDE_PLUGIN_ROOT}/skills/design-system-conventions/SKILL.md` first.** Follow the full spec at **`${CLAUDE_PLUGIN_ROOT}/docs/specs/extend-command.md`** exactly — it is canonical.

**Preconditions (hard gate, checked before the interview):** the **Figma Readiness Gate** passes (the conventions-skill gate: `primitives` + `semantic` variables resolved in every chosen mode, typography as real text styles, Foundations pages built, core component inventory built — Figma has priority, so a new component is added onto a solid Figma foundation, never before one exists); populated `ds-manifest.json` (inventory + two-tier tokens + parity map); conventions in place; healthy token pipeline (defer to `/infra` if broken); clean-enough tree (code lands on a branch); `figc status` green. Gather the gate facts via `@design-system-wizard:figc-operator` and evaluate the fixed criteria. If any precondition fails, stop and name the command that resolves it (`/setup` for an unsolid Figma file, `/infra` for a broken pipeline) — never half-add a component.

This command is **plan → approve → execute**. The plan phase writes nothing.

## Plan phase (read-only)

For each requested component:

1. **Context-of-use interview — run in this conversation.** You conduct it directly with AskUserQuestion / batched questions, following the `@design-system-wizard:context-interviewer` question set (purpose, surfaces, density/frequency, variants, sizes, states, content, interaction, a11y bar, composition, constraints). Do NOT dispatch it as a subagent. **Adaptive:** for a well-known type (Button, Dialog, Combobox, Tabs…) pre-fill defaults and ask only to confirm/override the uncertain points; for a novel type ask the full set. Ask in **one focused, grouped round**. **De-duplication guardrail:** if the system already covers this job, stop and redirect to a variant addition or `/audit` — `/extend` adds, it does not duplicate.
2. **UX/UI pattern research.** Dispatch `@design-system-wizard:ux-pattern-researcher` (one per component; **parallel across a batch**) for a cited pattern brief: anatomy, canonical variants, full keyboard map + a11y roles/states/focus **per the WAI-ARIA APG** (cited to the specific pattern page), common states, sourced Do/Don't, and the **confirmed backing Radix primitive + shadcn styled source** (flagging any component that is a shadcn *composition* rather than a 1:1 primitive).
3. **Synthesize the component proposal.** One written plan per component (for a batch, **one combined plan**): a **draft Component Contract** (props table with inline per-prop a11y, variants, sizes, states, Do/Don't seeded from research); the **semantic tokens referenced** — for each, state whether it already exists or is **NEW**, and flag every NEW semantic token explicitly and prominently (components reference semantic tokens only, never primitives, never hex); and the Figma build plan, code plan, Storybook plan, and manifest + parity-map rows.

**STOP. Present the plan with any new tokens highlighted in the summary, and WAIT for the human's explicit approval. Nothing is created, bound, generated, or pushed until approval. For a batch the human approves (or trims) once — partial approval is fine.**

## Execute phase (only after approval)

Strictly sequential per component (Steps 0–8 below); across a batch, finish one component fully and run `parity-verifier` before starting the next. Code writes land on a git branch, never `main`.

0. **New semantic tokens (only if the plan flagged any).** `@design-system-wizard:figc-operator` creates them as **aliases of existing primitives** (never new raw values), binds where appropriate, and triggers a token re-export/rebuild so the code layer has the names before anything references them. Skip entirely if none were flagged.
1. **Build in Figma.** `@design-system-wizard:figc-operator` builds the component + variant matrix in **auto-layout with token-bound spacing on the 4px grid** (never absolute-positioned, never overlapping; per the Craft & Measurement Standard), **binds semantic tokens** to every color property (no hex, never `resize()` an instance), uses **Lucide** for any icons, then `figc shot`-verifies the variants.
2. **Generate code.** `@design-system-wizard:component-generator` adds the confirmed Radix primitive / shadcn source, Tailwind classes referencing the **semantic** CSS-variable layer (icons via `lucide-react`), matching the Figma variant matrix one-to-one.
3. **Document.** `@design-system-wizard:doc-writer` upgrades the draft into the shipped Component Contract (fixed section spine, full props table, inline a11y, Do/Don't, examples-as-data).
4. **Canvas doc page.** `@design-system-wizard:figma-doc-builder` renders the component's **own dedicated documentation page** (one page per component, usage-first): **use-case frames** (real instances + explanatory text showing how to use it) plus the variant/state matrix, anatomy callouts, and Do/Don't pairs. Optionally emit a companion **Markdown usage guide** for readability.
5. **Storybook.** `@design-system-wizard:storybook-builder` adds stories for the variants/states as examples-as-data.
6. **Manifest.** Add the inventory entry + parity-map rows (design name ↔ DTCG path ↔ CSS var ↔ Tailwind/React identifier ↔ Storybook story).
7. **Verify parity.** `@design-system-wizard:parity-verifier` runs the **deterministic parity lint** (exit code + JSON drift report) asserting name identity across every hop; a non-zero exit blocks completion.
8. **Craft QA — you (this command) own the loop.** Dispatch `@design-system-wizard:craft-reviewer` to `figc shot` + `figc bound` the component + its doc card and check the **Craft Checklist** (no overlaps, on-grid spacing, radii/type from scale, Lucide icons, auto-layout, contrast AA). It returns a `pass`/`fail` report with per-node **work-orders** — it does not fix or re-shoot itself. On `fail`: dispatch `@design-system-wizard:figc-operator` to apply the work-orders, then **re-invoke craft-reviewer**; repeat until it returns `pass`.
9. **Final checkpoint.** Confirm with evidence: parity passes, craft-review passes, the `figc shot` looks correct, the Storybook story renders. Update agent-readability artifacts so the component is discoverable, then report complete.
