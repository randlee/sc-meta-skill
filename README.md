# sc-meta-skill — skill-forge

The single source of truth for **forging cross-agent skills and agents** that
load across Claude Code, Codex, Cursor, and Hermes — and that land in
`synaptic-canvas-dolt` as versioned registry units.

The primary skill is **`skill-forge`**: a curated forge whose `SKILL.md` is a
table of contents over a reference library, so skills are built from canonical
documents rather than re-derived each time.

## Reference library

| Document | Governs |
|---|---|
| `skills/skill-forge/references/claude-code-skills-agents-guidelines.md` | two-tier skill/agent architecture (v0.7, copied verbatim) |
| `skills/skill-forge/references/dependency-model.md` | the five dependency classes (D1–D5) |
| `skills/skill-forge/references/harness-capability-matrix.md` | dated harness capabilities (drift guardrail) |

## Structure

```
sc-meta-skill/
  .claude-plugin/marketplace.json        # marketplace catalog (Claude Code + Codex)
  plugins/sc-meta-skill/
    .claude-plugin/plugin.json           # Claude Code manifest
    .codex-plugin/plugin.json            # Codex manifest
    skills/skill-forge/                  # CANONICAL body + reference library
      SKILL.md
      references/…/
    .cursor/skills/skill-forge/SKILL.md  # Cursor thin adapter → canonical body
```

One body, N thin adapters. The knowledge lives in `skills/skill-forge/`
exactly once; every harness adapter is frontmatter plus a pointer — never a
forked copy.

## Install

- **Claude Code:** `/plugin marketplace add randlee/sc-meta-skill`
- **Codex:** `codex plugin marketplace add randlee/sc-meta-skill`
- **Cursor:** add the repo (or `.cursor/` tree) to the project; Cursor loads
  `.cursor/skills/` natively.
- **Hermes:** install when complete (see `skills/skill-forge/SKILL.md`).
