# sc-meta-skill — skill-forge

The single source of truth for **multi-agent skill creation**: building skills
that orchestrate one or more agents, following the skills/agents architecture
copied from `synaptic-canvas`. Skills built here land in
`synaptic-canvas-dolt` as versioned registry units.

The primary skill is **`skill-forge`**: its `SKILL.md` is a table of contents
over a reference library, anchored by the skills/agents architecture
guidelines, so skills are built from canonical documents rather than re-derived
each time.

## Reference library

| Document | Role | Governs |
|---|---|---|
| `skills/skill-forge/references/claude-code-skills-agents-guidelines.md` | **primary** | two-tier skill/agent architecture (v0.7, copied verbatim) |
| `skills/skill-forge/references/dependency-model.md` | supplement | the five dependency classes (D1–D5) |
| `skills/skill-forge/references/harness-capability-matrix.md` | supplement | dated harness capabilities (drift guardrail) |

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
