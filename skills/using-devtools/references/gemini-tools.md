# Gemini CLI Tool Mapping

When running on Gemini CLI, use these equivalent tool names. Sourced from the Gemini CLI tool reference (<https://geminicli.com/docs/reference/tools/>).

| Claude Code | Gemini CLI Equivalent |
|-------------|-----------------------|
| Read        | read_file             |
| Write       | write_file            |
| Edit        | replace               |
| Glob        | glob                  |
| Grep        | grep_search           |
| Bash        | run_shell_command     |

## Extended mapping

Same row order as `codex-tools.md` for diffability.

| Claude Code         | Gemini CLI Equivalent                                  | Notes                                                                 |
|---------------------|--------------------------------------------------------|-----------------------------------------------------------------------|
| Agent               | no equivalent — degraded UX                            | Gemini extensions support an `agents/` directory, but there is no `Agent` dispatch tool exposed to the model in v1. Subagent ports are deferred to v2b. |
| Skill               | activate_skill                                         | Gemini's skill activation flow is functionally identical to Claude Code's `Skill` tool. Auto-triggers from skill `description`. |
| Task / TaskCreate / TaskUpdate / TaskList / TaskGet / TaskOutput / TaskStop | tracker_create_task / tracker_update_task / tracker_list_tasks / tracker_get_task / (no direct stop/output) — **experimental** | Gemini has experimental tracker tools. Treat as best-effort; subject to change. |
| TodoWrite           | write_todos                                            | Direct equivalent.                                                    |
| WebFetch            | web_fetch                                              | Direct equivalent.                                                    |
| WebSearch           | google_web_search                                      | Direct equivalent.                                                    |
| NotebookEdit        | no equivalent — degraded UX                            | Use `run_shell_command` with `jupyter` CLI.                           |
| AskUserQuestion     | ask_user                                               | Direct equivalent.                                                    |
| EnterPlanMode       | enter_plan_mode                                        | Direct equivalent.                                                    |
| ExitPlanMode        | exit_plan_mode                                         | Direct equivalent.                                                    |
| ListMcpResourcesTool | list_mcp_resources                                    | Direct equivalent.                                                    |
| ReadMcpResourceTool | read_mcp_resource                                      | Direct equivalent.                                                    |

## Gemini-only tools (no Claude Code equivalent)

These exist on Gemini and have no Claude Code counterpart. Surfaced here for completeness so a skill body that needs persistence/memory on Gemini knows it has options.

| Gemini CLI Tool      | Purpose                                                                |
|----------------------|------------------------------------------------------------------------|
| save_memory          | Persists facts to GEMINI.md (Gemini's equivalent of /remember).        |
| read_many_files      | Reads + concatenates multiple files in one call (no CC equivalent).    |
| list_directory       | Lists directory contents (Claude Code uses Glob or Bash `ls`).         |
| get_internal_docs    | Accesses Gemini CLI's own documentation.                               |

## Contributing

To add or correct a mapping:
1. Run `/tools` in Gemini CLI to see the complete tool list for your version.
2. Verify the Gemini equivalent against the official tool reference.
3. Update this file.
