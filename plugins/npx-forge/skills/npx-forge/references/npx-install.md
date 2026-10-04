# npx Skill Installation — reference

When installing agent skills via npx (most commonly `npx skills` or
`npx agent-skills-cli`), the CLI acts as a **loader**: it downloads a
repository/URL containing a `SKILL.md` (or a directory of skills) and installs
it into the target agent's designated configuration path.

## 1. URL references

```bash
# Full GitHub repository URL
npx skills add https://github.com/owner/repo-name

# Specific skill inside a multi-skill repository
npx skills add https://github.com/owner/repo-name --skill my-specific-skill

# Direct archive URL download
npx skills add https://example.com/downloads/my-skill-pack.zip
```

## 2. Global vs. local installation

The CLI decides where to save skill folders based on scope flags and vendor
defaults.

| Scope | Command flag | Location | Best used for |
|---|---|---|---|
| Local (project) | default (no flag) | inside the active project dir, e.g. `./.claude/skills/` or `./.cursor/skills/` | project-specific context, DB schemas, repo rules, team-shared tools committed to version control |
| Global (user) | `-g` / `--global` | the user's home folder, e.g. `~/.claude/skills/` or `~/.agents/skills/` | personal workflows, global CLI triggers, reusable utility scripts, system-wide assistants |

## 3. Agent-specific target paths

When you run `npx skills add <url>`, the CLI auto-detects installed agents, or
accepts an explicit target via `--agent <name>` (or `--tools <name>`).

### Claude (Claude Code)

- Global: `~/.claude/skills/<skill-name>/SKILL.md`
- Local: `./.claude/skills/<skill-name>/SKILL.md`
- Behavior: auto-exposes installed skills as `/slash-commands`, or loads them
  dynamically during natural-language prompts when relevance matches.

### Cursor

- Global: `~/.cursor/skills/<skill-name>/SKILL.md`
- Local: `./.cursor/skills/<skill-name>/SKILL.md`
- Behavior: integrates into Cursor's plugin system; skills can be tagged in
  Chat or Composer via the `@` menu.

### Codex (OpenAI Codex / OpenCode)

- Global: `~/.agents/skills/<skill-name>/` or `~/.codex/skills/`
- Local: `./.agents/skills/<skill-name>/`
- Behavior: uses the vendor-neutral `.agents/skills/` standard.

### Hermes (Nous Research)

- Global: `~/.hermes/skills/<skill-name>/`
- Local: `./.hermes/skills/`
- Behavior: bundled skills sync via its own CLI; user-installed skills from
  npx placed in `~/.hermes/skills/` or `.agents/skills/` override defaults.

## 4. Direct installation examples

```bash
# Target a specific agent explicitly (local)
npx agent-skills-cli add https://github.com/user/repo --agent cursor

# Target a specific agent explicitly (global)
npx agent-skills-cli add https://github.com/user/repo --agent claude --global

# Install across all detected agents globally
npx skills add https://github.com/user/repo -g -y
```

## 5. How npx resolves dependencies

When you execute a package via npx, it automatically resolves and installs all
required dependencies defined in that package's `package.json` into a
temporary cache before executing the package.

- **Automatic resolution** — if a package isn't installed locally in your
  project's `node_modules` or globally, npx downloads the package along with
  its entire dependency tree into a temporary directory (e.g. `~/.npm/_npx/`).
- **Transient storage** — after execution finishes, the downloaded
  dependencies remain cached temporarily for speed on subsequent runs, but do
  not pollute your system or local project folder.

## 6. Executing package install scripts

- **Lifecycle scripts** — if the fetched package contains standard npm
  lifecycle scripts (`preinstall`, `install`, `postinstall`), npm (which
  powers npx) executes them as part of the installation step before running
  the binary or command.
- **Security & `--ignore-scripts`** — because lifecycle scripts run arbitrary
  shell commands on your machine during fetch, executing untrusted packages
  with npx carries security risks. Prevent dependencies/packages from running
  install scripts by passing the flag through npm:

```bash
# Prevents npm from running postinstall/build scripts during package retrieval
npx --ignore-scripts <package-name>
```

## 7. Running custom installation scripts

To have npx execute a script that installs dependencies into an existing
target directory (e.g. a cloned project or CLI tool), invoke npm commands
directly through npx:

```bash
# Run npm install via npx in the current folder
npx -c "npm install"

# Run a specific post-install / setup script defined in a remote repo package
npx <package-name> install
```

## Key facts

- `-y` / `--yes` — skip interactive confirmation (non-interactive install).
- `--skill <name>` — select one skill from a multi-skill repo.
- `--agent <name>` / `--tools <name>` — pin the target when auto-detect is
  wrong or the target matters.
- Local is the **default** scope; `-g` opts into global.
