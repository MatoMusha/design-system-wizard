---
description: Create a design system from scratch — brand interview, token taxonomy, Figma primitives + semantic collections, base components, DTCG tokens, Component Contract docs, canvas foundations, and ds-manifest.json.
argument-hint: "[product or brand name]"
allowed-tools: Agent, Read, Write, Edit, Bash, AskUserQuestion
---

Create a new design system from scratch. `$ARGUMENTS` may name the product or brand.

**Load `${CLAUDE_PLUGIN_ROOT}/skills/design-system-conventions/SKILL.md` first** — it is the definition of good you build to (three conventions, two-tier token spine + parity, figc rules, the universal plan→approve→execute contract). Read it, don't reconstruct it from memory.

This command is **plan → approve → execute**. The plan phase writes nothing to Figma, code, tokens, or the manifest.

## Plan phase (read-only)

1. **Brand interview — run in this conversation.** You conduct it directly with AskUserQuestion / batched questions; do NOT dispatch it as a subagent (a subagent cannot interview the human). Follow the `@design-system-wizard:brand-interviewer` question set: existing product or greenfield; brand / main color; typography (font choices, tone of type); overall tone and personality; light/dark requirement; any hard brand constraints. Ask in one focused, grouped round; a second round only if an answer materially changes what else matters. Capture the answers as a brand brief.
2. **Dispatch the plan-phase workers (parallel — independent):**
   - `@design-system-wizard:token-architect` — propose the **two-tier taxonomy**: a `primitives` collection (raw color/spacing/radius/type scales) and a `semantic` collection that aliases primitives, with semantic names **engineered to emit the exact identifiers the target component library hard-codes** (`--background`, `--foreground`, `--primary`, `--primary-foreground`, `--border`, `--ring`, `--radius`, …), plus Light/Dark modes as the theming axis. Every proposed semantic token is surfaced by name.
   - `@design-system-wizard:component-planner` — propose the base component inventory and the Component Contract doc structure each will ship with.
3. **Write the DS plan.** One written document covering: the brand brief; the proposed token taxonomy with **every semantic token named** (never invented silently later); the component inventory; the canvas foundation pages to render (Color, Typography, Spacing, Radius, Elevation, + conditional Iconography/Motion); and the manifest that will be produced. Surface real choices (naming, font, token structure) as questions — do not decide them on the human's behalf.

**STOP. Present the plan and WAIT for the human's explicit approval. Do not create, write, bind, or push anything until the human approves.**

## Execute phase (only after approval)

Run in order — each step depends on the last. All Figma access goes through `figc-operator`; all code writes land on a git branch, never `main`.

1. `@design-system-wizard:figc-operator` — build the `primitives` and `semantic` variable collections + Light/Dark modes; bind semantic tokens as aliases of primitives (never hardcoded colors); build **typography as real Figma text styles** (not raw variables); build the base components with bound tokens. After every canvas write, `figc shot`-verify the node and inspect the screenshot.
2. `@design-system-wizard:token-writer` — emit the DTCG token JSON (`primitives.json`, `semantic.light.json`, `semantic.dark.json`) with aliases preserved as references.
3. `@design-system-wizard:doc-writer` — scaffold the Component Contract docs (code/web surface) per the Component Contract convention.
4. `@design-system-wizard:figma-doc-builder` — render the **canvas foundation pages** (Color swatch grid bound to the actual variables, Typography specimens from the real text styles, Spacing, Radius, Elevation, …) via `figc-operator`.
5. Write `ds-manifest.json` (brand brief + two-tier taxonomy + component inventory pointing at each Contract + parity map) and **validate it against `${CLAUDE_PLUGIN_ROOT}/schema/ds-manifest.schema.json`**.

Report what was built, the manifest path, and any follow-up (e.g. `/infra` to wire the design-to-code pipeline).
