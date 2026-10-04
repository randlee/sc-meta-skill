# sc-meta-skill

The single source of truth for **authoring cross-harness marketplace plugins
with dependency contracts**. This is the meta-skill that governs how the
Synaptic Canvas ecosystem builds skills that load across Claude Code, Codex,
Cursor, and Hermes — and that land in `synaptic-canvas-dolt` as versioned
registry units.

The skill itself is `marketplace-plugin-authoring`. It declares the five
dependency classes (D1 skill→skill, D2 skill→agent, D3 skill→artifact,
D4 skill→binding, D5 skill→harness) and ships a dated harness-capability
matrix so no plugin ever hardcodes a stale assumption ("Task tool", "Codex
can't spawn subagents", …).

## Structure

```
sc-meta-skill/
  .claude-plugin/marketplace.json        # marketplace catalog (Claude Code + Codex)
  plugins/sc-meta-skill/
    .claude-plugin/plugin.json           # Claude Code manifest
    .codex-plugin/plugin.json            # Codex manifest
    skills/marketplace-plugin-authoring/ # CANONICAL body (one source of truth)
      SKILL.md
      references/dependency-model.md
      references/harness-capability-matrix.md
    .cursor/skills/…/SKILL.md            # Cursor thin adapter → canonical body
```

One body, N thin adapters. The knowledge lives in
`skills/marketplace-plugin-authoring/` exactly once; every harness adapter is
frontmatter plus a pointer — never a forked copy.

## Install

- **Claude Code:** `/plugin marketplace add randlee/sc-meta-skill`
- **Codex:** `codex plugin marketplace add randlee/sc-meta-skill`
- **Cursor:** add the repo (or `.cursor/` tree) to the project; Cursor loads
  `.cursor/skills/` natively.
- **Hermes:** load `skills/marketplace-plugin-authoring/SKILL.md` via the
  curator / `skill_view`.
