---
name: marketplace-forge
description: "Package skills and agents as marketplace plugins."
version: 0.1.0
author: skillrx
license: MIT
platforms: [macos]
metadata:
  hermes:
    tags: [marketplace-forge, marketplace, plugins, packaging, claude, codex, cursor, hermes, sc-dolt]
    related_skills: [skill-forge]
---

# Marketplace Forge

marketplace-forge is the single source of truth for **packaging skills and
agents as marketplace plugins** — the distribution layer on top of
skill-forge's authoring layer. Where `skill-forge` governs how to *create* a
multi-agent skill, `marketplace-forge` governs how to *ship* it across Claude
Code, Codex, Cursor, and Hermes.

It does not re-teach the architecture or the dependency model — those are
`skill-forge`'s job. It provides the per-harness packaging references and the
publish workflow.

## When to Use

- Packaging an existing skill + agent set as an installable plugin.
- Adding a per-harness manifest (`.claude-plugin/`, `.codex-plugin/`) to a plugin.
- Publishing a marketplace catalog (`marketplace.json`) that lists plugins.
- Answering "what does plugin X's manifest need for harness Y?"

Don't use for: authoring the skill/agent content itself (use `skill-forge`);
or a single Hermes-only skill with no plugin packaging.

## Reference library

| Document | Governs |
|---|---|
| `references/claude-marketplace.md` | Claude Code `.claude-plugin/` manifest + `marketplace.json` |
| `references/codex-marketplace.md` | Codex `.codex-plugin/` manifest + `marketplace.json` |
| `references/cursor-marketplace.md` | Cursor `.cursor/` config dirs |
| `references/hermes-marketplace.md` | Hermes skills dir + sc-dolt registry landing |

Read the reference for the harness you are targeting; do not inline its fields
into a body.

## Packaging model (consumed from skill-forge)

The D1–D5 dependency model and the thin-adapter rule are defined in
`skill-forge`; marketplace-forge consumes them. In brief:

- **One body, N thin adapters.** The skill body lives once; each harness gets
  a manifest (metadata) or a pointer adapter — never a forked copy.
- **Per-harness manifest is the divergence point.** Claude/Codex use a
  `plugin.json`; Cursor uses `.cursor/` dirs; Hermes uses frontmatter only.
- **One marketplace catalog per repo** lists every installable plugin.

## Procedure — package a plugin

1. Author the canonical skill first (`skill-forge`); packaging never precedes
   content. *Done when the canonical body exists.*
2. Choose target harnesses and add a manifest per harness (`.claude-plugin/`,
   `.codex-plugin/`) or a config tree (`.cursor/`) or frontmatter (Hermes).
   *Done when every target harness has its adapter.*
3. Add the plugin to the repo's `marketplace.json` with name, source, category.
   *Done when the catalog lists the plugin and `source` resolves.*
4. Bump `version` in lockstep across every manifest + frontmatter.
   *Done when no manifest disagrees on version.*
5. Exercise install in each target harness. *Done when each harness installs
   and loads the plugin — not merely "files exist".*

## Pitfalls

1. **Forking the body per harness** — adapters are metadata, not content.
2. **Omitting `source`** in the catalog — the plugin won't resolve.
3. **Version drift** between `plugin.json` and SKILL.md frontmatter.
4. **Assuming Cursor has a `plugin.json`** — it reads `.cursor/` dirs, no manifest.

## Verification

- [ ] Canonical body exists (skill-forge output)
- [ ] One manifest per target harness; no forked bodies
- [ ] `marketplace.json` lists the plugin; `source` resolves to the plugin dir
- [ ] `version` in lockstep across all manifests + frontmatter
- [ ] Each target harness installs AND loads the plugin once
