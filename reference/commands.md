# Command reference

Every slash command, CLI command, flag, and environment variable the course covers, and where it's taught. Use this when you know what you want to do but not what it's called.

Module numbers link to the module that explains the command properly. This index only lists what a module actually mentions — nothing here is inferred or assumed.

---

## Session and context

| Command | Does | Module |
|---|---|---|
| `/context` | Show what's currently loaded into the context window, including memory files under settings | [5](../modules/module-05-setup-configuration-memory.md) |
| `/compact [instructions]` | Summarize conversation history so far, freeing context space. Accepts optional instructions on what to preserve (e.g. `/compact Focus on code samples and API usage`) | [9](../modules/module-09-slash-commands-skills-subagents.md), [13](../modules/module-13-prompting-strategies-multiagent.md) |
| `/clear` | Drop the conversation entirely and start fresh, with no summary | [9](../modules/module-09-slash-commands-skills-subagents.md), [13](../modules/module-13-prompting-strategies-multiagent.md) |
| `/resume` | Session management command mentioned alongside `/clear` | [9](../modules/module-09-slash-commands-skills-subagents.md) |
| `/help` | Show the full list of built-in slash commands | [9](../modules/module-09-slash-commands-skills-subagents.md) |
| `/diff` | Review uncommitted changes | [9](../modules/module-09-slash-commands-skills-subagents.md) |

## Configuration and memory

| Command | Does | Module |
|---|---|---|
| `/init` | Generate a starting `CLAUDE.md` from the codebase | [5](../modules/module-05-setup-configuration-memory.md), [9](../modules/module-09-slash-commands-skills-subagents.md) |
| `/memory` | Browse, edit, or toggle auto memory | [5](../modules/module-05-setup-configuration-memory.md) |
| `/permissions` | Setup-stage command for managing permission rules | [9](../modules/module-09-slash-commands-skills-subagents.md) |
| `/doctor` | Bundled skill mentioned as an example of a skill shipped with Claude Code | [9](../modules/module-09-slash-commands-skills-subagents.md) |

## Planning and effort

| Command | Does | Module |
|---|---|---|
| `/plan` | Prefix a prompt with plan mode (Claude reads and reasons but can't edit) | [7](../modules/module-07-planning-a-project.md), [9](../modules/module-09-slash-commands-skills-subagents.md) |
| `/effort` | In-session command for controlling how much reasoning effort Claude spends | [9](../modules/module-09-slash-commands-skills-subagents.md) |
| `/code-review` | Bundled skill/command for reviewing a diff | [9](../modules/module-09-slash-commands-skills-subagents.md) |

## Model

| Command | Does | Module |
|---|---|---|
| `/model` | Open the model picker; switch models mid-session. Enter saves the choice as your default, `s` switches for the current session only | [4](../modules/module-04-getting-started.md), [9](../modules/module-09-slash-commands-skills-subagents.md) |
| `/login` | Authenticate or switch accounts inside a running session | [4](../modules/module-04-getting-started.md) |

## MCP

| Command | Does | Module |
|---|---|---|
| `/mcp` | Show connected MCP servers and connection status; reauthenticate a server whose token expired | [10](../modules/module-10-mcp-foundations.md) |

## Plugins

| Command | Does | Module |
|---|---|---|
| `/plugin` | Browse and install skills/plugins from a marketplace | [9](../modules/module-09-slash-commands-skills-subagents.md) |
| `/plugin install <skill>@<marketplace>` | Install a specific skill or plugin from a named marketplace | [9](../modules/module-09-slash-commands-skills-subagents.md) |
| `/reload-plugins` | Apply changes to a locally-loaded plugin without restarting the session | [11](../modules/module-11-automation-subagents-plugins.md) |
| `/skill-name` | Directly invoke a skill by its name | [9](../modules/module-09-slash-commands-skills-subagents.md) |

## GitHub / CI

| Command | Does | Module |
|---|---|---|
| `/install-github-app` | Set up the Claude GitHub App, an auth secret, and a workflow PR for a repository in one command. Requires `gh` installed/authenticated and admin access to the repo | [14](../modules/module-14-cicd-and-reliability.md) |

---

## CLI commands and flags

| Command / flag | Does | Module |
|---|---|---|
| `claude` | Start a session in the current directory | [4](../modules/module-04-getting-started.md) |
| `claude --version` | Check the installed version | [4](../modules/module-04-getting-started.md) |
| `claude --model opus` (`--model`) | Start a session on a specific model | [4](../modules/module-04-getting-started.md) |
| `claude --permission-mode plan` (`--permission-mode`) | Start a session in a specific permission mode | [5](../modules/module-05-setup-configuration-memory.md), [7](../modules/module-07-planning-a-project.md), [8](../modules/module-08-executing-and-building.md) |
| `claude --settings` | Pass settings values for one invocation (command-line precedence tier) | [5](../modules/module-05-setup-configuration-memory.md) |
| `claude --plugin-dir ./my-plugin` (`--plugin-dir`) | Load a plugin from a local directory for one session, without installing it | [11](../modules/module-11-automation-subagents-plugins.md) |
| `claude mcp add --transport http <name> <url>` (`--transport http`) | Add a remote MCP server over HTTP | [10](../modules/module-10-mcp-foundations.md) |
| `claude mcp add --transport http <name> <url> --header "Authorization: Bearer <token>"` (`--header`) | Add an HTTP MCP server with an auth header | [10](../modules/module-10-mcp-foundations.md) |
| `claude mcp add <name> -- <command> [args...]` | Add a local stdio MCP server as a spawned process | [10](../modules/module-10-mcp-foundations.md) |
| `claude mcp add-json <name> '<json>'` | Add an MCP server from a raw JSON config (e.g. SSE, WebSocket) | [10](../modules/module-10-mcp-foundations.md) |
| `--scope project` | Share an MCP server with the team via `.mcp.json` | [10](../modules/module-10-mcp-foundations.md) |
| `--scope user` | Make an MCP server available across every project | [10](../modules/module-10-mcp-foundations.md) |
| `claude mcp list` | List configured MCP servers | [10](../modules/module-10-mcp-foundations.md) |
| `claude mcp get <name>` | Show details/diagnostics for one MCP server | [10](../modules/module-10-mcp-foundations.md) |
| `claude mcp remove <name>` | Remove an MCP server from config | [10](../modules/module-10-mcp-foundations.md) |
| `claude mcp login <name>` | OAuth sign-in for an MCP server that needs it | [10](../modules/module-10-mcp-foundations.md) |
| `claude mcp logout <name>` | Sign out of an MCP server | [10](../modules/module-10-mcp-foundations.md) |

---

## Environment variables

| Variable | Does | Module |
|---|---|---|
| `ANTHROPIC_API_KEY` | Authenticate via API key; setting it skips the browser login prompt and switches the auth flow | [4](../modules/module-04-getting-started.md), [14](../modules/module-14-cicd-and-reliability.md) |
| `CLAUDE_CODE_OAUTH_TOKEN` | One-year OAuth token used as the CI authentication secret in place of an API key | [14](../modules/module-14-cicd-and-reliability.md) |
| `GITHUB_TOKEN` | GitHub's default workflow token — commits made with it don't trigger further workflow runs, a common CI-not-triggering pitfall | [14](../modules/module-14-cicd-and-reliability.md) |
| `CLAUDE_PROJECT_DIR` | Available inside hook commands to reference the project root (e.g. `${CLAUDE_PROJECT_DIR}/.claude/hooks/block-rm.sh`) | [11](../modules/module-11-automation-subagents-plugins.md) |

---

## Keyboard shortcuts

| Shortcut | Does | Module |
|---|---|---|
| `Shift+Tab` | Cycle permission modes (`default → acceptEdits → plan → default`, with other modes slotting in if enabled) | [5](../modules/module-05-setup-configuration-memory.md), [7](../modules/module-07-planning-a-project.md), [8](../modules/module-08-executing-and-building.md) |
| `!` (prefix) | Run the rest of the line as a shell command directly, no model call — output still lands in the conversation | [6](../modules/module-06-built-in-tools-codebases.md) |

---

## Environment variables — Ollama (local models)

| Variable | Value | Purpose | Module |
|---|---|---|---|
| `ANTHROPIC_AUTH_TOKEN` | `ollama` | Placeholder token Ollama's local server accepts | [4](../modules/module-04-getting-started.md) |
| `ANTHROPIC_API_KEY` | `""` | Cleared so Claude Code doesn't try to use a real API key | [4](../modules/module-04-getting-started.md) |
| `ANTHROPIC_BASE_URL` | `http://localhost:11434` | Redirects Claude Code's API calls to the local Ollama server | [4](../modules/module-04-getting-started.md) |

Note: this integration is documented and maintained by Ollama (`docs.ollama.com/integrations/claude-code`), not by Anthropic — there's no `--provider` flag; Claude Code doesn't know it's talking to Ollama, it's just pointed at a different base URL.

---

Commands change — Claude Code ships weekly. If one here doesn't exist anymore, check [what's new](https://code.claude.com/docs/en/whats-new) before assuming this file is still accurate.
