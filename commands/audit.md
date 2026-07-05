---
description: Audit an existing design system against the three conventions — inventory design + code, score the Contract / Agent-Readability / Canvas checklists, and produce a prioritized fix plan before applying anything.
argument-hint: "[repo path or Figma file]"
allowed-tools: Agent, Read, Write, Edit, Bash
---

Audit an existing design system. `$ARGUMENTS` may name the target (a repo path and/or the Figma file to inspect).

**Load `${CLAUDE_PLUGIN_ROOT}/skills/design-system-conventions/SKILL.md` first** — the three conventions are the rubric you score against, and the figc / plan→approve→execute rules bind here too. Read it, don't reconstruct it.

This command is **plan → approve → execute**. The plan phase only reads and scores — it changes nothing.

## Plan phase (read-only)

1. **Inventory.** Dispatch `@design-system-wizard:ds-scanner` to inventory what exists on both surfaces: Figma variables/modes/text styles/components (via `@design-system-wizard:figc-operator` — the sole Figma interface, `figc status` first) and the code side (DTCG tokens, CSS vars, components, Storybook, `ds-manifest.json`).
2. **Score.** Dispatch `@design-system-wizard:rubric-evaluator` to score against the three conventions and emit:
   - the **Component Contract Evaluation Checklist** results (24 binary PASS/FAIL checks; report failing check numbers with the offending path; note `[blocker]` failures),
   - the **Agent-Readability Score** (0–100, with band and `topFixes`; the `ds-manifest.json` gate),
   - the **Canvas Documentation Checklist** results — **including the staleness gate**: a present-but-stale doc frame (`sourceHash` ≠ current) fails just like a missing one, and any `C-FRESH-1` failure caps the set at non-conformant.
3. **Fix plan.** Dispatch `@design-system-wizard:fix-planner` to turn the findings into a report + **prioritized fix plan** (blockers first, grouped by fix category, each fix naming the surface it touches). Surface any real decisions (naming, token restructuring) as questions rather than deciding them.

**STOP. Present the report + prioritized fix plan and WAIT for the human's explicit approval. Apply no fixes — touch no Figma, code, tokens, or manifest — until the human approves (and may approve a subset).**

## Execute phase (only after approval)

For each approved fix category, dispatch `@design-system-wizard:fix-applier`, which reuses `@design-system-wizard:figc-operator` (Figma fixes — bind semantic tokens, never hardcode, `figc shot`-verify after each write) and `@design-system-wizard:token-writer` (DTCG token fixes). Any new semantic token needed is surfaced first, never invented silently; code writes land on a git branch, never `main`. After applying, annotate `ds-manifest.json` with the audit results and the fixes applied.

Report the before/after scores, which fixes were applied vs deferred, and any remaining blockers.
