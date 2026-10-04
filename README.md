# sc-meta-skill — the forge marketplace

A marketplace containing three forge plugins for the Synaptic Canvas skill
pipeline:

| Plugin | For | Reference library |
|---|---|---|
| **skill-forge** | multi-agent skill *creation* (authoring) | skills/agents architecture guidelines (v0.7), dependency model, harness-capability matrix |
| **marketplace-forge** | plugin *packaging* (distribution) | claude-marketplace, codex-marketplace, cursor-marketplace, hermes-marketplace |
| **npx-forge** | *npx* distribution (loader model) | npx-install (URL refs, scope flags, per-agent target paths) |

Together they break the loop: **skill-forge** builds skills to a canonical
architecture, **marketplace-forge** ships them as installable marketplace
plugins, **npx-forge** distributes them via the npx loader — all landing in
`synaptic-canvas-dolt` as versioned registry units.

## Structure

```
sc-meta-skill/
  .claude-plugin/marketplace.json        # catalog: [skill-forge, marketplace-forge, npx-forge]
  plugins/skill-forge/                   # authoring
    .claude-plugin/plugin.json  .codex-plugin/plugin.json
    skills/skill-forge/SKILL.md + references/
    .cursor/skills/skill-forge/SKILL.md
  plugins/marketplace-forge/             # plugin packaging
    .claude-plugin/plugin.json  .codex-plugin/plugin.json
    skills/marketplace-forge/SKILL.md + references/
    .cursor/skills/marketplace-forge/SKILL.md
  plugins/npx-forge/                     # npx distribution
    .claude-plugin/plugin.json  .codex-plugin/plugin.json
    skills/npx-forge/SKILL.md + references/
    .cursor/skills/npx-forge/SKILL.md
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
- **npx:** `npx skills add https://github.com/randlee/sc-meta-skill` (see
  `npx-forge`).
