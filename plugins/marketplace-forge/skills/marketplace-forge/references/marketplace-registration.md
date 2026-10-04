# Marketplace Creation & Registration — per-platform minimums

How to stand up a marketplace and the minimum set of requirements to register
it, per platform. **Create** = the artifacts you ship; **register** = what
makes a consumer able to add/install it.

## Claude Code

Two models coexist. Know which one you are targeting.

### Model A — `.claude-plugin/marketplace.json` (live-repo format)

Used by the live repos (process-porn, atm-bd-orchestration). A marketplace is
a repo whose root `.claude-plugin/marketplace.json` lists plugins.

**Create (minimum):**
- `.claude-plugin/marketplace.json` — `name`, `owner{name}`, `plugins[]`.
- One plugin folder per plugin: `plugins/<name>/.claude-plugin/plugin.json`
  with `name`, `description`, `version`.

**Register (minimum):**
- Publish the repo; consumer runs `/plugin marketplace add owner/repo`.

### Model B — `docs/registries/nuget/registry.json` (infrastructure guide)

The fuller registry model (see `MARKETPLACE-INFRASTRUCTURE.md`). Claude Code
discovers the registry at
`raw.githubusercontent.com/owner/repo/{main,master}/docs/registries/nuget/registry.json`,
then falls back to GitHub Pages.

**Create (minimum required `registry.json` fields):**
- Root: `version`, `generated`, `repo`, `marketplace`, `packages`, `metadata`,
  `versionCompatibility`.
- `marketplace`: `name`, `version`, `status`, `url`.
- Each `packages[name]`: `name`, `version`, `status`, `tier`, `description`,
  `github`, `repo`, `path`, `readme`, `license`, `author`, `tags`, `artifacts`,
  `dependencies`, `changelog`, `lastUpdated`, `dependents`.

**Register (minimum):**
- A GitHub repo with the registry committed (raw URLs serve it — no extra
  infra needed); consumer runs `/plugin marketplace add owner/repo`.

## Codex

**Create (minimum):**
- `marketplace.json` at `$REPO_ROOT/.agents/plugins/marketplace.json` (or
  legacy-compat `.claude-plugin/marketplace.json`): `name` + `plugins[]`.
- Per plugin: `plugins/<name>/.codex-plugin/plugin.json` with `name`,
  `version` (strict semver), `description`.
- Per marketplace entry: `name`, `source` (local path / git-subdir / url /
  npm), `category`.

**Register (minimum):**
- Consumer runs `codex plugin marketplace add owner/repo` (or
  `add <url>` / `add ./local-marketplace-root`).

## Cursor

**No marketplace.** Cursor reads repo-local `.cursor/` config dirs.

**Create (minimum):**
- `.cursor/skills/<name>/SKILL.md` (+ `.cursor/agents/`, `.cursor/commands/`
  as needed).

**Register (minimum):**
- None — add the repo (or `.cursor/` tree) to the project; Cursor loads it
  natively.

## Hermes

**No marketplace manifest.** The registry layer is sc-dolt.

**Create (minimum):**
- `~/.hermes/profiles/<profile>/skills/<category>/<name>/SKILL.md` with
  frontmatter: `name`, `description`, `version`, `author`, `license`,
  `platforms`, `metadata.hermes.tags`.

**Register (minimum):**
- sc-dolt package entry (name, version, content-addressed files) + curator
  review pipeline. Install is deferred until the skill is complete.

## Others

| Platform | Create (minimum) | Register |
|---|---|---|
| Copilot | `.copilot-plugin/plugin.json` | per multi-harness defining doc (optional) |
| OpenCode / OpenClaw | `.agents/skills/<name>/SKILL.md` (vendor-neutral) | repo-local, like Cursor |

## Summary — minimum to register, per platform

| Platform | Marketplace artifact | Register command |
|---|---|---|
| Claude Code (A) | `.claude-plugin/marketplace.json` | `/plugin marketplace add owner/repo` |
| Claude Code (B) | `docs/registries/nuget/registry.json` | `/plugin marketplace add owner/repo` |
| Codex | `marketplace.json` | `codex plugin marketplace add owner/repo` |
| Cursor | `.cursor/` tree | none (repo-local) |
| Hermes | SKILL.md + frontmatter | sc-dolt + curator |
