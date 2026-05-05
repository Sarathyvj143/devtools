---
description: Check whether the devtools plugin has been updated since this project's agents were generated
argument-hint: (no arguments)
allowed-tools: [Read, Glob, Bash]
---

# Check Updates

$ARGUMENTS

## Instructions

This is a manual replacement for the SessionStart "plugin updated" notice. On Claude Code, the SessionStart hook already runs this logic automatically; this command lets the user re-run it on demand.

### Steps

1. **Find the plugin install path.** On Claude Code, read `~/.claude/plugins/installed_plugins.json` and find the entry whose key contains `devtools`. The `installPath` field is the plugin root.

   Bash:
   ```bash
   PLUGIN_PATH=$(python -c "
   import json
   d=json.load(open('$HOME/.claude/plugins/installed_plugins.json'))
   for k,v in d.items():
     if 'devtools' in k:
       print(v[0]['installPath'])
   " 2>/dev/null)
   ```

   PowerShell:
   ```powershell
   $pluginPath = python -c "import json; d=json.load(open(r'$env:USERPROFILE\.claude\plugins\installed_plugins.json')); print([v[0]['installPath'] for k,v in d.items() if 'devtools' in k][0])"
   ```

2. **Get the plugin's current git commit:**
   ```bash
   PLUGIN_COMMIT=$(cd "$PLUGIN_PATH" && git rev-parse --short HEAD 2>/dev/null || echo "unknown")
   ```

3. **Read the project's recorded commit:**
   ```bash
   if [ -f .claude/team-config.json ]; then
     GENERATED_COMMIT=$(python -c "import json; print(json.load(open('.claude/team-config.json')).get('plugin_commit','unknown')[:7])" 2>/dev/null)
   else
     echo "No .claude/team-config.json — run /assemble-team first."
     exit 0
   fi
   ```

4. **Compare and report.** Three cases:

   - **No team-config.json exists:** Tell the user *"No team has been assembled in this project yet. Run /assemble-team to generate agents."*
   - **Plugin commit matches recorded commit:** Tell the user *"Plugin and project agents are in sync (commit `<short>`)."*
   - **Plugin commit differs from recorded commit:** Print:
     ```
     DevTools plugin updated since agents were generated.
       Generated with: <recorded-commit>
       Current plugin: <plugin-commit>

       Run /assemble-team --update to refresh agents.
     ```

5. **List changed plugin files** (helpful context). Run `git diff --name-only <generated-commit> <plugin-commit>` inside the plugin directory and group results:
   - `agents/_bases/*` — base template changes that may trigger agent regeneration
   - `agents/_profiles/*` — service detection or composition changes
   - `skills/*` — new or updated skills
   - `commands/*` — command additions/changes
   - Anything else — informational

### Platform Notes

- **Claude Code:** uses `~/.claude/plugins/installed_plugins.json` to locate the plugin. SessionStart already injects an `<update-notice>` automatically — this command is for manual re-checks.
- **Codex:** the plugin install path is wherever the user cloned the repo (typically `~/.codex/devtools`). There is no `installed_plugins.json` registry on Codex. The user must pass the path or the command should fall back to checking `~/.codex/devtools` and `~/.gemini/extensions/devtools` by convention. **Note: this command is currently not installed on Codex** (Codex installs only `skills/`). If invoked there manually via the skill loader, treat the logic as advisory.
- **Gemini:** the plugin install path is `~/.gemini/extensions/devtools`. There is no SessionStart hook on Gemini — this command is the only update-check mechanism. Replace `~/.claude/plugins/...` lookup with a check at `~/.gemini/extensions/devtools` (or wherever the user installed per `GEMINI-INSTALL.md`).

### Output

Concise, single-paragraph summary in conversation. If a change exists, include:
- The two commit hashes (recorded vs current)
- Top 3-5 changed files (most relevant ones — base templates first)
- The recommended next action (`/assemble-team --update` for agent refresh)
