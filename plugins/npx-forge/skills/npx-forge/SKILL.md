---
name: npx-forge
description: "Package skills as npx-installable packages."
version: 0.1.0
author: skillrx
license: MIT
platforms: [macos]
metadata:
  hermes:
    tags: [npx-forge, npx, install, packaging, loader, claude, codex, cursor, hermes]
    related_skills: [skill-forge, marketplace-forge]
---

# npx Forge

npx-forge is the single source of truth for **distributing skills via npx** —
the loader model where `npx skills add <url>` (or `npx agent-skills-cli add`)
downloads a repository/URL containing a `SKILL.md` (or a directory of skills)
and installs it into the target agent's config path.

Where `marketplace-forge` packages a plugin with per-harness manifests
(`.claude-plugin/`, `.codex-plugin/`), `npx-forge` packages a skill for the
npx CLI loader, which resolves a URL/repo and writes the skill to the agent's
on-disk skills dir. It is the third distribution channel, alongside the
marketplace.

## When to Use

- Making a skill installable via `npx skills add <repo-url>`.
- Choosing local vs global install scope for a skill.
- Answering "where does skill X land for agent Y?"
- Deciding between marketplace packaging (`marketplace-forge`) and npx
  packaging (`npx-forge`).

Don't use for: authoring skill content (`skill-forge`); manifest-based
marketplace packaging (`marketplace-forge`); or a Hermes-only frontmatter skill.

## Reference library

| Document | Governs |
|---|---|
| `references/npx-install.md` | the npx install model: URL refs, scope flags, agent target paths, examples |

## The npx install model (summary)

The CLI is a loader: it downloads a repo/URL and writes the `SKILL.md` into
the target agent's designated path. Three decisions, in order:

1. **Source** — a GitHub URL, a specific `--skill` inside a multi-skill repo,
   or a direct archive URL.
2. **Scope** — local (default, `./.claude/skills/`, `./.cursor/skills/`) vs
   global (`-g`, `~/.claude/skills/`, `~/.agents/skills/`).
3. **Agent target** — auto-detected, or explicit via `--agent <name>` /
   `--tools <name>`.

## Per-agent target paths

| Agent | Global | Local |
|---|---|---|
| Claude Code | `~/.claude/skills/<skill>/SKILL.md` | `./.claude/skills/<skill>/SKILL.md` |
| Cursor | `~/.cursor/skills/<skill>/SKILL.md` | `./.cursor/skills/<skill>/SKILL.md` |
| Codex / OpenCode | `~/.agents/skills/<skill>/` (or `~/.codex/skills/`) | `./.agents/skills/<skill>/` |
| Hermes | `~/.hermes/skills/<skill>/` | `./.hermes/skills/` |

Note the vendor-neutral standard: Codex/OpenCode use `.agents/skills/`; Hermes
user-installed skills under `~/.hermes/skills/` or `.agents/skills/` override
bundled defaults.

## Procedure — make a skill npx-installable

1. Author the canonical skill first (`skill-forge`). *Done when the body exists.*
2. Structure the repo so the loader finds it: a single `SKILL.md` at the repo
   root, or a `skills/` directory the `--skill <name>` flag can select.
   *Done when `npx skills add <url>` resolves a `SKILL.md`.*
3. Decide the scope (local vs global) and document it; the CLI's default is
   local. *Done when the intended scope is stated.*
4. Exercise the install across each target agent path. *Done when each target
   agent loads the installed skill.*

## Pitfalls

1. **Scope surprise** — local is the default; a "global" workflow installed
   locally commits `./.claude/skills/` into the repo unexpectedly.
2. **Agent misdetection** — auto-detect may skip an agent; pass `--agent` /
   `--tools` explicitly when the target matters.
3. **Multi-skill repos** — `--skill <name>` selects one; without it the loader
   may take the whole directory.
4. **Archive URLs** — a direct `.zip`/`.tar` URL must contain the `SKILL.md`
   at the expected relative path.
5. **Untrusted lifecycle scripts** — npx runs a package's `preinstall` /
   `install` / `postinstall` on fetch; for untrusted packages pass
   `--ignore-scripts`. The package's own dependency tree resolves into a
   transient cache (`~/.npm/_npx/`), not the project folder.

## Verification

- [ ] Canonical body exists (skill-forge output)
- [ ] `npx skills add <url>` resolves a `SKILL.md`
- [ ] Intended scope (local vs global) matches the actual install path
- [ ] Each target agent loads the installed skill once
