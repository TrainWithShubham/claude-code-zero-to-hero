<!-- module: 3 | phase: 1 | last_verified: 2026-08-12 -->

# Module 3 — Installing Claude Code Everywhere

## 1. Installing on macOS, Windows, and Linux

### System requirements

- **OS:** macOS 13.0+ · Windows 10 1809+ or Server 2019+ · Ubuntu 20.04+ · Debian 10+ · Alpine Linux 3.19+
- **Hardware:** 4 GB+ RAM, x64 or ARM64
- **Shell:** Bash, Zsh, PowerShell, or CMD
- **Network:** internet connection required

### Install

The native installer is the recommended path:

```bash
# macOS, Linux, WSL
curl -fsSL https://claude.ai/install.sh | bash
```

```powershell
# Windows PowerShell
irm https://claude.ai/install.ps1 | iex
```

```batch
:: Windows CMD
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

Your prompt tells you which Windows shell you're in: `PS C:\` is PowerShell, `C:\` without the `PS` is CMD.

Other install methods: `brew install --cask claude-code`, `winget install Anthropic.ClaudeCode`, signed apt/dnf/apk repositories, and `npm install -g @anthropic-ai/claude-code` (requires Node.js 22+, though the installed binary doesn't use Node at runtime).

**The difference that matters: native installs auto-update in the background. Homebrew, WinGet, and Linux package-manager installs do not.** With a tool that ships weekly, an install method that never updates means silently drifting months behind. If you install via Homebrew or WinGet, either run the upgrade yourself or set `CLAUDE_CODE_PACKAGE_MANAGER_AUTO_UPDATE=1`.

### Verify

```bash
claude --version   # prints e.g. 2.1.211 (Claude Code)
claude doctor      # read-only install and settings diagnostics, without starting a session
```

### Choose a release channel

```json
{ "autoUpdatesChannel": "stable" }
```

- `latest` (default) — new features as soon as they ship
- `stable` — roughly a week behind, skipping releases with major regressions

Set it via `/config` → **Auto-update channel**. On Homebrew the channel is the cask name instead: `claude-code` is stable, `claude-code@latest` is latest.

### Windows: native vs WSL

| Option | Requires | Sandboxing | Use when |
|---|---|---|---|
| Native Windows | Nothing (Git for Windows optional) | **Not supported** | Windows-native projects and tools |
| WSL 2 | WSL 2 enabled | **Supported** | Linux toolchains, or sandboxed command execution |
| WSL 1 | WSL 1 enabled | Not supported | WSL 2 unavailable |

**Sandboxing works on WSL 2 and not on native Windows.** If you plan to let Claude run infrastructure commands, that's an architectural decision, not a preference — make it before you install.

Git for Windows is optional. Without it, Claude Code uses the PowerShell tool instead of Bash. If Git Bash is installed but not found:

```json
{ "env": { "CLAUDE_CODE_GIT_BASH_PATH": "C:\\Program Files\\Git\\bin\\bash.exe" } }
```

---

## 2. Full platform tour

| Platform | Best for | What you get |
|---|---|---|
| **CLI** | Terminal workflows, scripting, remote servers | Full feature set. Agent SDK and scripting are CLI-only |
| **Desktop** | Visual review, parallel sessions, managed setup | Diff viewer, app preview, computer use, Dispatch |
| **VS Code** | Working without switching to a terminal | Inline diffs, integrated terminal, file context |
| **JetBrains** | IntelliJ, PyCharm, WebStorm, and others | Diff viewer, selection sharing, terminal session |
| **Web** | Long tasks that don't need much steering | Cloud sessions that keep running after you disconnect |
| **Mobile** | Starting and monitoring while away | Cloud sessions, Remote Control, Dispatch to Desktop |

Two things worth knowing early:

- **You can mix surfaces on the same project.** Configuration, project memory, and MCP servers are shared across the local surfaces. You don't have to pick one.
- **The CLI is the most complete surface.** Scripting and the Agent SDK are CLI-only. Desktop and the IDE extensions trade some CLI features for visual review and tighter editor integration.

### Working away from your terminal

These are easy to confuse, so here's what actually differs — what triggers the work, and where Claude runs:

| | Trigger | Claude runs on |
|---|---|---|
| **Dispatch** | Message a task from the Claude mobile app | Your machine (Desktop) |
| **Remote Control** | Drive a running session from claude.ai/code or mobile | Your machine (CLI or VS Code) |
| **Channels** | Events pushed from a chat app or your own server | Your machine (CLI) |
| **Web / cloud sessions** | Start a session on the web | Anthropic cloud |
| **Self-hosted environments** | Start a cloud session, pick your org's environment | Your organization's infrastructure |
| **Scheduled tasks / routines** | A schedule | CLI, Desktop, or cloud |

---

## 3. Authentication and account setup

**Claude Code requires a Pro, Max, Team, Enterprise, or Console account. The free Claude.ai plan does not include Claude Code access.**

Run `claude` and a browser opens for login. Two things that come up constantly:

- **Browser doesn't open?** Press `c` to copy the login URL and paste it in yourself.
- **Browser shows a code instead of redirecting back?** Paste it at the `Paste code here if prompted` prompt. This happens whenever the browser can't reach Claude Code's local callback server — common in **WSL2, SSH sessions, and containers**.

Use `/logout` to sign out and `/status` to see which credential is currently active.

**The most common auth confusion:** if `ANTHROPIC_API_KEY` is set in your environment, it takes precedence over your subscription once you approve it. A stale key from an old experiment produces auth failures that look like a broken install. Run `unset ANTHROPIC_API_KEY` and check `/status`.

### Account types

- **Pro or Max** — log in with your Claude.ai account
- **Claude for Teams** — self-service plan with collaboration features, admin tools, and billing management
- **Claude for Enterprise** — adds SSO, domain capture, role-based permissions, compliance API, and managed policy settings
- **Claude Console** — API-based billing. Admins invite users and assign either the **Claude Code** role (can only create Claude Code API keys) or **Developer** (any key type)

### Tokens for CI

Where there's no browser, generate a one-year OAuth token:

```bash
claude setup-token
export CLAUDE_CODE_OAUTH_TOKEN=your-token
```

The token prints once and isn't saved anywhere — copy it immediately. It can only make model requests, so it can't establish Remote Control sessions.

---

## 4. Your first Claude Code session

Open a project, run `claude`, and build this habit from day one:

**Ask Claude to explain the codebase before asking for any change.**

Ask the questions you'd ask a senior engineer on the team:

- How does logging work?
- How do I make a new API endpoint?
- What edge cases does `CustomerOnboardingFlowImpl` handle?
- Why does this code call `foo()` instead of `bar()` on line 333?

No special prompting is needed. This is one of the highest-return uses of Claude Code — it cuts ramp-up time on an unfamiliar codebase and takes load off your teammates. It's also a safe first session, because nothing is being changed.

---

## 5. Running Claude Code on other providers

If your organization can't send code to Anthropic's API directly, Claude Code runs on infrastructure you or your cloud provider control:

| Route | What it is |
|---|---|
| **Amazon Bedrock** | Claude Code against Bedrock-hosted models, with AWS IAM and billing |
| **Google Cloud's Agent Platform** | The same on Google Cloud |
| **Microsoft Foundry** | The same on Azure |
| **Claude apps gateway** | A self-hosted gateway that signs developers in with your IdP and routes inference to whichever provider you configure |
| **Other LLM gateways** | Generic gateway/proxy support via `ANTHROPIC_AUTH_TOKEN` |

Set the provider's environment variables before running `claude`, or pick **3rd-party platform** at the login prompt — that launches an interactive setup wizard for Bedrock and Google Cloud. No browser login is needed.

```bash
export CLAUDE_CODE_USE_BEDROCK=1   # or CLAUDE_CODE_USE_VERTEX / CLAUDE_CODE_USE_FOUNDRY
```

Cloud provider credentials sit at the top of the authentication precedence order, so once one of these variables is set it wins over any API key or subscription login on the machine.

Third-party providers work in the **CLI** and **VS Code**. Enterprise Desktop deployments support Google Cloud's Agent Platform and gateway providers; for Bedrock or Foundry, use the CLI or VS Code.

> **On running Claude Code against a local model (Ollama and similar):** this isn't a supported configuration and isn't covered in Anthropic's documentation. Community setups exist that point Claude Code at an OpenAI-compatible local endpoint, but they break in ways that are hard to debug, and local model quality is low enough that many features taught in this course won't behave as described. If your goal is keeping inference inside your own boundary, use a gateway or a cloud provider from the table above instead.

---

## 6. Interface deep dive

### Mode and model controls

| Shortcut | Does |
|---|---|
| `Shift+Tab` | Cycle permission modes: `default` (shown as **Manual**) → `acceptEdits` → `plan`, plus `auto` / `bypassPermissions` if enabled |
| `Option/Alt+P` | Switch model without clearing your prompt |
| `Option/Alt+T` | Toggle extended thinking (no effect on Fable 5, which always thinks) |
| `Option/Alt+O` | Toggle fast mode |

### Session control

| Shortcut | Does |
|---|---|
| `Esc` | Interrupt Claude mid-turn — work done so far is kept |
| `Esc` `Esc` | Clear the input draft, or open the rewind menu when input is empty |
| `Ctrl+O` | Toggle transcript viewer — tool usage, timestamps, model per message |
| `Ctrl+B` | Background running tasks |
| `Ctrl+T` | Toggle Claude's to-do checklist |
| `Ctrl+R` | Reverse-search prompt history |
| `Ctrl+G` | Open your prompt in `$EDITOR` |
| `Ctrl+L` | Redraw a garbled terminal |
| `Ctrl+C` | Interrupt, or clear input; twice to exit |

### Input

- `/` — commands and skills · `!` — shell mode · `@` — file path autocomplete
- `?` on an empty prompt — toggle the shortcut help panel
- `Shift+Enter` for multiline (native in iTerm2, WezTerm, Ghostty, Kitty, Warp, Apple Terminal, Windows Terminal)
- Prompt history is stored **per working directory** and persists across sessions, so `Up` recalls prompts from previous sessions in the same project

### Vim mode

Enable via `/config` → **Editor mode**, or in settings:

```json
{
  "editorMode": "vim",
  "vimInsertModeRemaps": { "jj": "<Esc>" }
}
```

`Esc` enters NORMAL mode; `i` / `I` return to INSERT. In NORMAL mode, when the cursor can't move further, `j`/`k` navigate command history instead.

### Appearance

`/theme` for the theme picker (`Ctrl+T` inside the picker toggles syntax highlighting in code blocks), plus a customizable status line and `/help`.

---
<!-- nav -->

[← Prompt Engineering for Agentic Work](module-02-prompt-engineering.md) · [All modules](../README.md#the-course) · [The Agentic Loop and Built-In Tools →](module-04-agentic-loop-tools.md)
