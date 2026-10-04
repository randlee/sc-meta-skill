---
name: marketplace-plugin-authoring
description: "Author marketplace plugins with dependency contracts."
version: 0.1.0
author: skillrx
license: MIT
platforms: [macos]
metadata:
  hermes:
    tags: [plugins, marketplace, claude, codex, cursor, hermes, dependencies, cross-harness, sc-dolt]
    related_skills: [agent-invocation, portability-patterns, skill-management, hermes-agent-skill-authoring]
---

# Marketplace Plugin Authoring

A **marketplace plugin** is a versioned package of agent files, skills, and
commands that loads across Claude Code, Codex, and (optionally) Cursor. This
skill is the single source of truth for building one **when it has
dependencies** — on other skills, on named ATM teammates or subagents, on
rendered manifests, and on harness capabilities that drift over time.

It does NOT re-teach SKILL.md format (`hermes-agent-skill-authoring`) or the
agent-file / Pydantic sidecar contract (`agent-invocation`,
`portability-patterns`). It governs the layer above those: the plugin
package, its dependency declarations, and the guardrail against stale
harness assumptions.

## When to Use

- Building a new marketplace plugin for Claude Code / Codex / Cursor.
- Adding a skill, agent, or command to an existing plugin.
- A plugin's prose asserts a harness capability ("use the Task tool",
  "Codex can't spawn subagents") — this skill pins the canonical, dated
  source for those claims.
- Reconciling the three current layouts (sc-publish, atm-bd-orchestration,
  process-porn) into one shape.

Don't use for: a lone Hermes-only SKILL.md with no agents/commands/manifest;
or editing a skill's *content* (that is the target skill's own job).

## Core idea — dependencies are the hard part

A plugin with no dependencies is just files. The plugins that break are the
ones whose parts refer to each other — and to the world outside the plugin —
in ways that drift. There are five dependency classes, each with ONE correct
declaration site:

| Class | What it depends on | Declared in | Example |
|---|---|---|---|
| D1 skill→skill | another skill's mechanics | `related_skills` + a "mechanics live in X" line | `.codex` thin adapter → canonical `.claude` skill |
| D2 skill→agent | a named teammate / subagent | agent file + `spawn_policy` / identity contract | `publisher` + channel workers |
| D3 skill→artifact | an install-time rendered manifest | `install.py` + j2 template | `release/publish-artifacts.toml` |
| D4 skill→binding | a pinned runtime library | install/CI pins the wheel | `sc-compose` bindings |
| D5 skill→harness | a capability of the target harness | the canonical matrix (below) | "Task vs Agent tool", "Codex subagents" |

Full taxonomy with worked examples from the three live repos:
`references/dependency-model.md`.

## The harness-capability contract (the drift guardrail)

The single most common failure: a plugin hardcodes a harness fact that was
true last quarter and is false now. Two live, verified examples:

- **Claude Code** renamed the `Task` tool to the `Agent` tool (v2.1.63).
  Prose that says "invoke via the Task tool" is stale (process-porn).
- **Codex CLI** now supports subagents (headless `codex --yolo exec`,
  parallel by default). A plugin that assumes "Codex can't launch subagents"
  is stale and silently under-delivers.

Rule — **never hardcode a harness capability in a plugin body.** Either write
harness-agnostic language ("a subagent or child agent, whichever the harness
provides" — atm-bd-orchestration does this correctly), or reference the
canonical matrix at `references/harness-capability-matrix.md`, which carries a
`verified-as-of` date and a refresh procedure. When a harness ships a
capability change, the matrix is the ONE file that changes — not every
plugin that leans on it.

## Anatomy — two distribution models

Pick one per plugin and do not mix them.

### Marketplace distribution (Claude Code `/plugin`)

Used by atm-bd-orchestration and process-porn. The repo IS a marketplace; a
consumer installs via the Claude Code plugin marketplace.

```
<repo>/
  .claude-plugin/marketplace.json     # 1 per repo: lists plugins w/ source pointers
  plugins/<name>/                     # NOTE: process-porn uses packages/ — migrate to plugins/
    .claude-plugin/plugin.json        # manifest: name, version, author, agents[], skills[]
    agents/*.md                       # named agent files (Pydantic sidecars live alongside)
    skills/<skill>/                   # SKILL.md + references/ + scripts/ + templates/
    assets/                           # shared assets (atm-bd-orchestration)
    scripts/
    tests/
```

### Vendored-kit distribution (install.py)

Used by sc-publish. The kit is vendored into a consumer repo; `install.py`
renders per-consumer manifests from j2 templates. Cross-harness is expressed
as sibling config dirs, and the thin-adapter rule keeps one body canonical.

```
plugins/<name>/
  .claude/agents/*.md                 # CANONICAL agents
  .claude/skills/<skill>/             # CANONICAL skills (SKILL.md + ref/ + evals/)
  .codex/skills/<skill>/SKILL.md      # THIN adapter → canonical .claude skill
  .cursor/agents|commands|skills/     # native Cursor adaptations (foreground-only)
  install.py                          # caller-owned JSON → rendered manifests
  release/*.toml.j2                   # channel/artifact contract templates
  manifest.toml                       # (simple release-helper plugins only)
  .github/                            # CI workflows + actions
```

**Thin-adapter rule:** a cross-harness adapter file is frontmatter plus one
line — "Read the canonical `.claude/skills/<n>/SKILL.md` completely and follow
it" — with a relative path. Never fork the body across harnesses; forked
bodies drift. (sc-publish's `.codex/skills/prerelease` is the reference.)

## Procedure — author a plugin

1. **Classify dependencies first.** Walk the D1–D5 table; list every
   dependency before writing any body. Each class has a declaration site —
   write the dependency into its site, never into prose you'll forget to
   update. *Done when every dependency is listed with its declaration site.*
2. **Pick the distribution model** (marketplace vs vendored-kit) and the
   canonical harness (`.claude/` in the kit model; the marketplace's primary
   skill in the marketplace model). *Done when the model is named and no
   other model's artifacts are mixed in.*
3. **Author the canonical body once**, in the canonical harness. Keep each
   SKILL.md under ~110 lines; push depth to `references/`. *Done when the
   canonical body exists and loads.*
4. **Add thin adapters** for the other harnesses — frontmatter + pointer line.
   Only write a native rewrite (Cursor's foreground-only flow) when the
   harness genuinely lacks the canonical model, and record that reason in the
   adapter header. *Done when every target harness has an adapter or a
   recorded no-op reason.*
5. **Verify harness capabilities against the matrix**, not from memory. Grep
   the plugin for capability claims and reconcile each with
   `references/harness-capability-matrix.md`. *Done when zero capability
   claims are unverified against the dated matrix.*
6. **Write the manifest** (`plugin.json` or `manifest.toml`) with name,
   version, author, and the `agents`/`skills` pointers. Bump `version` in
   lockstep everywhere it appears. *Done when the manifest parses and its
   pointers resolve to files that exist.*
7. **Exercise in each harness.** Acceptance is "loaded AND exercised", not
   "files exist". *Done when each target harness loads and runs the plugin
   once.*

## sc-dolt landing (the registry contract)

Each plugin is a versioned unit in the sc-dolt registry. Its machine-readable
surface is: `name`, `version`, `provides` (agents + skills), `depends_on`
(the D1–D5 declarations, by name + version range), and `harnesses`
(claude / codex / cursor / hermes). Author plugins so these five fields are
recoverable from the manifest + frontmatter, and sc-dolt's schema can be
built against them without knowing which plugin "exists this week". This is
the break-the-loop move: the registry is driven by a stable contract, not by
the churn of individual skills.

## Pitfalls

1. **Hardcoding "Task tool".** Renamed to `Agent` (Claude Code v2.1.63).
   Reference the matrix, or write "the subagent tool".
2. **Assuming Codex has no subagents.** It does (headless `codex --yolo exec`,
   parallel). Don't write a degraded Codex path that ignores them.
3. **Forking the body per harness.** Adapters are metadata, not content. A
   `.codex` skill with a full body is a fork that will drift from `.claude`.
4. **Treating Cursor's inline-only stance as permanent law.** It is a current
   design stance; re-verify before asserting it in a new plugin.
5. **Mixing `plugins/` and `packages/`.** Pick `plugins/`; migrate outliers.
6. **A capability claim with no date.** Every matrix row carries
   `verified-as-of`; a claim without a date is a stale claim waiting to happen.

## Verification

- [ ] Every dependency listed with a class (D1–D5) and a declaration site
- [ ] One distribution model, no mixed artifacts
- [ ] One canonical body; adapters are frontmatter + pointer only
- [ ] `grep -riE "Task tool|can'?t spawn|no subagent|does not support"` returns
      zero unverified capability claims
- [ ] Manifest parses; `agents`/`skills` pointers resolve to real files
- [ ] `version` bumped in lockstep across manifest + frontmatter
- [ ] Each target harness loads AND runs the plugin once
- [ ] `name` / `version` / `provides` / `depends_on` / `harnesses` recoverable
      from manifest + frontmatter for sc-dolt
