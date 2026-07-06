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
   - `@design-system-wizard:token-architect` — propose the **two-tier taxonomy**: a `primitives` collection (raw color/spacing/radius/type scales) and a `semantic` collection that aliases primitives, with semantic names **engineered to emit the exact identifiers the target component library hard-codes** (`--background`, `--foreground`, `--primary`, `--primary-foreground`, `--border`, `--ring`, `--radius`, …), plus Light/Dark modes as the theming axis. **Ground every scale in a proven reference per the Craft & Measurement Standard** — color in **Radix Colors** (12-step, light+dark) or shadcn defaults; spacing on the **4px grid** (Tailwind scale); a modular **type scale** (as text styles) with correct line-heights; radius + a 5-step elevation scale; **icons = Lucide**. Never hand-mix bland ramps. Every proposed semantic token is surfaced by name, with its grounding source cited.
   - `@design-system-wizard:component-planner` — propose the base component inventory (each backed by a Radix/shadcn primitive, icons from **Lucide**) and the Component Contract doc structure each will ship with.
3. **Write the DS plan.** One written document covering: the brand brief; the proposed token taxonomy with **every semantic token named** and its grounding source; the component inventory; the **canonical Figma file structure** (`Cover · Foundations · Components · Patterns`) and the foundation pages to render (Color, Typography, Spacing, Radius, Elevation, **Icons (Lucide)**, + conditional Motion); and the manifest that will be produced. Surface real choices (naming, font, token structure) as questions — do not decide them on the human's behalf.

**STOP. Present the plan and WAIT for the human's explicit approval. Do not create, write, bind, or push anything until the human approves.**

## Execute phase (only after approval)

Run in order — each step depends on the last. Figma access goes through `figc-operator` (the canonical operator) except the canvas-docs step, which `figma-doc-builder` runs itself under the same figc conventions; all code writes land on a git branch, never `main`.

1. `@design-system-wizard:figc-operator` — build the `primitives` and `semantic` variable collections + Light/Dark modes; bind semantic tokens as aliases of primitives (never hardcoded colors); build **typography as real Figma text styles**; import the **Lucide** icon set as SVGs into the Icons foundation; build the base components. **Everything is Figma auto-layout with token-bound gap/padding on the 4px grid — never absolute-positioned, never overlapping** (per the Craft & Measurement Standard). After every canvas write, `figc shot`-verify and inspect.
2. `@design-system-wizard:token-writer` — emit the DTCG token JSON (`primitives.json`, `semantic.light.json`, `semantic.dark.json`, `typography.json`) with aliases preserved as references.
3. `@design-system-wizard:doc-writer` — scaffold the Component Contract docs (code/web surface) per the Component Contract convention.
4. `@design-system-wizard:figma-doc-builder` — render the **canvas documentation** into the canonical file structure (`Cover · Foundations · Components · Patterns`): foundation pages (Color swatch grid bound to the actual variables, Typography specimens from the real text styles, Spacing, Radius, Elevation, **Icons (Lucide)**) and each component as its own auto-layout doc card on an organized Components page. **This layer is mandatory — never skip it, never scatter components across pages, never overlap frames.**
5. **Craft-verification gate — you (this command) own the loop.** Dispatch `@design-system-wizard:craft-reviewer` to `figc shot` + `figc bound` the foundation pages and every component frame and inspect against the **Craft Checklist** (no overlaps; all spacing on-grid/token-bound; radii from scale; type from scale; Lucide icons on-grid; auto-layout everywhere; contrast AA). craft-reviewer is a read-only inspector — it returns a `pass`/`fail` report with per-node **work-orders**, it does not fix or re-shoot itself. On `fail`: dispatch `@design-system-wizard:figc-operator` to apply its work-orders, then **re-invoke craft-reviewer** to re-inspect. Repeat this dispatch→fix→re-inspect loop until it returns `pass`. Do not report success until it does.
6. Write `ds-manifest.json` (brand brief + two-tier taxonomy + component inventory pointing at each Contract + parity map + canvas doc refs) and **validate it against `${CLAUDE_PLUGIN_ROOT}/schema/ds-manifest.schema.json`**.

Report what was built, the manifest path, the craft-review result, and any follow-up (e.g. `/infra` to wire the design-to-code pipeline).
