# DevTools — Codex Installation

## Setup

1. Clone the repo:

   ```bash
   git clone https://github.com/your-username/devtools.git ~/.codex/devtools
   ```

2. Symlink skills into Codex's skills directory:

   ```bash
   mkdir -p ~/.agents/skills
   ln -s ~/.codex/devtools/skills ~/.agents/skills/devtools
   ```

   > **Note:** Verify symlink path against current Codex docs — path may vary by version.

3. For tool name differences between Claude Code and Codex, see:
   `skills/using-devtools/references/codex-tools.md`

## Updating

```bash
cd ~/.codex/devtools && git pull
```
