<!-- module: 11 | phase: 3 | last_verified: 2026-08-12 -->

# Module 11 — Claude Code Across Every Surface

Same engine everywhere, tuned differently. This module is about picking the right one and knowing where each stops.

**You can mix surfaces on the same project.** Configuration, project memory, and MCP servers are shared across the local surfaces. The CLI remains the most complete one — scripting, the Agent SDK, and agent teams are CLI-only.

---

## 1. Your IDE — VS Code and JetBrains

For most developers this is the surface they'll actually live in. Both integrations connect to the same CLI you already installed — neither replaces it.

### VS Code

Install the **Claude Code extension** from the marketplace. You need VS Code 1.94.0 or higher and a paid Claude subscription or Console account; no API key. Anthropic calls this the recommended way to use Claude Code in VS Code.

What the extension gives you over a terminal:

- **Review and edit plans before accepting them** — plan mode becomes an editable document rather than a wall of text
- **Inline diffs** in the editor, with auto-accept as an option
- **@-mentions with line ranges** — select code and Claude sees it automatically. `Option+K` (Mac) / `Alt+K` (Windows/Linux) inserts the reference into your prompt as `@app.ts#5-10`
- **Multiple conversations** in separate tabs or windows, plus browsable history
- **Focus view** — hides tool calls, tool results, and thinking behind expandable rows, leaving your prompts and Claude's answers. `Ctrl+Option+F` / `Ctrl+Alt+F`, or **Claude Code: Toggle Focus view** in the Command Palette. It persists across sessions

Two shortcuts worth memorising:

| Shortcut | Does |
|---|---|
| `Cmd+Esc` / `Ctrl+Esc` | Toggle focus between the editor and Claude's prompt box |
| `Option+K` / `Alt+K` | Insert an @-mention for the current selection |

The prompt box footer shows how many lines are selected. Click the indicator to toggle whether Claude can see them — the eye-slash icon means hidden.

### JetBrains

One plugin covers **IntelliJ IDEA, PyCharm, Android Studio, WebStorm, PhpStorm, and GoLand**.

The important install detail: **the plugin does not bundle the CLI.** It runs `claude` in your IDE's integrated terminal and connects to it. Install the CLI first, then the [Claude Code plugin](https://plugins.jetbrains.com/plugin/27310-claude-code-beta-) from the JetBrains Marketplace, then restart the IDE completely. A "Cannot launch Claude Code" notification means `claude` isn't on your PATH — set the full path under **Settings → Tools → Claude Code [Beta] → Claude command**.

What you get:

- **Quick launch** — `Cmd+Esc` / `Ctrl+Esc` opens Claude Code from the editor
- **Diffs in the IDE's native diff viewer** instead of the terminal. Change it with **Diff tool** in `/config` (`auto` for the IDE, `terminal` to keep them inline)
- **Selection and open-tab context** shared automatically
- **File reference shortcut** — `Cmd+Option+K` / `Alt+Ctrl+K` inserts `@src/auth.ts#L1-99`
- **Diagnostic sharing** — lint and syntax errors from the IDE flow to Claude as you work

Run `claude` from the IDE's integrated terminal and everything is active. From an external terminal, run `/ide` to connect — it confirms with something like `Connected to IntelliJ IDEA.` If it finds a running IDE without the plugin, `/ide` installs it and asks you to restart.

### The `ide` MCP server

Both integrations work by running a local MCP server named `ide` that the CLI connects to. It's hidden from `/mcp` because there's nothing to configure — but it's worth knowing it exists if your organisation allowlists MCP tools with a `PreToolUse` hook.

Only one of its tools is visible to the model: `mcp__ide__getDiagnostics`, which returns the IDE's errors and warnings. The rest is internal RPC the CLI uses to open diffs and read selections. The JetBrains plugin exposes no code-execution tool to the model.

### Two things to be careful about

**Your selection travels with your prompt.** While connected, the CLI attaches the current editor selection and the active file's path to each message — the transcript shows `⧉ Selected N lines from <file>`. To keep a sensitive file such as `.env` out, add a **`Read` deny rule** for its path. A matching deny rule blocks both the selected text and the open-file notice.

**`acceptEdits` is riskier inside JetBrains.** Claude may be able to modify IDE configuration files that the IDE then executes automatically — which sidesteps the bash permission prompts. Anthropic's own guidance is to prefer manual approval for edits in JetBrains.

---

## 2. Claude Code Desktop

The Claude Desktop app has three tabs: **Chat**, **Cowork**, and **Code**. This module is about the Code tab.

### What you get

- **Parallel sessions with git isolation** — each session in its own worktree, arranged as drag-and-drop panes
- **Integrated terminal and file editor**, plus opening files in other apps
- **Visual diff review** before accepting changes
- **App preview** for local dev servers, and **PR status monitoring**
- **Side chats** — ask a question without derailing the session
- **Computer use** (research preview) — Claude clicks through native macOS apps for things only a GUI can verify
- **iOS Simulator pane** — opens automatically when Claude builds or runs an app, drives the simulator directly with no Accessibility or Screen Recording permissions needed. Local sessions only, macOS with Xcode
- **In-app browser** — Claude pulls up docs, designs, or any site and interacts with it
- **Dispatch** — message a task from the Claude mobile app and it spawns a Desktop session on your machine

### What's *not* in Desktop

This is the part that catches people moving between surfaces:

| Not available | Detail |
|---|---|
| **Agent teams** | **CLI only.** Dynamic workflows *do* run in Desktop, so Module 10 splits across surfaces |
| **Third-party providers** | Desktop connects to Anthropic's API by default. Gateway routing is supported, and Enterprise deployments can configure Google Cloud via managed settings. For Amazon Bedrock or Microsoft Foundry, use the CLI or VS Code |
| **Computer use on Linux** | Not yet available in the Linux desktop app (beta) |
| **Inline code suggestions** | No autocomplete-style completions — Desktop works through conversational prompts |
| **Terminal-dialog commands** | `/permissions` replies `isn't available in this environment`. `/config` opens Settings → Claude Code and **ignores any argument**, so `/config theme=dark` does nothing. Edit settings files directly, or run the command from the standalone CLI |

---

## 3. Claude Code on the web, and Routines

**Cloud sessions.** Connect a GitHub repo, no local setup, and the work keeps running after you disconnect. Best for long-running tasks that don't need much steering.

**Routines** are saved Claude Code configurations — a prompt, one or more repositories, and a set of connectors — that run automatically on Anthropic-managed infrastructure. Create them at `claude.ai/code/routines`, from the Desktop app, or with `/schedule` in the CLI.

Three trigger types, combinable on one routine:

| Trigger | Fires on |
|---|---|
| **Schedule** | Hourly, daily, weekdays, or weekly — minimum interval is one hour. Also one-off runs |
| **API** | A POST to a per-routine endpoint with a bearer token |
| **GitHub** | Repository events — pull requests and releases, with filters |

```text
/schedule daily PR review at 9am
/schedule in 2 weeks, open a cleanup PR that removes the feature flag
```

Three things worth knowing before you rely on them:

- **Routines belong to an individual claude.ai account**, not a team. Anything a routine does through your connected GitHub identity or connectors appears as **you** — commits and PRs carry your user.
- **Runs are autonomous.** There's no permission-mode picker and no approval prompts. What a routine can reach is set by its repositories, its environment's network access, and the connectors you include. Scope each to what it actually needs.
- **A green status means the session started and exited without an infrastructure error — not that the task succeeded.** Open the run and read the transcript.

---

## 4. Claude Code on mobile

Start, monitor, and steer tasks from your phone, with push notifications when a long task finishes or Claude needs input.

Mobile is a thin client. It reaches cloud sessions, drives a local session through Remote Control, and can send a task to Desktop with Dispatch.

---

## 5. Remote Control

Continue a running local session from your phone, tablet, or any browser via claude.ai/code or the mobile app. Start it with `claude remote-control`.

### Telling the away-from-terminal options apart

These four blur together. The distinction is what triggers the work and where Claude actually runs:

| | Trigger | Claude runs on |
|---|---|---|
| **Dispatch** | Message a task from the mobile app | **Your machine** (Desktop) |
| **Remote Control** | Drive a running session from claude.ai or mobile | **Your machine** (CLI or VS Code) |
| **Web / cloud sessions** | Start a session on the web | Anthropic cloud |
| **Routines** | A schedule, API call, or GitHub event | Anthropic cloud |
| **Channels** (Module 9) | An event pushed from Telegram, Discord, or a webhook | **Your machine** (CLI) |

---

## 6. Claude Tag — bringing Claude into Slack

Tag `@Claude` in any channel to assign it a task from a thread.

**The plan split matters:**

- **Claude Tag** is available on **Team and Enterprise** plans. It runs `@Claude` as **your organization's shared identity with admin-configured access**.
- **Claude Code in Slack** is the earlier integration, which runs each session under **an individual user's account**. On **Pro and Max**, where Claude Tag isn't available, this remains the setup path.

The difference is access control, not features — which is exactly why it matters to an enterprise audience. A shared org identity with admin-configured access is auditable in a way that per-user sessions are not.

---

## 7. Claude in Chrome

Connects Claude Code to the **Claude in Chrome** browser extension, giving browser automation from the CLI or the VS Code extension.

```bash
claude --chrome     # or run /chrome and pick "Enabled by default"
/chrome             # status, permissions, reconnect, choose a browser
```

### Requirements that will block people

- **Claude in Chrome extension v1.0.36 or later**
- **A direct Anthropic plan** — Pro, Max, Team, or Enterprise
- **You must be signed in with `/login`.** An API key or a long-lived `claude setup-token` token keeps Chrome integration **off**, even if you pass `--chrome`, because the extension can't authenticate with those credentials
- **Not available on Amazon Bedrock, Google Cloud, or Microsoft Foundry**
- **Not supported in WSL**

Works with Chrome, Edge, and other Chromium browsers — Brave, Arc, Vivaldi, Opera.

### What it can do

- **Live debugging** — read console errors and DOM state, then fix the code that caused them
- **Design verification** — build a UI from a mock, open it, and compare
- **Web app testing** — form validation, visual regressions, user flows
- **Authenticated web apps** — Claude **shares your browser's login state**, so it can work in Gmail, Google Docs, Notion, or your CRM with no API connector
- **Data extraction**, **file uploads** (10 MB total per upload; hard-linked files are refused), and **GIF recording**

Claude opens new tabs for browser work, and browser actions run in a visible window in real time. At a login page or CAPTCHA it pauses and asks you to take over.

### Two cautions worth teaching

- **Enabling Chrome by default increases context usage**, because browser tools always load. Use `--chrome` per session if you notice it.
- **A recorded GIF captures everything visible, including account details on logged-in pages.** Review before sharing.

### Plan mode behaves sensibly here

Read-only browser calls run without a prompt — `read_page`, `get_page_text`, `find`, console and network reads, screenshots. State-changing calls prompt: clicks, typing, navigation, tab management, GIF recording.

### The demo worth doing

```text
I just updated the login form validation. Open localhost:3000, submit the
form with invalid data, and check whether the error messages appear correctly.
```

Then follow it with a console read and a fix. That loop — change, verify in a real browser, fix — is the whole value of the integration in one exercise.

---
<!-- nav -->

[← Subagents and Orchestration at Scale](module-10-subagents-orchestration.md) · [All modules](../README.md#the-course) · [CI/CD, Code Review, and Security →](module-12-cicd-review-security.md)
