# DevTools -- Codex Installation

## Setup (Linux/Mac)

1. Clone the repo:

   ```bash
   git clone https://github.com/Sarathyvj143/devtools.git ~/.codex/devtools
   ```

2. Symlink skills into Codex's skills directory:

   ```bash
   mkdir -p ~/.agents/skills
   ln -s ~/.codex/devtools/skills ~/.agents/skills/devtools
   ```

   > **Note:** Verify symlink path against current Codex docs -- path may vary by version.

## Setup (Windows)

1. Clone the repo:

   ```powershell
   git clone https://github.com/Sarathyvj143/devtools.git %USERPROFILE%\.codex\devtools
   ```

2. Create a directory junction (symlink alternative -- no admin required):

   ```cmd
   mkdir %USERPROFILE%\.agents\skills 2>nul
   mklink /J %USERPROFILE%\.agents\skills\devtools %USERPROFILE%\.codex\devtools\skills
   ```

   Or in PowerShell (requires admin):
   ```powershell
   New-Item -ItemType Junction -Path "$env:USERPROFILE\.agents\skills\devtools" -Target "$env:USERPROFILE\.codex\devtools\skills"
   ```

   Or in Git Bash:
   ```bash
   mkdir -p ~/.agents/skills
   # Git Bash on Windows supports symlinks if developer mode is enabled
   ln -s ~/.codex/devtools/skills ~/.agents/skills/devtools
   ```

## Tool Name Mapping

For tool name differences between Claude Code and Codex, see:
`skills/using-devtools/references/codex-tools.md`

## Updating

There is **no `/plugin update` mechanism on Codex** — the marketplace flow is Claude-Code-only. To pull the latest plugin contents:

```bash
cd ~/.codex/devtools && git pull
```

On Windows (cmd):
```cmd
cd %USERPROFILE%\.codex\devtools && git pull
```

Skills auto-load on next session via the existing symlink. There is no agents-regen step on Codex in v1 (no agents are installed).
