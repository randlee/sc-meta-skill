# Claude Code Marketplace — reference

Verified via live repos (`randlee/process-porn`, `randlee/atm-bd-orchestration`)
and the `multi-harness-skill-package` defining doc. A **marketplace** is a repo;
a **plugin** is a folder carrying a `.claude-plugin/plugin.json` manifest.

## Two-layer model

| Layer | File | Role |
|---|---|---|
| Marketplace catalog | `<repo>/.claude-plugin/marketplace.json` | lists installable plugins |
| Plugin manifest | `plugins/<name>/.claude-plugin/plugin.json` | one plugin = one folder |

## Plugin manifest — `plugin.json`

```json
{
  "name": "my-plugin",
  "description": "One-line capability statement.",
  "version": "1.0.0",
  "author": { "name": "randlee", "url": "https://github.com/randlee" },
  "repository": "https://github.com/org/repo",
  "license": "MIT",
  "homepage": "https://github.com/org/repo",
  "keywords": ["skill", "category"],
  "skills": "./skills/",
  "commands": "./skills/my-skill/commands/",
  "hooks": {
    "SessionStart": [
      { "matcher": "", "hooks": [ { "type": "command", "command": "..." } ] }
    ]
  }
}
```

Key fields:
- `skills` — dir pointer to bundled skills (`./skills/`).
- `commands` — dir of slash-command files.
- `hooks` — session lifecycle injection (SessionStart, etc.).
- SKILL.md frontmatter carries `allowed-tools`, `compatible-with`, `tags`.

## Marketplace catalog — `marketplace.json`

```json
{
  "$schema": "https://anthropic.com/claude-code/marketplace.schema.json",
  "name": "my-marketplace",
  "description": "...",
  "owner": { "name": "randlee" },
  "plugins": [
    {
      "name": "my-plugin",
      "description": "...",
      "author": { "name": "randlee" },
      "category": "workflow",
      "source": "./plugins/my-plugin",
      "homepage": "https://github.com/org/repo"
    }
  ]
}
```

`source` is a `./`-prefixed path relative to the marketplace root, pointing at
the plugin folder. A single repo can list multiple plugins.

## Install

```
/plugin marketplace add <owner>/<repo>
```

## SKILL.md frontmatter (Claude Code)

```yaml
---
name: my-skill
description: "..."          # critical: Claude selects among 100+ skills by this
allowed-tools: [Bash, Read, Grep]   # optional tool gating
tags: [category, topic]
---
```

## Notes

- `$schema` pins the marketplace JSON schema for editor validation.
- The marketplace name and a plugin name need not match; a marketplace can
  host several plugins.
