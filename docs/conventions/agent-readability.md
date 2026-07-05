# Convention: Agent-Readability

**Status:** Normative
**Applies to:** any design system managed by `design-system-wizard`
**Enforced by:** `/audit` (emits the **Agent-Readability Score**)
**Related conventions:** [Component Contract](./component-contract.md) · [Parity Map](./parity-map.md) · [Token Taxonomy](./token-taxonomy.md)

---

## 1. Purpose

**Agent-Readability** is the standard that makes a design system reliably consumable by an AI coding agent. A system is *agent-readable* when an agent can, without human help and without guessing:

1. **Discover** every token and component and how they relate,
2. **Understand** each component's API, types, states, and accessibility contract,
3. **Resolve** a design-side name to the exact code-side identifier it becomes, and
4. **Generate correct code** whose only inputs are the system's own published, machine-readable artifacts.

The governing principle is: **the machine interface is the source interface, not a byproduct of the human one.** Documentation is *data* that both renders for humans and feeds agents. Every guarantee below is testable, and its evaluation lives in §9, the **Agent-Readability Checklist**.

The single shared state file, **`ds-manifest.json`**, is the anchor of this convention. It is the documented entry point (§3), the carrier of token identity (§5), and the index the checklist reads.

---

## 2. Dimensions at a glance

| # | Dimension | One-line rule | Weight |
|---|-----------|---------------|-------:|
| D1 | **Discoverability** | One machine-readable manifest indexes all tokens, components, and their relationships, reachable from one documented entry point. | 22 |
| D2 | **Structured docs** | Docs are typed structured data, not freeform prose; each component conforms to the Component Contract. | 20 |
| D3 | **Token identity & semantics** | Every token has one canonical, stable, semantic identifier that survives design→code, with light/dark modes. | 20 |
| D4 | **Agent guidance artifacts** | Generated CLAUDE.md / AGENTS.md / editor rules exist in the DS repo and in consuming projects. | 12 |
| D5 | **Machine interfaces** | `--json` everywhere, stable documented error codes, examples-as-data, optional dense/MCP mode. | 16 |
| D6 | **Determinism & self-description** | Versioned schema, stable ordering, self-describing outputs. | 10 |
| | | **Total** | **100** |

---

## 3. D1 — Discoverability

### D1.1 Single machine-readable manifest
**Rule.** The system MUST publish exactly one canonical manifest, `ds-manifest.json`, that indexes every token and every component. No agent should need to crawl source files to enumerate the system.

The manifest MUST contain at minimum these top-level keys:

```jsonc
{
  "$schema": "https://design-system-wizard.dev/schema/ds-manifest/1.2.0.json",
  "manifestVersion": "1.2.0",
  "system": { "name": "acme-ds", "version": "4.3.0", "generatedAt": "2026-07-05T09:00:00Z" },
  "entry": { "readFirst": "docs/agents/OVERVIEW.md", "mcp": "acme-ds-mcp", "denseMode": true },
  "tokens": [ /* see D3 / D5 */ ],
  "components": [ /* see D1.2 */ ],
  "relationships": [ /* see D1.3 */ ],
  "parity": { "ref": "ds-parity.json", "coverage": 0.98 }
}
```

- **Compliant:** an agent reads `ds-manifest.json` and enumerates all 214 tokens and 48 components with one file read.
- **Non-compliant:** tokens live only in `theme.css`, components only in a Storybook sidebar; there is no single index, so the agent must scrape and infer.

### D1.2 Component entries are addressable and linked
**Rule.** Each `components[]` entry MUST carry a stable `id`, a human `name`, the code `importPath`, the export symbol, and a `contractRef` pointing to its Component Contract record (D2).

```jsonc
{
  "id": "cmp.button",
  "name": "Button",
  "importPath": "@acme/ui",
  "export": "Button",
  "contractRef": "contracts/button.json",
  "tokensUsed": ["color.action.bg", "radius.control", "space.control.x"],
  "status": "stable"
}
```

- **Compliant:** `contractRef` resolves to a valid Component Contract file; `importPath` + `export` compile.
- **Non-compliant:** entry has a name but no `importPath`, so the agent cannot write a working import.

### D1.3 Explicit relationships
**Rule.** The manifest MUST record token↔component and component↔component relationships as typed edges, so an agent can answer "which components use `color.action.bg`?" or "what does `Dialog` compose?" without inference.

```jsonc
"relationships": [
  { "type": "component-uses-token", "from": "cmp.button", "to": "color.action.bg" },
  { "type": "component-composes", "from": "cmp.dialog", "to": "cmp.button" },
  { "type": "token-aliases", "from": "color.action.bg", "to": "color.brand.600" }
]
```

- **Compliant:** every `tokensUsed` value has a matching `component-uses-token` edge.
- **Non-compliant:** relationships are implied only by reading component source.

### D1.4 One documented entry point
**Rule.** `entry.readFirst` MUST point to a real, committed file that tells an agent, in order: where the manifest is, how to query it, how to run the CLI with `--json`, and where the guidance artifacts live. This is the "read me first for machines" contract.

- **Compliant:** `docs/agents/OVERVIEW.md` exists and links to manifest, CLI usage, and CLAUDE.md.
- **Non-compliant:** `entry.readFirst` is null, or points to a marketing README with no machine instructions.

---

## 4. D2 — Structured docs

### D2.1 Docs are structured data, not freeform prose
**Rule.** Component and token documentation MUST be stored as typed structured nodes (e.g. `prose`, `code`, `list`, `table`, `props`, `a11y`) — a typed node tree that renders to human docs AND is parsed by agents. Freeform MDX where prose and API are indistinguishable is non-conformant.

```jsonc
// docs node tree (excerpt)
{ "kind": "props", "rows": [
  { "name": "variant", "type": "'solid' | 'outline' | 'ghost'", "required": false, "default": "'solid'" },
  { "name": "onClick", "type": "(e: MouseEvent) => void", "required": false }
]}
```

- **Compliant:** an agent reads `props` nodes and gets exact types deterministically.
- **Non-compliant:** prop types are described in an English paragraph ("accepts a variant, usually solid").

### D2.2 Conformance to the Component Contract
**Rule.** Every component's structured doc MUST be a valid **Component Contract** record. This convention does **not** redefine that structure — it *requires conformance to it*. The Component Contract defines the per-component fields (props, variants, states, slots, a11y roles/keyboard, do/don't, code examples). Agent-Readability only asserts: the contract exists, validates against its schema, and is reachable via `contractRef` (D1.2).

- **Compliant:** `contracts/button.json` validates against the Component Contract schema; every prop has a type and every interactive state a keyboard behavior.
- **Non-compliant:** the contract omits the `a11y` block, so an agent cannot know the ARIA role or focus behavior.

### D2.3 Reference vs guide separation
**Rule.** Generated **reference** (API surface: props, tokens, types — mechanically derived) MUST be separated from hand-written **guides** (concepts, when-to-use). An agent MUST be able to select reference-only content to avoid narrative ambiguity.

- **Compliant:** `docs/reference/**` is generated and machine-parsed; `docs/guides/**` is prose for humans; the manifest flags which is which.
- **Non-compliant:** API tables are hand-edited inside a tutorial and drift from the code.

---

## 5. D3 — Token identity & semantics

### D3.1 One canonical identifier per token
**Rule.** Each token MUST have exactly one canonical id (`canonicalName`) that is the single source of truth across every emitted form. The CSS custom property, the JS export, the Figma variable, and the manifest entry all reference the same canonical name.

```jsonc
{
  "canonicalName": "color.action.bg",
  "cssVar": "--color-action-bg",
  "jsPath": "tokens.color.action.bg",
  "figmaVar": "color/action/bg",
  "tier": "semantic",
  "value": { "light": "#1f6feb", "dark": "#4c8dff" },
  "aliasOf": "color.brand.600",
  "deprecated": false
}
```

- **Compliant:** `--color-action-bg`, `tokens.color.action.bg`, and Figma `color/action/bg` all map back to `color.action.bg`.
- **Non-compliant:** CSS says `--btn-blue`, JS says `actionBackground`, Figma says `Action/BG` — three names, no canonical link.

### D3.2 Semantic-first, scale-generated
**Rule.** Tokens MUST expose a **semantic** tier (`action.bg`, `text.muted`, `surface.raised`) layered over a generated **primitive** scale (`brand.50…900`). Agents consume semantic tokens; primitives are the generated backing. Every token declares its `tier` (`primitive` | `semantic` | `component`).

- **Compliant:** a component references `color.action.bg`, which aliases `color.brand.600`.
- **Non-compliant:** components hardcode `color.brand.600` (a primitive) directly, with no semantic layer to reason about intent.

### D3.3 Light/dark awareness
**Rule.** Any mode-varying token MUST carry an explicit per-mode value object (`{ "light": …, "dark": … }`). Mode MUST NOT be inferred from separate, unlinked files.

- **Compliant:** the manifest entry above carries both modes on one record.
- **Non-compliant:** `light.css` and `dark.css` define `--color-action-bg` independently with no shared identity.

### D3.4 Name-parity via the Parity Map
**Rule.** The design→code survival of a token name is proved by the **Parity Map** (`ds-parity.json`), which this convention treats as an *input*, not something it redefines. Agent-Readability requires only that: the parity map exists, `manifest.parity.ref` points to it, and its `coverage` is reported. A token whose Figma variable has no code counterpart (or vice-versa) is an **unparified** token and MUST be listed.

```jsonc
// ds-parity.json (excerpt) — owned by the Parity Map convention
{ "pairs": [
    { "figmaVar": "color/action/bg", "canonicalName": "color.action.bg", "status": "matched" }
  ],
  "unparified": [ { "figmaVar": "color/legacy/accent", "reason": "no-code-token" } ],
  "coverage": 0.98
}
```

- **Compliant:** 98% of Figma variables resolve to a canonical token; the 2% gap is enumerated.
- **Non-compliant:** no parity map, so an agent cannot know whether a Figma variable exists in code.

---

## 6. D4 — Agent guidance artifacts

### D4.1 Guidance generated in the DS repo
**Rule.** The design system repo MUST contain generated `CLAUDE.md` (and/or `AGENTS.md`) plus editor rule files, describing how to discover the manifest, run the CLI, and apply tokens/components. These MUST be generated (from the manifest), not hand-maintained, so they cannot drift.

- **Compliant:** `wizard generate agents` writes `CLAUDE.md`, `AGENTS.md`, `.cursor/rules/ds.mdc` with a "generated — do not edit" header and a manifest hash.
- **Non-compliant:** a hand-written `CLAUDE.md` exists but predates the last three token additions.

### D4.2 Guidance injected into consuming projects
**Rule.** `init --features agents` MUST be able to write guidance artifacts into a *consuming* project, telling that project's agent how to import from the system, which import paths to use, and the rule "never hardcode a value that a token exists for."

- **Compliant:** after `wizard init --features agents`, the app repo has an `AGENTS.md` section pointing to `@acme/ui` and the token import.
- **Non-compliant:** consuming teams copy usage notes by hand from Slack.

### D4.3 Guidance is accurate and dated
**Rule.** Each guidance artifact MUST embed the system version and manifest hash it was generated from, so an agent can detect staleness.

- **Compliant:** header reads `# Generated from acme-ds@4.3.0 (manifest sha256:9f2c…)`.
- **Non-compliant:** no version marker; the agent cannot tell if guidance matches the installed package.

---

## 7. D5 — Machine interfaces

### D5.1 `--json` on every command
**Rule.** Every CLI command MUST support `--json`, emitting a stable, documented shape to stdout with nothing else on stdout (logs go to stderr).

- **Compliant:** `wizard audit --json` prints one JSON object; `wizard tokens list --json` prints an array.
- **Non-compliant:** `--json` still interleaves progress spinners on stdout, breaking parsing.

### D5.2 Stable, documented error codes
**Rule.** Failures MUST return a stable machine error code and non-zero exit status. Codes are documented and versioned; an agent branches on `code`, never on message text.

```jsonc
{ "ok": false, "error": { "code": "E_TOKEN_UNPARIFIED", "message": "…", "hint": "run wizard parity sync", "docs": "…/errors#E_TOKEN_UNPARIFIED" } }
```

- **Compliant:** the error-code table lists `E_TOKEN_UNPARIFIED`, `E_CONTRACT_INVALID`, `E_MANIFEST_STALE`, each with a stable meaning.
- **Non-compliant:** errors are free-text strings that change between releases.

### D5.3 Examples-as-data
**Rule.** Component usage examples MUST be stored as structured, machine-readable records (source + metadata) that both render live for humans and feed agents as verified snippets. Screenshots-only examples are non-conformant.

```jsonc
{ "id": "ex.button.loading", "componentId": "cmp.button",
  "source": "<Button loading>Save</Button>", "lang": "tsx", "renders": true }
```

- **Compliant:** an agent copies `ex.button.loading.source` and it compiles.
- **Non-compliant:** the only example is a PNG in the docs site.

### D5.4 Dense mode and MCP (recommended)
**Rule.** The system SHOULD offer a token-efficient **dense** output (`--dense`) that strips to essentials (ids, types, import paths — no prose) and SHOULD expose an **MCP server** so agents query the system live. These are scored but not mandatory for a passing grade.

- **Compliant:** `wizard tokens list --json --dense` returns `[{"n":"color.action.bg","css":"--color-action-bg"}]`; an MCP server answers `getComponent("Button")`.
- **Non-compliant:** the only machine output is the full verbose manifest, costing an agent thousands of tokens per query.

---

## 8. D6 — Determinism & self-description

### D6.1 Versioned schema
**Rule.** Every emitted artifact (manifest, contract, parity, error) MUST carry a `$schema` and version, and MUST validate against that schema.

- **Compliant:** `manifestVersion: "1.2.0"` and a resolvable `$schema`.
- **Non-compliant:** the manifest shape changes silently between releases with no version bump.

### D6.2 Stable ordering
**Rule.** Array outputs (tokens, components, checks) MUST be deterministically ordered (e.g. sorted by `id`), so repeated runs diff cleanly and agents can cache.

- **Compliant:** two consecutive `--json` runs are byte-identical.
- **Non-compliant:** token order depends on filesystem enumeration and shuffles per run.

### D6.3 Self-describing outputs
**Rule.** Machine outputs MUST identify what they are and how to interpret them — a `kind`/`type` discriminator and the schema ref — so an agent needs no out-of-band knowledge.

- **Compliant:** audit output begins `{ "kind": "agent-readability-report", "schema": "…/report/1.0.0.json" }`.
- **Non-compliant:** a bare array with no envelope, meaning unclear.

---

## 9. Evaluation — the Agent-Readability Checklist

`/audit` runs this rubric and emits an **Agent-Readability Score** from **0–100**. Every check is objective and machine-verifiable against the artifacts above. Each dimension's checks sum to its dimension weight (§2); the dimension is scored as `weight × (points earned / points possible)`, and the total is the sum of dimension scores, rounded to an integer.

Checks are **binary** (0 or full) or **graded** (a `0.0–1.0` ratio × the check's points, used for coverage-style checks).

### D1 — Discoverability (22 pts)
| Check | Type | Pts | Pass condition |
|-------|------|----:|----------------|
| C1.1 | binary | 6 | `ds-manifest.json` exists, validates, single canonical file |
| C1.2 | graded | 6 | Fraction of components with valid `id`+`importPath`+`export`+`contractRef` |
| C1.3 | graded | 5 | Fraction of `tokensUsed` references backed by a `component-uses-token` edge |
| C1.4 | binary | 5 | `entry.readFirst` resolves to a committed machine-instruction file |

### D2 — Structured docs (20 pts)
| Check | Type | Pts | Pass condition |
|-------|------|----:|----------------|
| C2.1 | graded | 7 | Fraction of components whose docs are typed nodes (not freeform MDX) |
| C2.2 | graded | 9 | Fraction of components with a Component Contract that validates against its schema |
| C2.3 | binary | 4 | Reference (generated) and guides (prose) are separated and flagged in the manifest |

### D3 — Token identity & semantics (20 pts)
| Check | Type | Pts | Pass condition |
|-------|------|----:|----------------|
| C3.1 | graded | 6 | Fraction of tokens with one `canonicalName` mapped to css+js+figma forms |
| C3.2 | binary | 4 | A semantic tier exists over a generated primitive scale (`tier` present on all) |
| C3.3 | graded | 4 | Fraction of mode-varying tokens carrying explicit `{light,dark}` values |
| C3.4 | graded | 6 | Parity coverage: `manifest.parity.coverage` (unparified tokens enumerated) |

### D4 — Agent guidance artifacts (12 pts)
| Check | Type | Pts | Pass condition |
|-------|------|----:|----------------|
| C4.1 | binary | 5 | Generated CLAUDE.md/AGENTS.md + editor rules present in the DS repo |
| C4.2 | binary | 4 | `init --features agents` produces consuming-project guidance |
| C4.3 | binary | 3 | Every guidance artifact embeds system version + manifest hash |

### D5 — Machine interfaces (16 pts)
| Check | Type | Pts | Pass condition |
|-------|------|----:|----------------|
| C5.1 | graded | 5 | Fraction of CLI commands supporting clean `--json` (stdout JSON-only) |
| C5.2 | binary | 4 | Documented, versioned error-code table; failures return `code` + non-zero exit |
| C5.3 | graded | 4 | Fraction of components with ≥1 examples-as-data record that renders |
| C5.4 | binary | 3 | `--dense` mode OR MCP server available (either satisfies) |

### D6 — Determinism & self-description (10 pts)
| Check | Type | Pts | Pass condition |
|-------|------|----:|----------------|
| C6.1 | binary | 4 | All artifacts carry `$schema`/version and validate |
| C6.2 | binary | 3 | Repeated `--json` runs are byte-identical (stable ordering) |
| C6.3 | binary | 3 | Outputs carry a `kind`/`type` discriminator + schema ref |

### Score bands
| Score | Band | Meaning |
|------:|------|---------|
| 0–40 | **Not agent-ready** | An agent must guess; expect incorrect code. Blocks release. |
| 41–70 | **Partially agent-ready** | Core discovery works; gaps in contracts, parity, or interfaces cause errors. |
| 71–90 | **Good** | An agent generates correct code for most tasks; minor manual fallbacks. |
| 91–100 | **Excellent** | Fully self-describing; an agent operates unaided. |

### Gate rule
Regardless of total, a system with a failing **C1.1** (no valid manifest) is capped at **band "Not agent-ready"**, because no other check can be trusted without the entry index.

---

## 10. Sample `/audit` output

```jsonc
{
  "kind": "agent-readability-report",
  "schema": "https://design-system-wizard.dev/schema/report/1.0.0.json",
  "system": { "name": "acme-ds", "version": "4.3.0" },
  "score": 84,
  "band": "good",
  "dimensions": [
    { "id": "D1", "label": "Discoverability",        "score": 20.5, "weight": 22,
      "checks": [
        { "id": "C1.1", "pts": 6, "earned": 6, "pass": true },
        { "id": "C1.2", "pts": 6, "earned": 5.6, "ratio": 0.94, "detail": "45/48 components fully addressable" },
        { "id": "C1.3", "pts": 5, "earned": 5, "ratio": 1.0 },
        { "id": "C1.4", "pts": 5, "earned": 4, "pass": true }
      ]},
    { "id": "D2", "label": "Structured docs",         "score": 15.2, "weight": 20 },
    { "id": "D3", "label": "Token identity & semantics","score": 18.1, "weight": 20 },
    { "id": "D4", "label": "Agent guidance artifacts", "score": 9,    "weight": 12 },
    { "id": "D5", "label": "Machine interfaces",       "score": 13.2, "weight": 16 },
    { "id": "D6", "label": "Determinism & self-description","score": 8, "weight": 10 }
  ],
  "topFixes": [
    { "check": "C2.2", "gain": 3.4, "action": "Add Component Contract a11y block to 3 components",
      "code": "E_CONTRACT_INVALID" },
    { "check": "C4.3", "gain": 3.0, "action": "Stamp manifest hash into generated CLAUDE.md",
      "code": "E_MANIFEST_STALE" }
  ]
}
```

The human-readable rendering prints the same numbers plus the band, and lists `topFixes` (checks ordered by score gain per unit of effort) so a team can raise the score fastest.

---

## 11. Relationship to sibling conventions

- **Component Contract** is an *input* to **D2 (Structured docs)**. Agent-Readability does not define the per-component structure; it requires each component to have a valid contract (C2.2) reachable via `contractRef` (C1.2).
- **Parity Map** (`ds-parity.json`) is an *input* to **D3.4 (name-parity)**. Agent-Readability consumes its `coverage` and `unparified` list; it does not define how parity is computed.
- **Token Taxonomy** defines the tier vocabulary (primitive/semantic/component) that D3.2 checks for.
- **`ds-manifest.json`** is the shared spine: it is what D1 indexes, what D3 stores identity in, and what D6 requires to be versioned and self-describing. The Agent-Readability Score is, in effect, a measure of how completely and honestly the manifest and its referenced artifacts describe the system to a machine.
