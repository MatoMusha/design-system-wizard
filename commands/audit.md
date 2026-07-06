---
description: Audit an existing design system against the conventions — inventory design + code, score the Contract / Agent-Readability / Canvas / Craft checklists, and produce a prioritized fix plan before applying anything.
argument-hint: "[repo path or Figma file]"
allowed-tools: Agent, Read, Write, Edit, Bash
---

Audit an existing design system. `$ARGUMENTS` may name the target (a repo path and/or the Figma file to inspect).

**Load `${CLAUDE_PLUGIN_ROOT}/skills/design-system-conventions/SKILL.md` first** — the three conventions are the rubric you score against, and the figc / plan→approve→execute rules bind here too. Read it, don't reconstruct it.

This command is **plan → approve → execute**. The plan phase only reads and scores — it changes nothing.

## Plan phase (read-only)

1. **Inventory.** First, if Figma is reachable, dispatch `@design@design-system-wizard:figc-operator` (the canonical Figma operator — `figc status` first) to pull the Figma slice: variable collections/modes, text styles, components, and canvas doc frames with node ids. Then dispatch `@design-system-wizard:ds-scanner`, handing it that Figma slice, to scan the code side directly (DTCG tokens, CSS vars, components, Storybook, `ds-manifest.json`) and merge both into one inventory. If Figma is unreachable, dispatch ds-scanner code-only and it marks the Figma side `unavailable`.
2. **Score.** First dispatch `@design-system-wizard:craft-reviewer` (read-only inspector) to `figc shot` + `figc bound` the file and return its **Craft Checklist** findings: overlaps/absolute positioning, off-grid spacing, hardcoded/unbound colors, radii/type off-scale, non-Lucide icons, contrast failures. Then dispatch `@design-system-wizard:rubric-evaluator`, handing it craft-reviewer's findings, to score against the three conventions and emit:
   - the **Component Contract Evaluation Checklist** results (24 binary PASS/FAIL checks; report failing check numbers with the offending path; note `[blocker]` failures),
   - the **Agent-Readability Score** (0–100, with band and `topFixes`; the `ds-manifest.json` gate),
   - the **Canvas Documentation Checklist** results — **including the staleness gate**: a present-but-stale doc frame (`sourceHash` ≠ current) fails just like a missing one, and any `C-FRESH-1` failure caps the set at non-conformant.
   - the **Craft Checklist** verdict (from craft-reviewer's findings above), with `[blocker]` failures reported.
3. **Fix plan.** Dispatch `@design-system-wizard:fix-planner` to turn the findings into a report + **prioritized fix plan** (blockers first, grouped by fix category, each fix naming the surface it touches). Surface any real decisions (naming, token restructuring) as questions rather than deciding them.

**STOP. Present the report + prioritized fix plan and WAIT for the human's explicit approval. Apply no fixes — touch no Figma, code, tokens, or manifest — until the human approves (and may approve a subset).**

## Execute phase (only after approval)

For each approved fix, dispatch `@design-system-wizard:fix-applier` (one invocation per fix, in the plan's dependency order). fix-applier applies the **code-side** categories directly with its own tools — DTCG token JSON, Component Contract docs, component source, manifest — to each owning specialist's standard. It cannot touch Figma itself, so for **Figma/canvas** categories it returns a **work-order**; you (this command) then dispatch `@design-system-wizard:figc-operator` (variable creation/binding — bind semantic tokens, never hardcode, `figc shot`-verify after each write) or `@design-system-wizard:figma-doc-builder` (canvas frame regeneration), and hand the result back to fix-applier to verify the targeted rubric check now passes. Any new semantic token needed is surfaced first, never invented silently; code writes land on a git branch, never `main`. After applying, annotate `ds-manifest.json` with the audit results and the fixes applied.

Report the before/after scores, which fixes were applied vs deferred, and any remaining blockers.
