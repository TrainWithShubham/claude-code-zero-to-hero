# Troubleshooting

Symptom → cause → fix. Organised by where you're stuck, not by module.

Most problems fall into one of four buckets: **it won't install**, **it won't log in**, **something I configured isn't loading**, or **it's slow or stuck**. Work out which bucket you're in first — the diagnostic commands are different for each.

---

## Start here: the four commands

Before searching for your specific error, run these. They resolve a surprising share of issues on their own.

| Command | Run it when | What it does |
|---|---|---|
| `/doctor` | Anything configuration-shaped | Checks installation health, invalid settings files, unused extensions, duplicate subagent names, and `CLAUDE.md` content Claude can derive from the codebase — then proposes fixes it applies after you confirm |
| `claude doctor` | Claude Code won't start at all | Same diagnostics, read-only, from your shell without starting a session |
| `/context` | "Claude is ignoring my instructions" | Shows what actually loaded — system prompt, tools, MCP tools, subagents and their source, memory files, skills |
| `/status` | A setting isn't taking effect | Which settings sources are active, including whether managed settings are in effect |

**The single most useful diagnostic flag** is `claude --safe-mode`. It starts a session with every customisation disabled — `CLAUDE.md`, skills, plugins, hooks, MCP servers, custom commands and agents — while authentication, model selection, built-in tools, and permissions keep working. If the problem disappears in safe mode, one of those surfaces is the cause.

> Safe mode still applies your organisation's managed hooks and settings policy. Managed plugins, skills, `CLAUDE.md`, and MCP servers are turned off.

If the problem *survives* safe mode, go one level cleaner:

```bash
cd /tmp && CLAUDE_CONFIG_DIR=/tmp/claude-clean claude
```

That bypasses everything under `~/.claude`, and launching from a directory with no `.claude/`, `.mcp.json`, or `CLAUDE.md` skips project configuration too. Expect first-run setup screens — seeing them confirms the clean directory is in effect. On Linux and Windows you'll be asked to log in again, because credentials live under the config directory; on macOS they're in the Keychain and carry over.

---

## "Claude is ignoring my CLAUDE.md"

Run `/context` first. There are only three possibilities, and they need different fixes.

**1. The file isn't in the breakdown at all.** It didn't load. Check its location — and note that **subdirectory `CLAUDE.md` files load on demand**, not at session start. They load when Claude reads a file in that directory with the Read tool. Not at launch, and not when *writing* a file there.

**2. It loaded, but Claude isn't following one instruction.** That's a writing problem, not a loading problem. Adherence drops when an instruction is vague enough to read two ways, when two files conflict, or when the file has grown long enough that individual rules get less attention.

**3. A subagent ignored it.** The built-in **Explore and Plan agents skip `CLAUDE.md` entirely.** Restate the instruction in your delegating prompt. Custom subagents do load it — but critical instructions belong in the agent file body, which becomes its system prompt.

And the distinction Module 7 makes: `CLAUDE.md` is for "we do it this way here." If something must *never* happen, that's a permission rule or a hook, not a memory file.

---

## "My hook never fires"

Run `/hooks`. If your hook isn't listed, it isn't being read.

| Cause | Fix |
|---|---|
| Hooks defined in a standalone file | There is **no standalone hooks file** for project or user config. They go under the `"hooks"` key in `settings.json`. Only plugins load a separate `hooks/hooks.json` |
| `matcher` is a JSON array | It's a **single string**. Use `\|` for multiple tools: `"Edit\|Write"` |
| `matcher` uses `,` on an older version | v2.1.191+ treats `,` as a list separator. Earlier versions read it as a literal character and match nothing — use `\|` |
| `matcher` is lowercase | Matching is **case-sensitive**. Tool names are capitalised: `Bash`, `Edit`, `Write`, `Read` |
| Misspelled tool name | Matches nothing, fails silently |

An array value is a schema error with a wide blast radius: Claude Code rejects the **whole** user, project, or local settings file, and *no* hook from that file appears in `/hooks`. (In managed settings, only the invalid entry is stripped.)

Edits to `settings.json` apply to the running session after a brief file-stability delay — no restart needed. If `/hooks` still shows the old definition after a few seconds, run it again to refresh.

If the hook is listed and still doesn't fire, watch it live: start with `claude --debug` and trigger the tool call. The log records each event, which matchers were checked, and the hook's exit code and output.

---

## "My MCP server isn't there"

Run `/mcp`. A correctly-defined server can still provide no tools:

- **Project-scoped servers in `.mcp.json` need a one-time approval.** If you dismissed the prompt, the server stays disabled until you approve it from `/mcp`.
- **A relative path in `command` or `args`** resolves against the directory you launched from, not the location of `.mcp.json`. Use absolute paths for local scripts; executables on your `PATH` like `npx` or `uvx` are fine as-is.
- **Connected but zero tools** means it started and isn't returning a tool list. Select **Reconnect** from `/mcp`. If it stays at zero, run `claude --debug=mcp` and read the server's stderr at `~/.claude/debug/<session-id>.txt`.

Two location mistakes worth checking before anything else:

| Symptom | Cause |
|---|---|
| Servers in `.mcp.json` never load | The file is under `.claude/`, or uses Claude Desktop's config format. Project MCP config goes at the **repository root** |
| Servers added under `mcpServers` in `settings.json` never appear | `settings.json` doesn't read an `mcpServers` key. Use `.mcp.json`, or `claude mcp add --scope user` |
| Server starts without its environment variables | Set per-server `env` inside the `.mcp.json` entry — that doesn't depend on the launch environment or workspace trust |

---

## "My skill doesn't run"

| Symptom | Fix |
|---|---|
| Not in `/skills` at all | It's at `.claude/skills/name.md`. It needs a **folder**: `.claude/skills/name/SKILL.md` |
| Listed, but Claude never invokes it | Check the badge in `/skills`. A **"user-only"** label means `disable-model-invocation: true` is set. Otherwise the description doesn't match how you phrase the request |

---

## "A setting isn't applying"

Settings merge across managed, user, project, and local scopes. Managed applies first; among the rest the closer scope wins — local, then project, then user. CLI flags and environment variables act as another override layer on top.

The two that catch everyone:

- **`~/.claude.json` is not a settings file.** It holds app state and UI toggles. `permissions`, `hooks`, and `env` belong in `~/.claude/settings.json`. Two different files.
- **`settings.local.json` overrides `settings.json`.** If a value seems ignored, check whether the same key is set locally.

One that looks like a bug and isn't: a `Bash(rm *)` deny rule **won't block `/bin/rm` or `find -delete`**. Prefix rules match the literal command string, not the underlying executable. Add explicit patterns for each variant, or use a `PreToolUse` hook or the sandbox when you need a hard guarantee.

---

## Performance and stability

### High CPU or memory

1. Run `/compact` regularly. If it returns `Not enough messages to compact.`, the conversation has too few turns to summarise — which can happen even with a full context if one large paste filled it
2. Restart between major tasks
3. Add large build directories to `.gitignore`
4. Restart with `claude --safe-mode` to find out whether a plugin, MCP server, or hook is responsible

If memory stays high, `/heapdump` writes a heap snapshot and a `-diagnostics.json` breakdown to `~/Desktop`, and prints a summary. The command doesn't appear in the menu — type it in full.

> **The `.heapsnapshot` file contains every string in the process, including your full conversation and credentials.** Never attach it to a public issue. If you're reporting a leak, attach only `-diagnostics.json`, which carries the statistics and no conversation content.

### `Autocompact is thrashing`

Compaction succeeded but a file or tool output immediately refilled the window several times running, so Claude Code stopped retrying. Recover by:

1. Asking Claude to read the oversized file in chunks — a line range or a single function
2. Running `/compact` with a focus that drops the large output: `/compact keep only the plan and the diff`
3. Moving the large-file work to a **subagent**, so it runs in a separate context window
4. `/clear`, if the earlier conversation is no longer needed

### It's hung

Press `Ctrl+C`. If that doesn't work, close the terminal — **you won't lose the conversation**. Run `claude --resume` in the same directory to pick it back up.

### Search isn't finding files

If the Search tool, `@file` mentions, custom agents, or custom skills can't find things, the bundled `ripgrep` binary may not run on your system. Install your platform's package and switch to it:

```bash
brew install ripgrep          # macOS
sudo apt install ripgrep      # Ubuntu/Debian
apk add ripgrep               # Alpine
pacman -S ripgrep             # Arch
winget install BurntSushi.ripgrep.MSVC   # Windows
```

Then set `USE_BUILTIN_RIPGREP` to `0`, in your shell environment or the `env` block of `settings.json`:

```json
{ "env": { "USE_BUILTIN_RIPGREP": "0" } }
```

Confirm with `claude doctor` — the Search line should show your system ripgrep's path instead of `OK (bundled)`.

**On WSL, search returns fewer results than expected** because of cross-filesystem read penalties. `claude doctor` still shows Search as OK. Either write narrower searches ("find JWT validation in the auth-service package"), move the project to the Linux filesystem (`/home/`) rather than `/mnt/c/`, or run Claude Code natively on Windows.

### Garbled text in an editor's integrated terminal

Boxes, smears, or wrong glyphs in the VS Code, Cursor, or Devin Desktop terminal usually means the terminal's GPU renderer. Run `/terminal-setup` to set `terminal.integrated.gpuAcceleration` to `"off"`, then reload the window.

### A table got cut off

A Markdown table over 200 rows renders the first 200 and a `… N more rows not shown` line. Only the display is capped — the full table is still in the conversation and `/copy` copies every row. For anything that large, ask Claude to write it to a file.

---

## Install and login

The full error-to-fix table lives in [Troubleshoot installation and login](https://code.claude.com/docs/en/troubleshoot-install). The ones that come up most in a classroom:

| What you see | What it means |
|---|---|
| `command not found: claude` | The install directory isn't on your `PATH`. Add it and open a **new** terminal |
| `syntax error near unexpected token '<'`, or PowerShell parse errors quoting HTML | The install script returned an HTML page — usually a proxy or captive portal, not a real script |
| `EACCES` / permission errors during npm install | Use the native installer rather than fighting npm's global directory permissions |
| `Killed`, or exit code 137, on a Linux server | Out of memory. Free memory or add swap |
| `TLS connect error`, `unable to get local issuer certificate` | Corporate CA certificates need configuring |
| `Error loading shared library` on Linux | musl vs glibc binary mismatch |
| `cannot execute binary file: Exec format error` in WSL | A WSL1 native-binary issue |
| `App unavailable in region` | Claude Code isn't available in your country |
| Login loop, `OAuth error`, `403 Forbidden` | Reset your login. In WSL2, SSH, or containers the browser handoff is the usual culprit |

Check you can actually reach the download server:

```bash
curl -sI https://downloads.claude.ai/claude-code-releases/latest
```

A `200` on the first line means you got there. A `403` usually means a proxy or network filter is blocking the host.

---

## Still stuck

1. `/doctor` for the setup checkup, `/mcp` for server status
2. `/debug [issue]` — turns on debug logging for the session and asks Claude to diagnose it using the log output and settings paths
3. `/feedback` reports the problem to Anthropic from inside Claude Code
4. Check [known issues](https://github.com/anthropics/claude-code/issues)
5. **Ask Claude directly.** It has built-in access to its own documentation — often faster than searching

For account, billing, or subscription problems, contact Anthropic support instead: sign in at claude.ai, click your initials, and select **Get help**.

---

**Related:** [Command reference](commands.md) · [Topic index](topics.md) · [Module 5 — permissions, memory, config](../modules/module-05-permissions-memory-config.md) · [Module 7 — hooks](../modules/module-07-skills-commands-hooks.md) · [Module 9 — MCP](../modules/module-09-mcp.md)
