# Phase 4 — Verification Log

> **Status (2026-05-05):** Static validation complete. Real-machine verification on Gemini CLI and Codex CLI is **deferred** — requires user action on machines with those CLIs installed.

## What was verified (static, in this session)

| Check | Result |
|---|---|
| `gemini-extension.json` is valid JSON | ✅ PASS |
| Required manifest fields per Phase 0 (`name`, `version`, `description`) all present | ✅ PASS |
| `contextFileName: GEMINI.md` target file exists | ✅ PASS |
| `GEMINI.md` cross-references (`skills/using-devtools/references/gemini-tools.md`, `GEMINI-INSTALL.md`) resolve | ✅ PASS |
| `skills/using-devtools/SKILL.md` exists at canonical path | ✅ PASS |
| `skills/using-devtools/references/codex-tools.md` and `gemini-tools.md` exist | ✅ PASS |
| `hooks/hooks.json` is valid JSON | ✅ PASS |
| Codex no-regression: original 6 tool-mapping rows preserved verbatim in `codex-tools.md` | ✅ PASS |
| Codex no-regression: original install commands (clone, `ln -s`, `mklink /J`, `New-Item Junction`, Git Bash) all preserved in `.codex/INSTALL.md` | ✅ PASS |

## What is deferred (real-machine verification)

These steps cannot be performed from a Claude Code session. They require user action on a machine with the relevant CLI installed.

### Gemini CLI verification (deferred)

User must:

1. Follow `GEMINI-INSTALL.md` Option A on a real machine.
2. Run `gemini` in a project directory. Confirm `GEMINI.md` is loaded as context (e.g. ask Gemini to summarize what's installed; the response should reference the components table).
3. Confirm at least 2 skills auto-trigger from descriptions OR are manually invokable via `activate_skill` (the canonical test: ask a question whose phrasing matches a skill's description).
4. Confirm `using-devtools` meta-skill loads and that `skills/using-devtools/references/gemini-tools.md` is reachable.
5. Capture Gemini CLI version (`gemini --version`).
6. Append a line to this file: `Verified on Gemini CLI v<X.Y> on <date> — <name>.`

### Codex CLI verification (deferred)

User must:

1. Re-run `.codex/INSTALL.md` on a clean install on a real Codex machine.
2. Confirm `using-devtools` skill loads in Codex; confirm `codex-tools.md` is referenced; confirm the expanded mapping doesn't break anything that worked under the original 6-row table.
3. Capture Codex CLI version.
4. Append a line to this file: `Codex regression-verified on Codex v<A.B> on <date> — <name>.`

## Why not verify in this session

The Claude Code environment used to author the v1 port doesn't have either Gemini CLI or Codex CLI installed as separate binaries. The plan's hard rule explicitly anticipated this case:

> "If no Codex machine is available, the Codex regression check is marked **deferred** and Phase 5's `CLAUDE.md` wording must say so honestly."

Same applies to Gemini. Phase 5's wording reflects this status — see `CLAUDE.md` Platform Support section.

## Verification entries

(Append below as machines verify. Format: one line per platform per verification event.)

- _No real-machine verifications yet._
