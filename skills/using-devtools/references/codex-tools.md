# Codex Tool Mapping

When running on Codex, use these equivalent tool names:

| Claude Code | Codex Equivalent |
|------------|-----------------|
| Read       | ReadFile        |
| Write      | WriteFile       |
| Edit       | PatchFile       |
| Glob       | Glob            |
| Grep       | Grep            |
| Bash       | Shell           |

> **Verify against your Codex version.** Current OpenAI Codex CLI primarily exposes a single `shell` tool (with `apply_patch` invoked via shell). The named-tool mappings above derive from earlier Codex SDK conventions and may not match every Codex version. If a named tool is unavailable, fall back to `shell` for the equivalent operation. Source: <https://developers.openai.com/codex/cli/features>.

## Extended mapping (Phase 1)

These tools appear in Claude Code skills, agents, or commands but were missing from the original 6-row table. On Codex, most fall back to `shell` or have no direct equivalent.

| Claude Code         | Codex Equivalent                         | Notes                                                                 |
|---------------------|------------------------------------------|-----------------------------------------------------------------------|
| Agent               | no equivalent — degraded UX              | Codex has no subagent dispatch tool. Skip or invoke as a shell script.|
| Skill               | no equivalent — degraded UX              | Skills are still loaded as docs from `~/.agents/skills/`, but there is no `Skill` invocation tool. The model must read the skill content directly. |
| Task / TaskCreate / TaskUpdate / TaskList / TaskGet / TaskOutput / TaskStop | no equivalent — degraded UX | Codex has no background task system. Track work in the conversation only. |
| TodoWrite           | no equivalent — degraded UX              | Keep todos inline in the chat thread.                                 |
| WebFetch            | first-party web search tool              | Codex has web search; use it for URL lookups.                         |
| WebSearch           | first-party web search tool              | Same as above.                                                        |
| NotebookEdit        | no equivalent — degraded UX              | Use `shell` with the `jupyter` CLI for `.ipynb` edits.                |
| AskUserQuestion     | no equivalent — direct conversation      | Ask the user in the conversation thread.                              |
| EnterPlanMode       | no equivalent — degraded UX              | Codex has its own approval modes (Read-only / Auto / Full Access).    |
| ExitPlanMode        | no equivalent — degraded UX              | Same as above.                                                        |
| ListMcpResourcesTool | MCP via Codex MCP support               | Codex supports Model Context Protocol servers; exact tool name varies by version. |
| ReadMcpResourceTool | MCP via Codex MCP support                | Same as above.                                                        |

## Contributing

To add or correct a mapping:
1. Verify the Codex equivalent against your installed Codex version (run the equivalent of `/tools` if available, or check the Codex docs for your version).
2. Update this file with verified mappings.
3. Cite the Codex docs source for any change.
