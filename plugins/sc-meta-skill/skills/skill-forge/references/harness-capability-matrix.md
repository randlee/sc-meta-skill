# Harness Capability Matrix (canonical, dated)

This is the single source of truth for "what can each harness do". Plugins
reference it; they never inline a capability fact. Every row carries a
`verified-as-of` date. When a harness ships a change, THIS file is the one
place that changes — then plugins are grepped for stale claims against it.

**Verified 2026-10-04 (skillrx).** Refresh procedure: before authoring a
plugin that leans on a capability not already listed, or whenever a target
harness ships a subagent/tooling change — update the row, bump the date, and
grep every plugin for the stale phrasing (see the grep in the parent SKILL.md
Verification section).

## Subagent / parallel-agent model

| Harness | Spawn primitive | Capability | verified-as-of |
|---|---|---|---|
| **Claude Code** | `Agent` tool (renamed from `Task` in v2.1.63) | subagents + background agents (Multitask) | 2026-10-04 |
| **Codex CLI** | headless subagent: `codex --yolo exec "<prompt>"` | supported; parallel by default (~6); hooks like Claude Code | 2026-10-04 |
| **Cursor** | foreground/inline | treated inline-only by sc-publish kit (no background subagents) — a design stance, re-verify | 2026-10-04 |
| **Hermes** | `delegate_task` (subagents) + `atm send` (named teammates) | subagents, named teammates, `kanban_*` board, cron | 2026-10-04 |

## Stale-phrasing index

Greppable markers that have caused or will cause drift. Keep this list in
lockstep with the matrix.

| Stale phrasing | Correct | Since |
|---|---|---|
| "Task tool" | `Agent` tool (Claude Code) | v2.1.63 |
| "Codex can't / cannot spawn subagents" | headless subagents supported | ~2025 |
| "Codex has no hooks" | Codex supports hooks like Claude Code | ~2025 |

## Rules

1. Prefer harness-agnostic wording in plugin bodies: "a subagent or child
   agent, whichever the harness provides" survives every row change.
2. When a capability MUST be named, reference this matrix by path; do not
   restate the fact in the body.
3. A capability claim without a `verified-as-of` date is a stale claim waiting
   to happen — reject it in review.
