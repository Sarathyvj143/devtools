# DevTools — Gemini CLI Installation (v1)

**v1 scope: skills only.** Commands and agents are not yet ported to Gemini (deferred to v2). Hooks have no Gemini equivalent — `GEMINI.md` substitutes for the Claude Code SessionStart hook.

There is **no marketplace install** on Gemini. Same as Codex: clone the repo and link it into Gemini's discovery paths.

## Option A — Install as a Gemini extension (recommended)

This loads `GEMINI.md` as the extension's context file and auto-discovers all skills under `skills/`.

### Linux / macOS

```bash
git clone https://github.com/Sarathyvj143/devtools.git ~/.gemini/extensions/devtools
```

Or, if you already cloned the repo elsewhere:

```bash
ln -s /path/to/devtools ~/.gemini/extensions/devtools
```

### Windows

PowerShell (admin, for symlinks) — preferred:

```powershell
New-Item -ItemType Junction -Path "$env:USERPROFILE\.gemini\extensions\devtools" -Target "C:\path\to\devtools"
```

Or `cmd` (no admin required, junction):

```cmd
mkdir %USERPROFILE%\.gemini\extensions 2>nul
mklink /J %USERPROFILE%\.gemini\extensions\devtools C:\path\to\devtools
```

Or Git Bash (developer mode required for symlinks):

```bash
ln -s /c/path/to/devtools ~/.gemini/extensions/devtools
```

### Verify

Start `gemini` in any project. The devtools extension should be active, `GEMINI.md` should be loaded as context, and skills under `skills/` should appear when you run `/tools` or activate one by name.

## Option B — Skills-only install (no extension)

If you don't want the full extension (e.g. you already have devtools installed as a Codex skills symlink), Gemini will discover skills from the user-level skills path:

```bash
ln -s /path/to/devtools/skills ~/.gemini/skills/devtools
```

> **Codex users:** Gemini also discovers from `~/.agents/skills/`, which is the same path Codex uses. If you already followed `.codex/INSTALL.md`, **Gemini support comes for free at the user-tier — no additional symlink needed.** Skills will be auto-discovered.

`GEMINI.md` is **not** loaded in skills-only mode (the extension manifest is what triggers it). You'll get the skills + auto-trigger behavior, but the bootstrap content lives only in the `using-devtools` skill body.

## Updating

There is **no `/plugin update` mechanism on Gemini** — the marketplace flow is Claude-Code-only. To pull the latest plugin contents:

```bash
cd ~/.gemini/extensions/devtools && git pull
```

Or, if installed via Option B:

```bash
cd /path/to/devtools && git pull
```

Same caveat as Codex: skills auto-load on next session. There is no agents-regen step for Gemini in v1 (no agents are installed).

## Tool Name Mapping

For tool-name differences between Claude Code and Gemini CLI, see:
`skills/using-devtools/references/gemini-tools.md`

## What's NOT installed in v1 (and why)

- **Slash commands** (`/remember`, `/orchestrate`, `/assemble-team`, etc.) — Claude Code uses Markdown command files; Gemini requires TOML. Translation deferred to v2a.
- **Subagents** (`agents/`) — frontmatter compatibility between Claude Code and Gemini agent loaders not yet validated. Deferred to v2b.
- **Hooks** (`hooks/hooks.json`) — Gemini has no `SessionStart` event with the same shape. The session-start payload is mirrored into `GEMINI.md` instead.

## Verification

This install procedure is **not yet machine-verified.** Phase 4 of the port plan requires running the procedure on a real Gemini CLI install and capturing the version. Until then:

> **Status:** v1 install path documented; **not verified on Gemini CLI v\<X.Y\>**.

Once Phase 4 completes, this section will be updated with a `Verified on Gemini CLI v<X.Y>` line and a brief verification log.
