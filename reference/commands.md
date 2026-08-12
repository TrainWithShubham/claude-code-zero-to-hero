# Command reference

Every command and flag the course covers, and where it's taught. Use this when you know what you want to do but not what it's called.

Module numbers link to the module that explains the command properly.

---

## Session and context

| Command | Does | Module |
|---|---|---|
| `/clear` | Reset context between unrelated tasks | [2](../modules/module-02-prompt-engineering.md) |
| `/compact [instructions]` | Replace history with a summary, optionally focused | [4](../modules/module-04-agentic-loop-tools.md) |
| `/context` | Show what's currently filling the context window, including loaded memory files | [4](../modules/module-04-agentic-loop-tools.md) |
| `/rewind` | Restore or summarize code and conversation from an earlier point | [6](../modules/module-06-planning-real-projects.md) |
| `/branch [name]` | Copy the conversation and switch into the copy | [6](../modules/module-06-planning-real-projects.md) |
| `/resume [name]` | Switch to another conversation | [6](../modules/module-06-planning-real-projects.md) |
| `/rename <name>` | Name the current session so you can resume it by name | [6](../modules/module-06-planning-real-projects.md) |
| `/btw` | Ask a side question whose answer never enters conversation history | [2](../modules/module-02-prompt-engineering.md) |
| `/export` | Copy or save the conversation as readable text | [6](../modules/module-06-planning-real-projects.md) |
| `/copy [N]` | Copy the last response — or the Nth-latest — to the clipboard. Press `w` in the code-block picker to write to a file instead | [Troubleshooting](troubleshooting.md) |

## Configuration and diagnostics

| Command | Does | Module |
|---|---|---|
| `/init` | Generate a starter `CLAUDE.md` from your codebase | [2](../modules/module-02-prompt-engineering.md) |
| `/memory` | Browse and edit memory files; toggle auto memory | [5](../modules/module-05-permissions-memory-config.md) |
| `/config` | Settings, including output style and auto-update channel | [3](../modules/module-03-installing-everywhere.md) |
| `/doctor` (alias `/checkup`) | Full setup checkup; proposes `CLAUDE.md` trims | [5](../modules/module-05-permissions-memory-config.md) |
| `/hooks` | Show what hooks are registered | [7](../modules/module-07-skills-commands-hooks.md) |
| `/mcp` | Inspect and authenticate MCP servers | [9](../modules/module-09-mcp.md) |
| `/status` | Which credential is active | [3](../modules/module-03-installing-everywhere.md) |
| `/usage` | What's driving your plan limits, by skill, subagent, plugin, MCP server | [12](../modules/module-12-cicd-review-security.md) |
| `/theme` | Theme picker | [3](../modules/module-03-installing-everywhere.md) |
| `/login` · `/logout` | Authenticate or sign out | [3](../modules/module-03-installing-everywhere.md) |
| `/skills` | Available skills from project, user, and plugin sources | [Troubleshooting](troubleshooting.md) |
| `/debug [issue]` | Turn on debug logging mid-session and have Claude diagnose from the log | [Troubleshooting](troubleshooting.md) |
| `/heapdump` | Heap snapshot + memory breakdown to `~/Desktop`. Doesn't appear in the menu — type it in full | [Troubleshooting](troubleshooting.md) |
| `/terminal-setup` | Fix garbled text in an integrated terminal by disabling GPU acceleration | [Troubleshooting](troubleshooting.md) |
| `/feedback` | Report a problem to Anthropic from inside Claude Code | [Troubleshooting](troubleshooting.md) |

## Permissions and sandboxing

| Command | Does | Module |
|---|---|---|
| `Shift+Tab` | Cycle permission modes | [5](../modules/module-05-permissions-memory-config.md) |
| `/permissions` | Manage permission rules (CLI only — not in Desktop) | [5](../modules/module-05-permissions-memory-config.md) |
| `/sandbox` | Sandbox panel: Mode, Overrides, Dependencies | [4](../modules/module-04-agentic-loop-tools.md) |

## Planning and review

| Command | Does | Module |
|---|---|---|
| `/plan` | Prefix a single prompt with plan mode | [6](../modules/module-06-planning-real-projects.md) |
| `/code-review [level] [target]` | Review a diff. `--fix` applies, `--comment` posts to a PR | [12](../modules/module-12-cicd-review-security.md) |
| `/code-review ultra` | Escalate to the deeper cloud review | [12](../modules/module-12-cicd-review-security.md) |
| `/review` | Alias of `/code-review` | [12](../modules/module-12-cicd-review-security.md) |
| `/simplify` | Cleanup-only review — applies fixes without hunting bugs | [12](../modules/module-12-cicd-review-security.md) |
| `/security-review` | One security pass over the current branch | [12](../modules/module-12-cicd-review-security.md) |
| `/goal [condition\|clear]` | Keep Claude working across turns until a condition holds. No argument shows the current goal | [2](../modules/module-02-prompt-engineering.md) |

## Orchestration

| Command | Does | Module |
|---|---|---|
| `claude agents` | Agent view — every session on one screen | [10](../modules/module-10-subagents-orchestration.md) |
| `/agents` | Prints a reminder to edit `.claude/agents/` (no longer a wizard) | [10](../modules/module-10-subagents-orchestration.md) |
| `/workflows` | Watch, pause, stop, or **save** a dynamic workflow run | [10](../modules/module-10-subagents-orchestration.md) |
| `/deep-research <question>` | The bundled research workflow | [10](../modules/module-10-subagents-orchestration.md) |
| `/effort <level>` | `low` · `medium` · `high` · `xhigh` · `max` · `ultracode` | [10](../modules/module-10-subagents-orchestration.md) |
| `/model` | Switch model | [10](../modules/module-10-subagents-orchestration.md) |

## Extensions

| Command | Does | Module |
|---|---|---|
| `/plugin` | Plugin panel: Discover, Installed, Marketplaces, Errors | [8](../modules/module-08-plugin-ecosystem.md) |
| `/plugin marketplace add <source>` | Register a catalog | [8](../modules/module-08-plugin-ecosystem.md) |
| `/plugin install <name>@<marketplace>` | Install a plugin | [8](../modules/module-08-plugin-ecosystem.md) |
| `/reload-plugins` | Apply plugin changes without restarting | [8](../modules/module-08-plugin-ecosystem.md) |
| `/claude-security` | Scan codebase, scan changes, suggest patches | [8](../modules/module-08-plugin-ecosystem.md) |
| `claude mcp add --transport http\|sse\|stdio` | Add an MCP server | [9](../modules/module-09-mcp.md) |
| `claude mcp login` · `logout` · `list` | Authenticate and inspect MCP servers | [9](../modules/module-09-mcp.md) |

## Surfaces and remote

| Command | Does | Module |
|---|---|---|
| `/ide` | Connect an external terminal session to your running VS Code or JetBrains IDE | [11](../modules/module-11-every-surface.md) |
| `/chrome` | Chrome integration status, permissions, browser choice | [11](../modules/module-11-every-surface.md) |
| `claude remote-control` | Drive this session from a phone or browser | [11](../modules/module-11-every-surface.md) |
| `/schedule` (alias `/routines`) | Create and manage cloud routines | [11](../modules/module-11-every-surface.md) |
| `/install-github-app` | Set up GitHub Actions in one command | [12](../modules/module-12-cicd-review-security.md) |
| `Ctrl+]` | Reopen the most recent artifact | [15](../modules/module-15-capstone.md) |

## CLI flags worth knowing

| Flag | Does | Module |
|---|---|---|
| `-p "<prompt>"` (`--print`) | Non-interactive run. Exits `0` on success, non-zero on failure | [12](../modules/module-12-cicd-review-security.md#1-headless-mode--claude-code-without-a-terminal) |
| `--bare` | Skip auto-discovery of hooks, skills, plugins, MCP, auto memory, and `CLAUDE.md`. **Use this in CI** | [12](../modules/module-12-cicd-review-security.md#1-headless-mode--claude-code-without-a-terminal) |
| `--output-format text\|json\|stream-json` | Response shape. `json` adds `result`, `session_id`, and cost | [12](../modules/module-12-cicd-review-security.md#1-headless-mode--claude-code-without-a-terminal) |
| `--json-schema '<schema>'` | With `--output-format json`, validate the result into `structured_output` | [12](../modules/module-12-cicd-review-security.md#1-headless-mode--claude-code-without-a-terminal) |
| `--include-partial-messages` | With `stream-json` and `--verbose`, stream tokens as they're generated | [12](../modules/module-12-cicd-review-security.md#1-headless-mode--claude-code-without-a-terminal) |
| `--allowedTools "Bash,Read,Edit"` | Pre-approve tools. Uses permission rule syntax — `Bash(git diff *)`, space before `*` | [12](../modules/module-12-cicd-review-security.md#1-headless-mode--claude-code-without-a-terminal) |
| `--append-system-prompt "<text>"` | Add instructions while keeping default behaviour | [12](../modules/module-12-cicd-review-security.md#1-headless-mode--claude-code-without-a-terminal) |
| `--safe-mode` | Start with every customisation disabled. **The first debugging step** | [Troubleshooting](troubleshooting.md) |
| `--debug` · `--debug=mcp` | Log hook evaluation or MCP stderr to `~/.claude/debug/<session-id>.txt` | [Troubleshooting](troubleshooting.md) |
| `--continue` · `--resume [name]` | Resume the last or a named session | [6](../modules/module-06-planning-real-projects.md) |
| `--from-pr <number>` | Find the session that created a PR | [6](../modules/module-06-planning-real-projects.md) |
| `--fork-session` | Branch a session from the command line | [6](../modules/module-06-planning-real-projects.md) |
| `-n <name>` | Name a session at startup | [6](../modules/module-06-planning-real-projects.md) |
| `--permission-mode <mode>` | Start in a specific permission mode | [5](../modules/module-05-permissions-memory-config.md) |
| `--chrome` | Enable Chrome integration for the session | [11](../modules/module-11-every-surface.md) |
| `--channels plugin:<name>@<marketplace>` | Enable a channel | [9](../modules/module-09-mcp.md) |
| `--add-dir <path>` | Grant access to another directory | [6](../modules/module-06-planning-real-projects.md) |
| `--worktree` | Start in a new git worktree | [10](../modules/module-10-subagents-orchestration.md) |
| `--teammate-mode <mode>` | Agent team display mode | [10](../modules/module-10-subagents-orchestration.md) |
| `claude setup-token` | One-year OAuth token for CI | [3](../modules/module-03-installing-everywhere.md) |
| `claude doctor` · `claude --version` | Diagnostics without starting a session | [3](../modules/module-03-installing-everywhere.md) |
| `claude gateway --config <file>` | Run the Claude apps gateway | [13](../modules/module-13-infrastructure-cloud-enterprise.md) |

## Environment variables

| Variable | Does | Module |
|---|---|---|
| `BASH_MAX_OUTPUT_LENGTH` | Bash read-back window — 30,000 default, 150,000 ceiling | [4](../modules/module-04-agentic-loop-tools.md) |
| `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` | Enable agent teams | [10](../modules/module-10-subagents-orchestration.md) |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1` | Turn off auto memory | [5](../modules/module-05-permissions-memory-config.md) |
| `CLAUDE_CODE_USE_BEDROCK` · `_VERTEX` · `_FOUNDRY` | Route through a cloud provider | [3](../modules/module-03-installing-everywhere.md) |
| `CLAUDE_CODE_ENABLE_TELEMETRY=1` + `OTEL_*` | OpenTelemetry export | [13](../modules/module-13-infrastructure-cloud-enterprise.md) |
| `CLAUDE_CONFIG_DIR` | Move config off `~/.claude` — also the way to get a clean session for debugging | [14](../modules/module-14-agent-sdk.md), [Troubleshooting](troubleshooting.md) |
| `USE_BUILTIN_RIPGREP=0` | Use your system `ripgrep` when the bundled binary won't run | [Troubleshooting](troubleshooting.md) |
| `CLAUDE_CODE_PRINT_BG_WAIT_CEILING_MS` | Cap how long `claude -p` waits on background subagents (10 min default; `0` = no limit) | [12](../modules/module-12-cicd-review-security.md#1-headless-mode--claude-code-without-a-terminal) |
| `ENABLE_TOOL_SEARCH=auto` | Threshold-based MCP tool loading | [9](../modules/module-09-mcp.md) |
| `DISABLE_AUTOUPDATER=1` | Stop background updates | [3](../modules/module-03-installing-everywhere.md) |

---

Commands change. If one here doesn't exist, check [what's new](https://code.claude.com/docs/en/whats-new) — and see [Module 16](../modules/module-16-staying-sharp.md) for why `/fork` became `/branch`.
