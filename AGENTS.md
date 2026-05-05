# Repository Guidelines

## Project Structure & Module Organization

This repository is a portable AI development toolkit, organized around reusable Markdown and JSON assets.

- `skills/`: workflow skills. Start new skills by copying `skills/_template/` and editing `SKILL.md`.
- `agents/`: specialized agent prompts. Base roles live in `agents/_bases/`; service and composition metadata live under `agents/_profiles/`.
- `commands/`: slash-command definitions such as `remember.md` and `assemble-team.md`.
- `hooks/`: lifecycle hook configuration and scripts, including `hooks.json` and `run-hook.cmd`.
- `docs/`: design specs and implementation plans.
- `memories/`: portable memory content.
- `tests/`: reserved for future tests; currently contains only `.gitkeep`.

## Build, Test, and Development Commands

There is no project-wide build system or package manager configured. Use lightweight validation before committing:

```powershell
rg --files
```

Lists tracked content and helps verify new files are in the expected directory.

```powershell
Get-Content hooks\hooks.json | ConvertFrom-Json
```

Validates hook JSON syntax in PowerShell.

```powershell
git diff --check
```

Checks for whitespace errors before commit.

For plugin consumers, update installed copies with `/plugin update devtools`; refresh generated project agents with `/assemble-team --update`.

## Coding Style & Naming Conventions

Use concise Markdown with clear headings and actionable instructions. Keep filenames lowercase with hyphens for commands and skills, for example `memory-cleanup.md` or `using-devtools`. JSON files should use two-space indentation and stable key ordering where practical. Shell or batch scripts should be cross-platform aware when invoked by hooks; prefer absolute paths in generated runtime instructions.

## Testing Guidelines

No automated test framework is currently defined. For changes to JSON profiles or hooks, validate syntax locally. For Markdown skills, agents, and commands, manually review examples, trigger conditions, and referenced paths. If adding executable logic, place tests under `tests/` and document the exact command required to run them.

## Commit & Pull Request Guidelines

Recent history uses Conventional Commits, such as `feat: ...` and `fix: ...`. Follow that pattern with a short imperative subject and a scoped explanation when helpful.

Pull requests should include a summary of changed skills, agents, commands, or hooks; validation performed; and any compatibility notes for Claude Code, Codex, Cursor, Gemini CLI, or OpenCode. Include screenshots only when UI-facing documentation or marketplace presentation changes.

## Security & Configuration Tips

Do not commit local secrets, `.env` files, generated logs, or machine-specific editor settings. Keep `.mcp.json` as a template unless a change is intentionally portable.
