<!-- module: 11 | phase: 3 | format: reference | last_verified: 2026-09-15 -->
# Module 11 — Advanced Automation with Sub-Agents and Plugins

By the end of this module you can automate repeatable work using hooks, plugins, and specialised agents that run tasks in the background.

## Hooks, Plugins and Automation Agents

**Hooks: reacting to lifecycle events**

Hooks let you attach your own commands, HTTP calls, or even prompts to specific points in a Claude Code session, without waiting for Claude to remember to do something on its own.

- Hooks fire on named lifecycle events. The most commonly used ones: `PreToolUse` (before a tool call runs — can block it), `PostToolUse` (after a tool call succeeds), `SessionStart` and `SessionEnd`, `UserPromptSubmit` (before Claude processes what you typed), and `Stop` (when Claude finishes a turn). Claude Code also fires more specialised events for subagents, file changes, worktrees, and model switches — the full list lives in the official hooks reference, since new events get added as the product evolves.
- A hook is configured as JSON with a `matcher` (which tool or condition it applies to) and one or more handlers. A handler has a `type` — `command` (run a local script), `http` (call an endpoint), `mcp_tool` (call a tool from a connected MCP server), `prompt`, or `agent` — plus the fields that type needs (a `command` path, a `url`, a `prompt` string).
- Hooks live in `.claude/settings.json` (shared, commit it), `.claude/settings.local.json` (personal, gitignored), `~/.claude/settings.json` (every project on your machine), or bundled inside a plugin's `hooks/hooks.json`. Priority and shareability differ by location — project-level `settings.json` is the right place for a rule the whole team should follow.
- A command hook receives structured JSON on stdin describing the event (session ID, working directory, the tool call in question, and more) and communicates a decision back either through its exit code or by printing JSON. Exit code `2` is the reliable way to block an action outright; JSON output lets you also return a reason or extra context for Claude to see.

Example: block a destructive shell command before it runs.

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/block-rm.sh"
          }
        ]
      }
    ]
  }
}
```

Example: lint every file Claude edits, automatically, no need to ask.

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "./scripts/lint-changed-file.sh" }
        ]
      }
    ]
  }
}
```

**Plugins: bundling automation so it travels**

A plugin packages skills, agents, hooks, and MCP servers into one shareable unit, instead of leaving them scattered across a personal `.claude/` directory that only you have.

- A plugin is a directory with a `.claude-plugin/plugin.json` manifest (name, description, version, optional author) at its root. Everything else — `skills/`, `commands/`, `agents/`, `hooks/hooks.json`, `.mcp.json`, `.lsp.json` — sits at the plugin root too, never nested inside `.claude-plugin/`.
- Plugin skills are namespaced automatically: a `hello` skill inside a plugin named `my-toolkit` becomes `/my-toolkit:hello`, so two plugins can each ship a skill with the same short name without colliding.
- Test a plugin locally with `claude --plugin-dir ./my-plugin` before sharing it — this loads it for that session only, no install step needed. After editing a loaded plugin, `/reload-plugins` picks up the changes without restarting the session.
- Plugins are distributed through **marketplaces** — a repository (or a hosted list) that catalogs installable plugins. Anthropic maintains an official curated marketplace (`claude-plugins-official`, registered automatically the first time you run Claude Code interactively) and a community marketplace (`claude-community`) that users add explicitly and install from as `@claude-community`. A team can also run its own private marketplace for internal plugins.
- The practical reason this matters for automation specifically: a hook that only exists in your personal `settings.local.json` dies with your machine. The same hook shipped inside a plugin's `hooks/hooks.json` travels with the plugin — anyone on the team who installs it gets the identical automation, and updating the plugin updates everyone's copy.

**Specialised background agents for repeatable automation**

Beyond one-off hooks, Claude Code lets you define custom sub-agents — focused agents with their own system prompt, tool restrictions, and sometimes their own model — for jobs you'd otherwise redo manually every time. This module builds on the sub-agent basics from Module 9; what's new here is treating a sub-agent as a piece of standing automation rather than a one-time delegate.

- Define an agent as a file with YAML frontmatter (name, description, allowed tools, optional model) plus a body describing its job — the same shape whether it lives standalone in `.claude/agents/` or inside a plugin's `agents/` directory.
- A narrow, well-described agent gets invoked automatically when its description matches the task at hand, or explicitly by name or by @-mention. Naming a repeatable check as an agent — a testing agent, a verification agent, a release-notes agent — turns "remember to check this" into "the agent for this always runs," which is the same shift hooks make for lower-level tool calls.
- Plugins can go one step further and set a default agent for the whole session via a `settings.json` at the plugin root with an `agent` key — enabling the plugin changes how the main thread behaves, not just what extra tools or skills are available.
- Background monitors (a `monitors/monitors.json` file inside a plugin) go further still: a monitor runs a long-lived command — tailing a log file, polling a status endpoint — and streams its output to Claude as notifications during the session, without you asking it to check.

**Common pitfalls**

- Writing a hook that blocks silently. If a `PreToolUse` hook denies an action, give it a clear `permissionDecisionReason` or stderr message — a bare non-zero exit with no explanation just looks like a mysterious failure.
- Putting `commands/`, `agents/`, or `hooks/` inside a plugin's `.claude-plugin/` directory. Only `plugin.json` belongs there; everything else sits at the plugin root.
- Assuming a plugin update reaches everyone instantly. Depending on the source, updates may require a version bump in `plugin.json` or a `/reload-plugins` run, and MCP server or LSP changes inside a plugin may need a fresh session to take effect in a non-interactive context.
- Writing an automation agent with an overly broad description, so it fires (or gets picked) for tasks it wasn't designed for. Keep the description specific to the one job the agent exists to do.

**Key takeaways**

- Hooks attach commands, HTTP calls, or prompts to lifecycle events (`PreToolUse`, `PostToolUse`, `SessionStart`, `Stop`, and others) and are configured as JSON matchers plus handlers in a settings file or a plugin.
- Plugins bundle skills, agents, hooks, and MCP servers behind one manifest (`.claude-plugin/plugin.json`) so automation can be shared through a marketplace instead of living only on one machine.
- Specialised sub-agents turn a repeatable manual check — testing, verification, review — into standing automation that runs the same way every time, and plugins can even set one as the session's default agent.
- Test locally with `--plugin-dir` and `/reload-plugins` before sharing a plugin, and always give blocking hooks a clear reason.

<!-- nav -->
[← Module 10: MCP Foundations](module-10-mcp-foundations.md) | [Module 12: Advanced Project: URL Shortener →](module-12-url-shortener-project.md)
