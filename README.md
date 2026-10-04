# sc-meta-skill — the forge marketplace

A marketplace containing two forge plugins for the Synaptic Canvas skill
pipeline:

| Plugin | For | Reference library |
|---|---|---|
| **skill-forge** | multi-agent skill *creation* (authoring) | skills/agents architecture guidelines (v0.7), dependency model, harness-capability matrix |
| **marketplace-forge** | plugin *packaging* (distribution) | claude-marketplace, codex-marketplace, cursor-marketplace, hermes-marketplace |

Together they break the loop: **skill-forge** builds skills to a canonical
architecture, **marketplace-forge** ships them as installable plugins across
Claude Code / Codex / Cursor / Hermes, and both land in
`synaptic-canvas-dolt` as versioned registry units.

## Structure

```
sc-meta-skill/
  .claude-plugin/marketplace.json        # catalog: [skill-forge, marketplace-forge]
  plugins/skill-forge/                   # plugin 1 — authoring
    .claude-plugin/plugin.json
    .codex-plugin/plugin.json
    skills/skill-forge/
      SKILL.md
      references/  (guidelines, dependency-model, harness-capability-matrix)
    .cursor/skills/skill-forge/SKILL.md
  plugins/marketplace-forge/             # plugin 2 — packaging
    .claude-plugin/plugin.json
    .codex-plugin/plugin.json
    skills/marketplace-forge/
      SKILL.md
      references/  (claude/codex/cursor/hermes marketplace)
    .cursor/skills/marketplace-forge/SKILL.md
```

One body, N thin adapters: the knowledge lives in each skill's `SKILL.md`
exactly once; every harness adapter is frontmatter plus a pointer — never a
forked copy.

## Install

- **Claude Code:** `/plugin marketplace add randlee/sc-meta-skill`
- **Codex:** `codex plugin marketplace add randlee/sc-meta-skill`
- **Cursor:** add the repo (or `.cursor/` tree) to the project; Cursor loads
  `.cursor/skills/` natively.
- **Hermes:** install when complete.
