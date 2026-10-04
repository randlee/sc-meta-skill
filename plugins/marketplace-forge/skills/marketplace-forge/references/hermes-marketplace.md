# Hermes — reference

Hermes has no plugin marketplace manifest. Skills load from
`~/.hermes/profiles/<profile>/skills/<category>/<name>/SKILL.md`. The adapter
is the **frontmatter**, not a manifest file.

## Layout

```
~/.hermes/profiles/<profile>/skills/<category>/<name>/
  SKILL.md              # frontmatter is the adapter
  references/           # lazy-loaded docs
  scripts/              # helpers
  templates/            # templates
```

## SKILL.md frontmatter (Hermes)

```yaml
---
name: my-skill
description: "..."          # ≤60 chars, capability statement
version: 1.0.0
author: skillrx
license: MIT
platforms: [macos]          # OS gate
metadata:
  hermes:
    tags: [category, topic]           # curator indexing
    related_skills: [other-skill]     # must resolve to an installed skill
---
```

Hermes-specific fields: `platforms` (OS gate), `metadata.hermes.tags` (curator
indexing), `metadata.hermes.related_skills` (cross-references). Lifecycle via
the curator review pipeline + `skill_manage` (create/patch/edit), not a
manifest file.

## sc-dolt landing

sc-dolt (`synaptic-canvas-dolt`) is the registry layer: packages, versions,
and content-addressed files. A multi-harness package is a package whose
manifest set spans harnesses (claude + codex + cursor + hermes). Discovery
(`sc search`), install (`sc install`), and channels (branches) apply unchanged
— the harness adapter is just more package content.

## Notes

- Hermes is frontmatter-only: no `plugin.json`, no `marketplace.json`.
- `platforms:` gates loading by host OS — audit against what the skill's prose
  and scripts actually invoke, never copy from a sibling.
- Install is deferred until a skill is complete (see `skill-forge`).
