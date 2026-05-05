# Gemini CLI Port — Research Notes (Phase 0)

> **Purpose.** Establish the facts needed to install this plugin on Gemini CLI without guessing. Inputs to Phases 1–5. Captured 2026-05-05.

## Codex baseline (pre-research snapshot)

What Codex actually delivers in this repo today, frozen here so later phases can prove no regression:

- `.codex/INSTALL.md`: clones the repo, then **only** symlinks `skills/` into `~/.agents/skills/devtools`. Commands, agents, hooks, memories are NOT installed. Includes a hedge: *"Verify symlink path against current Codex docs — path may vary by version."* No verified-on-version statement.
- `skills/using-devtools/references/codex-tools.md`: 6-row mapping (`Read → ReadFile`, `Write → WriteFile`, `Edit → PatchFile`, `Glob → Glob`, `Grep → Grep`, `Bash → Shell`). Missing: `Agent`, `Skill`, `Task`, `TodoWrite`, `WebFetch`, `WebSearch`, `NotebookEdit`, and any others the audit surfaces in Phase 1.
- Updates mechanism: `cd ~/.codex/devtools && git pull`. No `/plugin update` equivalent.
- `skills/using-devtools/SKILL.md` line 8 says *"installed as a single Claude Code plugin"* — incorrect for Codex users.
- `skills/using-devtools/SKILL.md` line 22 only references `codex-tools.md`, not Gemini.

## Q1 — `gemini-extension.json` manifest schema

Authoritative reference: <https://geminicli.com/docs/extensions/reference/>.

| Field | Type | Required | Notes |
|---|---|---|---|
| `name` | string | **yes** | Lowercase + dashes; no underscores or spaces; used for conflict resolution |
| `version` | string | **yes** | Semver |
| `description` | string | **yes** | Shown on geminicli.com/extensions |
| `mcpServers` | object | no | Map of MCP server configs; loaded at startup, just like `settings.json`-configured servers |
| `contextFileName` | string | no | Defaults to `GEMINI.md` if present in the extension dir |
| `excludeTools` | array | no | Block tools; supports per-command shape like `run_shell_command(rm -rf)` |
| `migratedTo` | string | no | URL of new repo for auto-migration |
| `plan` | object | no | Planning-feature config; optional `directory` property |
| `settings` | array | no | User-configurable env-var-mapped settings |
| `themes` | array | no | Custom UI theme definitions |

**Variable substitution** in manifest + hook files: `${extensionPath}`, `${workspacePath}`, `${/}`. Mirrors Claude Code's `${CLAUDE_PLUGIN_ROOT}` pattern.

**Current repo's `gemini-extension.json` has only**: `name`, `description`, `version`, `contextFileName`. That is **valid** (all required fields present) but minimal. Adding `mcpServers` is unnecessary for v1 since the plugin defines no MCP servers.

## Q2 — Skill loader paths

Authoritative reference: <https://geminicli.com/docs/cli/skills/>.

Skills are discovered from these tiers (precedence: extension < user < project):

| Tier | Primary path | Alias |
|---|---|---|
| **User** (global) | `~/.gemini/skills/` | `~/.agents/skills/` |
| **Project** (per-repo) | `<repo>/.gemini/skills/` | `<repo>/.agents/skills/` |
| **Extension** (bundled) | `<extension-dir>/skills/` | — |

Per the docs, *"if multiple skills share the same name, the version from the higher-precedence location is used."*

**The Codex alias `~/.agents/skills/` is also a valid Gemini location.** This means a user who has already symlinked `skills/` for Codex via the existing INSTALL flow gets Gemini support **for free** at the user-tier — no additional symlink needed. Worth surfacing in `GEMINI-INSTALL.md`.

For an extension-distributed install, skills go in `<extension-dir>/skills/` and are discovered automatically when the extension is active. No symlink required.

## Q3 — `SKILL.md` frontmatter format

Authoritative reference: <https://geminicli.com/docs/cli/creating-skills/>.

Required frontmatter:

| Field | Type | Constraint |
|---|---|---|
| `name` | string | Should match the directory name |
| `description` | string | **Critical** — drives auto-activation |

**No other frontmatter fields are documented as required or optional.** Our existing skills already use exactly these two fields → **format-compatible without changes.**

Required directory shape:

```
my-skill/
├── SKILL.md       (required, at root)
├── scripts/       (optional)
├── references/    (optional)
└── assets/        (optional)
```

Our skills follow this exact shape (e.g., `skills/using-devtools/SKILL.md` + `skills/using-devtools/references/`). **Compatible without restructuring.**

## Q4 — `contextFileName` scope

Authoritative reference: <https://geminicli.com/docs/extensions/reference/> + <https://google-gemini.github.io/gemini-cli/docs/extensions/getting-started-extensions.html>.

> *"Loaded in every session where the extension is active."*

So `contextFileName` is **per-extension, not global**. `GEMINI.md` will be loaded only when the devtools extension is active in a session. This is the correct behavior — matches Claude Code's plugin-loaded `CLAUDE.md`.

If `contextFileName` is omitted but a `GEMINI.md` is present in the extension directory, Gemini loads it automatically. Our manifest currently sets `"contextFileName": "GEMINI.md"` explicitly, which is fine.

## Q5 — `activate_skill` auto-trigger semantics

Authoritative references: <https://geminicli.com/docs/cli/skills/>, <https://geminicli.com/docs/tools/activate-skill/>.

Behavior:

1. At session start, Gemini CLI scans all discovery tiers and **injects each skill's `name` and `description` into the system prompt**.
2. The model auto-recognizes when a skill description matches the user's request.
3. The model calls the `activate_skill` tool, which loads the full `SKILL.md` body + grants access to the skill's directory.

**Note from the official docs:** *"Skills don't strictly auto-trigger — they require model recognition and user consent."* This is functionally identical to Claude Code's `Skill` tool flow. Same behavior, different tool name.

**Implication:** description quality is the single biggest factor for usefulness on both platforms — same constraint as Claude Code.

## Bonus finding — extension layout convention matches this repo

The reference page also documents these conventions for extension subdirectories:

| Subdir | Purpose | Format |
|---|---|---|
| `commands/` | Custom slash commands | TOML (`.toml`) |
| `hooks/hooks.json` | Lifecycle hooks | JSON |
| `skills/` | Agent skills | SKILL.md per folder |
| `agents/` | Subagents | `.md` files |
| `policies/` | Policy rules | TOML |

**This repo already uses `skills/`, `agents/`, `commands/`, `hooks/hooks.json` — directory layout matches Gemini's convention.** The format gaps that remain:

- `commands/*.md` (Claude Code) → must become `commands/*.toml` (Gemini). Real schema gap → v2a.
- `agents/*.md` (Claude Code: `name`, `description`, `model`, `allowed-tools`) vs Gemini agents `.md`. Frontmatter compatibility unknown — needs Phase 0 follow-up before v2b. **Open question, deferred.**
- `hooks/hooks.json` uses `${CLAUDE_PLUGIN_ROOT}` and Claude Code's `SessionStart` matcher — Gemini uses `${extensionPath}` and may have a different lifecycle event name. **Open question for any hook port, but v1 substitutes via `GEMINI.md` so this does not block v1.**

## Implications for the plan

### v1 simplifies — better than expected

1. **Skills work out of the box.** Format is already compatible. No frontmatter rewrite needed. Symlinking `skills/` into `~/.gemini/skills/` (or `~/.agents/skills/`, which Codex already uses) is sufficient.
2. **Codex users with existing symlink get Gemini support free.** Update `GEMINI-INSTALL.md` to mention this.
3. **`gemini-extension.json` already valid.** No schema additions needed for v1. Optional: add `excludeTools` if any tool needs blocking, but no current need.
4. **`contextFileName: GEMINI.md` is correctly scoped** to extension activation — matches plugin-load semantics.
5. **Auto-triggering parity.** Skill descriptions drive activation on both platforms identically. No copy changes required for skills to trigger correctly.

### v1 risks (still real)

1. **Skill body uses Claude-Code-only tool references** (`Skill`, `Agent`, `Task`, etc.). When a skill auto-triggers on Gemini, instructions like *"use the Skill tool to invoke X"* will confuse Gemini. **Phase 1 tool mapping must be exhaustive** — same finding as before, validated harder now.
2. **`using-devtools/SKILL.md` references commands** (`/remember`, `/memory-export`) that are not installed on Gemini in v1. Phase 2b's edits must qualify these "(Claude Code only)".

### v2 cheaper than originally estimated

1. **Commands → TOML:** the Gemini TOML format only requires `prompt` and `description` per the docs. A simple build script that reads our MD frontmatter (`description`) + body (`$ARGUMENTS`) and emits TOML with `description = "..."` and `prompt = "..."` covers it. Effort: ~1 hour, not 2.
2. **Agents:** if Gemini's agent `.md` accepts `name` + `description` frontmatter (likely, given the symmetry with skills), our base templates may port without rewrites — only `allowed-tools` and `model: inherit` need investigation. Effort estimate hinges on this open question.
3. **Hooks:** a single SessionStart equivalent (if Gemini has one, even with a different name) would let us remove the `GEMINI.md` substitute from v1. Unknown today, low priority.

## Summary table — Phase 0 acceptance criteria

| Criterion | Status | Source |
|---|---|---|
| Manifest schema documented | ✅ | <https://geminicli.com/docs/extensions/reference/> |
| Skill loader path documented | ✅ | <https://geminicli.com/docs/cli/skills/> |
| Auto-trigger from description confirmed | ✅ | <https://geminicli.com/docs/cli/skills/>, <https://geminicli.com/docs/tools/activate-skill/> |
| `contextFileName` scope documented | ✅ | <https://geminicli.com/docs/extensions/reference/>, <https://google-gemini.github.io/gemini-cli/docs/extensions/getting-started-extensions.html> |
| Codex baseline captured | ✅ | This document |
| Each finding cited | ✅ | URLs above |

## Sources

- [Gemini CLI — Extension reference](https://geminicli.com/docs/extensions/reference/)
- [Gemini CLI — Agent Skills](https://geminicli.com/docs/cli/skills/)
- [Gemini CLI — Creating Agent Skills](https://geminicli.com/docs/cli/creating-skills/)
- [Gemini CLI — `activate_skill` tool](https://geminicli.com/docs/tools/activate-skill/)
- [Gemini CLI — Custom commands](https://geminicli.com/docs/cli/custom-commands/)
- [Gemini CLI — Getting started with extensions](https://google-gemini.github.io/gemini-cli/docs/extensions/getting-started-extensions.html)
- [Gemini CLI — Get started with Agent Skills tutorial](https://geminicli.com/docs/cli/tutorials/skills-getting-started/)
