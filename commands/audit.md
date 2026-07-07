---
description: Audit an existing design system against the conventions — inventory design + code, score the Contract / Agent-Readability / Canvas / Craft checklists, and produce a prioritized fix plan before applying anything.
argument-hint: "[repo path or Figma file]"
allowed-tools: Agent, Read, Write, Edit, Bash
---

Audit an existing design system. `$ARGUMENTS` may name the target (a repo path and/or the Figma file to inspect).

**Load `${CLAUDE_PLUGIN_ROOT}/skills/design-system-conventions/SKILL.md` first** — the three conventions are the rubric you score against, and the figc / plan→approve→execute rules bind here too. Read it, don't reconstruct it.

This command is **plan → approve → execute**. The plan phase only reads and scores — it changes nothing.

**Figma has priority — this audit assesses the Figma file first.** Its primary job is to judge the **design source of truth**: is the Figma file solid (the Readiness Gate), and is every component **properly documented** (one page per component, usage-first, with use-case frames)? What is missing or under-documented in Figma is the headline of the report. The code-side surfaces (DTCG, Contracts, Storybook, parity) are scored **only when code exists** (i.e. after `/infra`); before that they are reported as `not-yet-generated`, not as failures.

## Plan phase (read-only)

1. **Inventory.** First, if Figma is reachable, dispatch `@design-system-wizard:figc-operator` (the canonical Figma operator — `figc status` first) to pull the Figma slice: variable collections/modes, text styles, components, and **per-component documentation pages** with node ids. Then dispatch `@design-system-wizard:ds-scanner`, handing it that Figma slice, to scan the code side directly (DTCG tokens, CSS vars, components, Storybook, `ds-manifest.json`) and merge both into one inventory — **flagging whether a code layer exists at all**. If Figma is unreachable, the audit cannot do its primary job: report that and stop. If the code layer is absent, ds-scanner marks the code side `not-yet-generated`.
2. **Figma assessment (primary).**
   - **Readiness Gate.** Evaluate the conventions-skill **Figma Readiness Gate** from the inventory (variables resolved in every chosen mode · typography as real text styles · Foundations pages built · core component inventory built) and report it as the top-line verdict: *solid* (ready for `/infra`) or *not solid*, naming each unmet criterion.
   - **Documentation completeness (the Figma-Documented bar).** For **every** component, check it has its **own documentation page** with **use-case frames** (real instances + explanatory text showing how to use it), a variant/state matrix, anatomy, and Do/Don't — the usage-first standard. Report every component that is **missing a page**, **under-documented** (no use-case frames, no anatomy, no Do/Don't), or **stale** (`sourceHash` ≠ current).
   - **Craft.** Dispatch `@design-system-wizard:craft-reviewer` (read-only inspector) to `figc shot` + `figc bound` the file and return its **Craft Checklist** findings: overlaps/absolute positioning, off-grid spacing, hardcoded/unbound colors, radii/type off-scale, non-Lucide icons, contrast failures.
3. **Score.** Dispatch `@design-system-wizard:rubric-evaluator`, handing it the Figma assessment + craft-reviewer's findings, to emit:
   - the **Canvas Documentation Checklist** results — **including the staleness gate** (a present-but-stale page fails like a missing one; any `C-FRESH-1` failure caps the set at non-conformant) and the **one-page-per-component** structure check,
   - the **Craft Checklist** verdict, with `[blocker]` failures reported,
   - the **Agent-Readability Score** (0–100, with band and `topFixes`; the `ds-manifest.json` gate),
   - **and — only if a code layer exists —** the **Component Contract Evaluation Checklist** results (24 binary PASS/FAIL checks; report failing check numbers with the offending path; note `[blocker]` failures) and a parity-lint check. If no code layer exists, mark these `not-yet-generated` and note that `/infra` produces them.
4. **Fix plan.** Dispatch `@design-system-wizard:fix-planner` to turn the findings into a report + **prioritized fix plan** — **Figma readiness and documentation gaps first**, then craft, then (where code exists) code-side fixes — blockers first, grouped by fix category, each fix naming the surface it touches. Surface any real decisions (naming, token restructuring) as questions rather than deciding them.

**STOP. Present the report + prioritized fix plan and WAIT for the human's explicit approval. Apply no fixes — touch no Figma, code, tokens, or manifest — until the human approves (and may approve a subset).**

## Execute phase (only after approval)

For each approved fix, dispatch `@design-system-wizard:fix-applier` (one invocation per fix, in the plan's dependency order). fix-applier applies the **code-side** categories directly with its own tools — DTCG token JSON, Component Contract docs, component source, manifest — to each owning specialist's standard. It cannot touch Figma itself, so for **Figma/canvas** categories it returns a **work-order**; you (this command) then dispatch `@design-system-wizard:figc-operator` (variable creation/binding — bind semantic tokens, never hardcode, `figc shot`-verify after each write) or `@design-system-wizard:figma-doc-builder` (canvas frame regeneration), and hand the result back to fix-applier to verify the targeted rubric check now passes. Any new semantic token needed is surfaced first, never invented silently; code writes land on a git branch, never `main`. After applying, annotate `ds-manifest.json` with the audit results and the fixes applied.

Report the before/after scores, which fixes were applied vs deferred, and any remaining blockers.
