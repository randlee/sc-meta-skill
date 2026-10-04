# Dependency Model — D1–D5, worked from the live repos

A plugin dependency is any fact the plugin's parts rely on that lives outside
the part itself. Each class has exactly one correct declaration site; a
dependency written anywhere else is a drift hazard.

## D1 — skill→skill (knowledge layering)

One skill's mechanics live in another skill; the consumer names the owner and
does not re-explain.

Declaration site: `metadata.hermes.related_skills` (Hermes) / a "mechanics
live in X" line near the top of the body (all harnesses).

Worked examples:

- **sc-publish** `.codex/skills/prerelease/SKILL.md` → the canonical
  `.claude/skills/prerelease/SKILL.md` via a relative path. The whole
  cross-harness thin-adapter rule is D1: the adapter's only dependency is the
  canonical body.
- **beads-management** (`~/github/hendrix/skills/`) → "everyday `bd` usage
  lives in `beads-usage`"; "ack/send mechanics live in `atm-messaging`".
- **agent-invocation** → the Pydantic sidecar contract is owned by
  `portability-patterns`; `agent-invocation` names it, doesn't restate it.

Anti-pattern: re-explaining the dependency's mechanics inline. When the owner
changes, the inline copy drifts silently.

## D2 — skill→agent (named teammate / subagent)

The plugin's flow depends on a named agent: an ATM teammate, a channel worker,
or a per-deliverable subagent.

Declaration site: the agent file itself (`agents/<name>.md`) plus the
invocation site's identity contract (`ATM_TEAM` / `ATM_IDENTITY`,
`spawn_policy: named_teammate_required`).

Worked examples:

- **sc-publish** `.claude/agents/publisher.md` declares
  `spawn_policy: named_teammate_required` and a production identity of exactly
  `publisher`; it fans out to role-specific channel workers
  (`crates-io-publisher`, `pypi-publisher`, …). The skill must NOT launch an
  unnamed background agent or a version-specific identity.
- **atm-bd-orchestration** `roles/dev-sanity.md` — "runs one check subagent
  per deliverable… closes them in whatever order their verdicts arrive"; the
  subagent's contract lives in its own `## Inputs` / `## Output Format`.

Key rule: the agent owns its contract (inputs/outputs), and the invocation
site names the agent — it never inlines the agent's whole prompt.

## D3 — skill→artifact (install-time rendered manifest)

The plugin's runtime behavior is driven by a manifest that is rendered at
install time for the specific consumer.

Declaration site: `install.py` + the `.toml.j2` template it renders; the
rendered file is the runtime artifact.

Worked examples:

- **sc-publish** `install.py` renders `release/publish-artifacts.toml`
  (consumer-specific) from `publish-artifacts.toml.j2`, and vendors
  `release/publish-channel-contracts.toml` as the shared channel contract.
  The README states the hard rule: "the installer never infers a publish
  surface or enables channels" — the caller owns an explicit JSON input.

Anti-pattern: a skill that guesses the manifest's contents instead of reading
the rendered file.

## D4 — skill→binding (pinned runtime library)

The plugin executes through a specific library version; an unpinned system
copy is a drift hazard.

Declaration site: the install/bootstrap step that pins the wheel, and the CI
that provisions it.

Worked example:

- **sc-publish** `bootstrap_sc_compose.py` provisions the pinned `sc-compose`
  Python bindings into a venv; the skill assumes the pinned wheel, never a
  system `sc-compose`.

## D5 — skill→harness (capability that drifts)

The plugin's prose asserts what a target harness can or cannot do. This is the
class that produced the stale-info bugs.

Declaration site: the canonical matrix
(`references/harness-capability-matrix.md`), referenced — never inlined — in
the body. Alternatively, harness-agnostic wording that avoids the assertion
entirely.

Worked examples (both verified):

- **process-porn** `skills/reviewing-ceremony/SKILL.md:39` and
  `agents/ceremony-review.md:13` say "invoke via the **Task tool**" — stale
  since Claude Code v2.1.63 renamed it to the `Agent` tool.
- **Codex** — a plugin that assumed "Codex can't launch subagents" was stale;
  Codex CLI added subagents (~2025).
- **atm-bd-orchestration** `skills/atm-bd-orchestration/roles/quality-mgr.md:106`
  is the correct pattern: "a subagent or child agent, **whichever the harness
  provides**" — no assertion, no drift.

## Cross-class rule

A dependency belongs to exactly one class. If a fact looks like it might be
D1 and D5 (e.g. "the subagent tool is the Task tool"), it is D5 — split the
harness assertion out of the knowledge pointer, so the pointer survives a
rename and the assertion is refreshed in one place.
