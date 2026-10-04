# Codex Marketplace — reference

Source: OpenAI Codex plugin docs (public plugins publish once to a universal
directory shared by ChatGPT + Codex; local/repo marketplaces are fully
supported for authoring/testing/team distribution).

## Two-layer model

| Layer | File | Role |
|---|---|---|
| Plugin manifest | `.codex-plugin/plugin.json` | one plugin = one folder |
| Marketplace catalog | `marketplace.json` | curated list of plugins |

## Plugin layout

```
my-plugin/
├── .codex-plugin/plugin.json   # REQUIRED — only file in this dir
├── skills/<name>/SKILL.md      # bundled skills
├── hooks/hooks.json            # optional hooks (auto-detected)
├── .mcp.json                   # optional bundled MCP servers
├── .app.json                   # optional registered MCP mappings (compat)
└── assets/                     # icons, logos, screenshots
```

## `plugin.json` — required fields

- `name` — kebab-case, stable ID (component namespace)
- `version` — strict semver
- `description`

## `plugin.json` — optional fields

`author` {name, email, url} · `homepage` · `repository` · `license` ·
`keywords[]` · `skills` · `mcpServers` · `apps` · `hooks` (path | array |
inline) · `interface` (see below).

`interface` (install-surface metadata): `displayName`, `shortDescription`,
`longDescription`, `developerName`, `category`, `capabilities[]`,
`websiteURL`, `privacyPolicyURL`, `termsOfServiceURL`, `defaultPrompt[]`,
`brandColor`, `composerIcon`, `logo`, `screenshots[]`.

## `marketplace.json`

Locations: repo `$REPO_ROOT/.agents/plugins/marketplace.json`, legacy-compat
`$REPO_ROOT/.claude-plugin/marketplace.json`, personal
`~/.agents/plugins/marketplace.json`.

Top-level: `name` (marketplace ID), optional `interface.displayName` (picker
title), `plugins[]` (ordered — order = render order).

Per-plugin entry:

```json
{
  "name": "my-plugin",
  "source": { "source": "local", "path": "./plugins/my-plugin" },
  "policy": { "installation": "AVAILABLE", "authentication": "ON_INSTALL" },
  "category": "Productivity"
}
```

`source.source` types:

| type | fields | notes |
|---|---|---|
| `local` | `path` | also allowed as plain string `"./plugins/x"` |
| `git-subdir` | `url`, `path`, `ref`\|\|`sha` | plugin in a subdir of a git repo |
| `url` | `url` | plugin at repo root |
| `npm` | `package`, `version`?, `registry`? | no lifecycle scripts; npm CLI required |

Policy: `installation` ∈ `AVAILABLE` | `INSTALLED_BY_DEFAULT` |
`NOT_AVAILABLE`; `authentication` ∈ `ON_INSTALL` | first-use. `category`
required. Unresolvable entries are skipped (not fatal).

Path rules: relative to marketplace root, `./`-prefixed, stay inside root.

## CLI surface

```
codex plugin marketplace add owner/repo [--ref main]
codex plugin marketplace add https://github.com/.../plugins.git --sparse .agents/plugins
codex plugin marketplace add ./local-marketplace-root
codex plugin marketplace list | upgrade [name] | remove <name>
codex   # then /plugins → interactive browser (install/uninstall, Space=toggle)
```

## Versioning + validation

- Versioning: strict semver in `plugin.json`. Git marketplace entries pin via
  `ref`/`sha`; npm entries via `version` (semver range/tag, not path/URL).
- Install cache: `~/.codex/plugins/cache/$MARKET/$PLUGIN/$VERSION/`
  (`$VERSION = local` for local plugins). Enable/disable in
  `~/.codex/config.toml`.
- Validation: NO `codex plugin validate`; validation at install (source
  resolution; bad entries skipped). Scaffold via `@plugin-creator`. Hooks are
  non-managed → user must review/trust before they run.

## sc-dolt → Codex export mapping

| sc-dolt package field | Codex artifact |
|---|---|
| package name | `plugin.json.name` (kebab-case) |
| version | `plugin.json.version` (strict semver) |
| description | `plugin.json.description` + `interface.shortDescription` |
| SKILL.md body + frontmatter | `skills/<name>/SKILL.md` (zero transform) |
| references/ · scripts/ · formulas/ | bundled under `skills/<name>/` |
| long description / docs | `interface.longDescription` |
| author / license / homepage | `plugin.json.author` / `license` / `homepage` |
| category | `interface.category` + marketplace entry `category` |

Export = one plugin folder + one marketplace.json entry. SKILL.md is the
common denominator across Claude/Codex/Hermes; only the packaging layer
differs. sc-dolt Dolt branch = marketplace channel.

## Exporter gaps (Codex)

1. No programmatic validate API → replicate install-time checks in exporter.
2. No uninstall/drift tracking → diff `~/.codex/plugins/cache/` + config.toml.
3. No public-directory API → sc-dolt is the registry/metadata layer.
4. No dependency resolution → sc-dolt provides it.
5. No update notification → version compare is client-side; sc-dolt emits delta.
6. No published JSON schema → doc-described only; exporter owns pydantic emitter.
