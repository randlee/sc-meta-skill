# Cursor — reference

Cursor has **no plugin marketplace** and no plugin manifest. It reads
repo-local `.cursor/` config dirs directly. Verified via `randlee/sc-publish`
(`.cursor/` tree) and the `feature/cursor-integration` branch of sc-publish.

## Layout

```
.cursor/
  agents/<name>.md        # named agent files
  commands/<name>.md      # slash commands
  skills/<name>/SKILL.md  # skills
```

## Runtime model (differs from Claude Code / Codex)

- **Foreground-only**: one foreground agent; no background subagents and no
  Multitask-style delegation.
- Channel/agent files are **inline playbooks**, not spawned subagents.
- sc-publish's `.cursor/agents/publisher.md` forbids "spawning Task subagents"
  and "running as a Multitask Mode background worker" — the Cursor flow runs
  every channel step inline, sequentially.

## SKILL.md frontmatter (Cursor)

Cursor skills use a `description` frontmatter (like Claude). A Cursor adapter
for a cross-harness skill is frontmatter plus a pointer to the canonical body:

```markdown
---
name: my-skill
description: "..."
---

Read the canonical skill at `../../../skills/my-skill/SKILL.md` completely and
follow it. Do not fork the body here.
```

## Notes

- Cursor is a config-dir harness, not a marketplace harness — it has no
  `plugin.json`, no `marketplace.json`, and no install command.
- The "no background subagents" constraint is a **current design stance**, not
  a permanent law: re-verify against `skill-forge`'s harness-capability-matrix
  before asserting it in a new plugin.
