---
name: skill-forge
description: "Create multi-agent skills from the architecture guidelines."
version: 0.1.0
author: skillrx
license: MIT
platforms: [macos]
metadata:
  hermes:
    tags: [skill-forge, skills, agents, multi-agent, plugins, marketplace, cross-harness, references, sc-dolt]
    related_skills: [agent-invocation, portability-patterns, skill-management, hermes-agent-skill-authoring]
---

# Skill Forge

skill-forge is the single source of truth for **multi-agent skill creation**:
building skills that orchestrate one or more agents, following the
skills/agents architecture in
`references/claude-code-skills-agents-guidelines.md` (copied verbatim from
synaptic-canvas). It is also the **curator of the reference library** those
skills are built from.

It does not re-teach SKILL.md format (`hermes-agent-skill-authoring`) or the
agent-file / Pydantic sidecar contract (`agent-invocation`,
`portability-patterns`). It provides access to the documents that do.

## When to Use

- Creating a multi-agent skill — a skill that orchestrates agents (the two-tier skill/agent architecture).
- Adding a skill, agent, or command to an existing plugin.
- A skill's prose asserts a harness capability ("use the Task tool",
  "Codex can't spawn subagents") — see the drift guardrail below.
- Reconciling the live plugin layouts (sc-publish, atm-bd-orchestration,
  process-porn) into one shape.

Don't use for: a lone Hermes-only SKILL.md with no agents/commands/manifest;
or editing a skill's *content* (that is the target skill's own job).

## The reference library

The reference documents are the forge's raw material. Read the one that
applies; do not inline its content into a skill body (progressive disclosure
— the skill's SKILL.md is a table of contents, not the whole book).

| Document | Role | What it governs | Read when |
|---|---|---|---|
| `references/claude-code-skills-agents-guidelines.md` | **primary** — the architecture to follow | two-tier skill/agent architecture (v0.7) | creating any multi-agent skill |
| `references/dependency-model.md` | supplement | the five dependency classes (D1–D5) | a skill has cross-references to other skills/agents/artifacts |
| `references/harness-capability-matrix.md` | supplement | dated harness capabilities | a skill asserts what a harness can do |

The guidelines document is the canonical architecture; it is copied verbatim
from `synaptic-canvas/docs/`. Note: it uses the term "Task tool" throughout —
that name is now `Agent` in Claude Code; `harness-capability-matrix.md` is the
current authority on harness naming and capability drift.

## The dependency model (summary)

A multi-agent skill's parts refer to each other — and to the world outside —
in ways that drift. Five classes, each with one correct declaration site:

| Class | Depends on | Declared in | Example |
|---|---|---|---|
| D1 skill→skill | another skill's mechanics | `related_skills` + "mechanics live in X" | `.codex` thin adapter → canonical `.claude` skill |
| D2 skill→agent | a named teammate / subagent | agent file + `spawn_policy` / identity | `publisher` + channel workers |
| D3 skill→artifact | an install-time rendered manifest | `install.py` + j2 template | `release/publish-artifacts.toml` |
| D4 skill→binding | a pinned runtime library | install/CI pins the wheel | `sc-compose` bindings |
| D5 skill→harness | a capability of the target harness | the canonical matrix | "Task vs Agent tool", "Codex subagents" |

Full taxonomy with worked examples from the live repos:
`references/dependency-model.md`.

## The drift guardrail

The single most common failure: a skill hardcodes a harness fact that was true
last quarter and is false now. Two live, verified examples:

- **Claude Code** renamed the `Task` tool to the `Agent` tool (v2.1.63).
- **Codex CLI** now supports subagents (headless `codex --yolo exec`,
  parallel by default).

Rule — **never hardcode a harness capability in a skill body.** Either write
harness-agnostic language ("a subagent or child agent, whichever the harness
provides" — atm-bd-orchestration does this correctly), or reference
`references/harness-capability-matrix.md`, which carries a `verified-as-of`
date and a refresh procedure. When a harness ships a change, the matrix is the
ONE file that changes — not every skill that leans on it.

## Anatomy — two distribution models

Pick one per plugin and do not mix them.

### Marketplace distribution (Claude Code `/plugin`, Codex `/plugins`)

```
<repo>/
  .claude-plugin/marketplace.json     # catalog (1 per repo; Codex reads this too)
  plugins/<name>/                     # migrate `packages/` → `plugins/`
    .claude-plugin/plugin.json        # Claude Code manifest
    .codex-plugin/plugin.json         # Codex manifest (+ interface block)
    skills/<skill>/                   # SKILL.md + references/
    agents/*.md                       # named agent files
    .cursor/skills/<skill>/SKILL.md   # Cursor thin adapter → canonical body
    assets/  scripts/  tests/
```

### Vendored-kit distribution (install.py)

Used by sc-publish: the kit is vendored into a consumer repo; `install.py`
renders per-consumer manifests from j2 templates. Cross-harness is expressed
as sibling config dirs (`.claude/`, `.codex/`, `.cursor/`), and the
thin-adapter rule keeps one body canonical.

**Thin-adapter rule:** a cross-harness adapter file is frontmatter plus one
line — "read the canonical `skills/<n>/SKILL.md` completely and follow it" —
with a relative path. Never fork the body across harnesses; forked bodies
drift. (This repo's own `.cursor/skills/skill-forge/SKILL.md` is the example.)

## Procedure — forge a multi-agent skill

1. **Classify dependencies first.** Walk the D1–D5 table; list every
   dependency before writing any body, and write each into its declaration
   site. *Done when every dependency is listed with its class + site.*
2. **Pick the distribution model** and the canonical harness. *Done when the
   model is named and no other model's artifacts are mixed in.*
3. **Author the canonical body once**, as a table of contents pointing into
   `references/`. Keep each SKILL.md under ~110 lines. *Done when the
   canonical body exists and loads.*
4. **Add thin adapters** for the other harnesses — frontmatter + pointer line.
   Only write a native rewrite when the harness genuinely lacks the canonical
   model, and record the reason. *Done when every target harness has an
   adapter or a recorded no-op reason.*
5. **Verify harness capabilities against the matrix**, not from memory. Grep
   the skill for capability claims and reconcile each. *Done when zero
   capability claims are unverified against the dated matrix.*
6. **Write the manifest** (`plugin.json` / `manifest.toml`) with name, version,
   author, and `agents`/`skills` pointers; bump `version` in lockstep. *Done
   when the manifest parses and its pointers resolve.*
7. **Exercise in each harness.** Acceptance is "loaded AND exercised", not
   "files exist". *Done when each target harness loads and runs it once.*

## Pitfalls

1. **Hardcoding "Task tool".** Renamed to `Agent` (Claude Code v2.1.63). The
   v0.7 guidelines reference itself still says "Task tool" — treat
   `harness-capability-matrix.md` as current.
2. **Assuming Codex has no subagents.** It does (headless `codex --yolo exec`,
   parallel). Don't write a degraded Codex path that ignores them.
3. **Forking the body per harness.** Adapters are metadata, not content.
4. **Mixing `plugins/` and `packages/`.** Pick `plugins/`; migrate outliers.
5. **A capability claim with no date.** Every matrix row carries
   `verified-as-of`; a claim without a date is a stale claim waiting to happen.

## Verification

- [ ] Every dependency listed with a class (D1–D5) and a declaration site
- [ ] One distribution model, no mixed artifacts
- [ ] One canonical body; adapters are frontmatter + pointer only
- [ ] `grep -riE "Task tool|can'?t spawn|no subagent|does not support"` returns
      zero unverified capability claims
- [ ] Manifest parses; `agents`/`skills` pointers resolve to real files
- [ ] `version` bumped in lockstep across manifest + frontmatter
- [ ] Each target harness loads AND runs the skill once
- [ ] `name` / `version` / `provides` / `depends_on` / `harnesses` recoverable
      from manifest + frontmatter for sc-dolt
