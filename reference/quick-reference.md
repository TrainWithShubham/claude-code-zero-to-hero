# Claude Code: Zero To Hero
### Master Cheatsheet — All 16 Modules

A quick-reference companion for the full course. Each topic gets a bit more than a bare cheatsheet — enough to actually understand the "why," not just the command. Facts here are checked against Anthropic's official Claude Code docs as of August 2026; Claude Code ships weekly, so version-specific details (exact numbers, flag names) are worth a quick re-check before you rely on them for something critical.

---

## PHASE 1: ZERO → COMFORTABLE

# Module 1 — Welcome to the Era of Agentic Coding

**1. What is Generative AI and an LLM**
- An LLM is a model trained on huge amounts of text to predict/generate language. "Generative" means it creates new content, not just classifies existing content.
- Claude is Anthropic's family of LLMs, tuned for helpfulness, safety, and reasoning.
- An LLM by itself just talks. Give it tools, and it becomes an *agent* — that's where Claude Code comes in.

**2. Claude's Model Lineup in 2026: Haiku 4.5, Sonnet 5, Opus 5, and the Mythos Tier**
- **Haiku 4.5** — fastest and cheapest, good for lightweight/high-volume tasks.
- **Sonnet 5** — the default model for Pro, Team Standard, and Enterprise seats; native 1M-token context, adaptive thinking on by default.
- **Opus 5** — the current top model for the hardest tasks; also a 1M-token context window.
- **Mythos tier** — sits above Opus. Claude Mythos 5 and Claude Fable 5 share the same underlying model; Fable adds extra safety measures for biology, cybersecurity, and LLM R&D.
- Rule of thumb: pick a model by task difficulty and budget, not by "biggest is always best" — Haiku for volume, Sonnet for daily driving, Opus/Mythos for the genuinely hard stuff.

**3. Claude.ai vs Claude API vs Claude Code vs Claude Cowork vs Agent SDK**
- **Claude.ai** — the chat interface for humans.
- **Claude API / Claude Platform** — programmatic access for building your own product.
- **Claude Code** — the agentic coding tool: CLI, IDE extensions, Desktop, Web, Mobile.
- **Claude Cowork** — agentic knowledge-work app for non-developers (builds folders, slide decks, spreadsheets).
- **Agent SDK** — the library version of Claude Code's own agent loop, for building your own agents/products (Module 14).
- **Claude Tag** — a Slack-based interface, tag @Claude in any channel.

**4. What Makes an AI System "Agentic": The Agentic Loop Explained**
- Agentic = a loop of **gather context → take action → verify results**, repeating until done — not just "has tools."
- Each tool result feeds the next decision. Contrast with a fixed workflow, which is a deterministic script with no judgment in the loop (you'll meet the scripted version as Dynamic Workflows in Module 10).

**5. Where Claude Code Fits — DevOps, Cloud, SRE, and Developers**
- It's not just "build me a website" — think debugging a Kubernetes rollout, reviewing a Terraform plan, automating a CI/CD pipeline, or triaging an incident.
- It sits alongside your existing terminal, IDE, CI system, and cloud console — it doesn't replace them.

**6. Course Roadmap and the Two Capstone Tracks**
- 16 modules, 4 phases: Zero→Comfortable, Comfortable→Productive, Productive→Advanced, Advanced→Hero.
- Everyone takes Modules 1–14; Module 15 splits into **Track A** (general dev) and **Track B** (DevOps/SRE) for the capstone.

---

# Module 2 — Prompt Engineering for Agentic Work

**1. Anatomy of a Good Prompt for an Agent**
- State the outcome and constraints, not the implementation steps — the loop figures out *how*.
- Constraints matter more here than for a chatbot: "don't touch the tests folder," "don't push to main."
- Agent prompts usually need *more* detail than chatbot prompts, not less. Automatic context-gathering means Claude can find things; it doesn't tell Claude which things matter or what "done" looks like.

**2. System Prompts, CLAUDE.md, and Output Styles — Where Instructions Live**
- **System prompt** — Claude Code's own built-in instructions (fixed, not yours to edit directly).
- **CLAUDE.md** — your persistent instructions layer, loaded every session (full detail in Module 5).
- **Output styles** — modify the system prompt itself to change role, tone, or default format on every turn. Selected with `/config` → Output style.

**3. Zero-Shot vs Few-Shot vs Chain-of-Thought**
- Zero-shot: just ask — works well since the loop explores on its own.
- Few-shot: show example output format — useful for commit messages, PR descriptions, structured reports.
- Chain-of-thought: ask it to reason before acting — current models already do adaptive/extended thinking by default, so this matters most for genuinely ambiguous asks.

**4. Common Mistakes Beginners Make**
- Over-specifying implementation steps (removes the agent's room to adapt).
- Under-specifying success criteria (no way for it to verify the work).
- One giant prompt instead of iterative steering.
- Forgetting to state hard constraints up front.

**5. Hands-On: Writing Your First Effective Prompts**
- **Try:** `Add a health-check endpoint to this API. It should return 200 and a JSON status.`
- Compare that against a version that dictates literal code line by line — notice which one Claude handles more gracefully when your repo doesn't match your assumptions exactly.

---

# Module 3 — Installing Claude Code Everywhere

**1. Installing on macOS, Windows, and Linux**
- Ships as native binaries; Windows can run without Git Bash (falls back to PowerShell).
- `npm install -g @anthropic-ai/claude-code` also still works as an install path if you prefer it.

**2. Full Platform Tour**
- **CLI** — terminal-first, most control.
- **VS Code / JetBrains** — inline diffs, @-mentions, in-editor plan review.
- **Desktop app** — parallel sessions, computer use, iOS Simulator pane, in-app browser.
- **Web** — cloud sessions, no local setup, connect a GitHub repo directly.
- **Mobile** — monitor/steer tasks from your phone, push notifications.
- Sessions are increasingly portable across all of these via Remote Control and `--cloud`/`--teleport`.

**3. Authentication and Account Setup**
- Individual (subscription or API key), Team, Enterprise (SSO, managed settings).
- If a browser OAuth callback can't reach localhost, you can paste the code manually instead.

**4. Your First Claude Code Session**
- **Try:** open a project and ask Claude to explain the codebase structure *before* asking for any change — good habit to build from day one.

**5. Running Claude Code on Other Providers**
- Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry, the self-hosted Claude apps gateway, or a generic LLM gateway — for organizations that can't send code to Anthropic's API directly.
- Set `CLAUDE_CODE_USE_BEDROCK` / `CLAUDE_CODE_USE_VERTEX` / `CLAUDE_CODE_USE_FOUNDRY` before running `claude`, or pick **3rd-party platform** at the login prompt for an interactive setup wizard.
- Cloud provider credentials outrank every other credential on the machine, so setting one of those variables wins over any API key or subscription login.
- Running against a local model (Ollama and similar) is not a supported configuration and isn't in Anthropic's docs — use a gateway if the goal is keeping inference inside your own boundary.

**6. Interface Deep Dive**
- `/theme`, `/help`, Vim mode, custom status line.
- Shift+Tab cycles permission modes — you'll meet this properly in Module 5, but notice it exists now.

---

## PHASE 2: COMFORTABLE → PRODUCTIVE

# Module 4 — The Agentic Loop and Built-In Tools
*(Full detail already delivered as its own pack — condensed here for the master reference.)*

**1. How Claude Code Works** — the loop is gather context → take action → verify results; every tool result shapes the next step; you can interrupt/redirect anytime.

**2. Read, Write, Edit** — Read = look, Edit = targeted change (usually reads first), Write = full overwrite. Low friction inside your working directory.

**3. Grep and Glob** — Grep = search by content, Glob = find by filename pattern. Both skip permission prompts inside your project.

**4. Bash Tool + Sandboxing** — Bash covers what dedicated tools don't (installs, tests, git). Sandbox mode (Seatbelt on macOS, bubblewrap on Linux/WSL2) isolates filesystem and network separately, so commands can run without a prompt every time. `BASH_MAX_OUTPUT_LENGTH`: 30,000 chars default, 150,000 ceiling.

**5. Context Window + Prompt Caching** — Every message is a fresh request; caching reuses a matching prefix. Model switches and CLAUDE.md edits both break the cache. `/context` diagnoses, `/compact` treats.

**6. Tools Reference Recap** — Read, Write, Edit, Grep, Glob, Bash this module; Subagents (Module 10) and MCP tools (Module 9) still to come.

---

# Module 5 — Permissions, Memory, and Configuration

**1. Choosing a Permission Mode**
- Modes: **default** (asks every time), **acceptEdits** (edits auto-approved, everything else still asks), **plan** (read-only research), **auto**, **dontAsk** (settings-only, silently denies anything not pre-approved), **bypassPermissions** (skips essentially every prompt — isolated environments only).
- Shift+Tab cycles default → acceptEdits → plan; auto and bypass join the cycle conditionally. The status bar always shows the active mode.

**2. Auto Mode Deep Dive**
- A separate safety-classifier model vets shell commands and risky actions — pushing to main, deploying, deleting pre-existing files, sending data externally — while routine reads/edits in your working directory run immediately, no prompt.
- Becoming the default permission mode for new Pro/Max/Team sessions from August 14, 2026.
- Respects boundaries stated in conversation — tell it "don't push," and it won't, even in auto mode.

**3. CLAUDE.md Explained**
- Load order, broadest to most specific — files are concatenated, not overridden, so the later one wins: **managed policy** → **user** (`~/.claude/CLAUDE.md`) → **project** (`./CLAUDE.md` or `./.claude/CLAUDE.md`) → **local** (`./CLAUDE.local.md`, gitignored). Nested directory-level files load on demand when Claude reads a file in that directory.
- `.claude/rules/` splits instructions into topic files, and a `paths:` frontmatter field scopes a rule so it only loads when Claude touches matching files — the fix for a CLAUDE.md that's grown too big.
- Keep the root file short and specific — under ~200 lines is a good target; push detail into separate files Claude can follow.
- It's read fresh every session as *context*, not enforced config — use a hook if you need something to actually be blocked, not just discouraged.

**4. Auto Memory**
- On by default. Claude writes its own notes — build commands, debugging insights, style preferences — to `~/.claude/projects/<project>/memory/`.
- `MEMORY.md` is the always-loaded index (capped around 200 lines); detailed notes spill into topic files.
- Toggle from `/memory`; every memory file is plain markdown you can read, edit, or delete.

**5. Exploring the .claude Directory and settings.json**
- `.claude/` (project) and `~/.claude/` (user) hold CLAUDE.md, settings.json, hooks, skills, commands, and subagents.
- Settings precedence, highest to lowest: **managed** (can't be overridden) → **CLI arguments** → **local** (`.claude/settings.local.json`) → **project** (`.claude/settings.json`) → **user** (`~/.claude/settings.json`). Managed outranks CLI flags, and local outranks project — both are easy to get backwards.
- Permission rules are the exception: they **merge** across scopes instead of overriding.

**6. Debugging Your Config**
- Four go-to diagnostics: `/context` (what's loaded), `/doctor` (full setup checkup, alias `/checkup`), `/hooks` (what's registered), `/mcp` (what's connected).

---

# Module 6 — Planning and Executing a Real Project

**1. Green Field vs Brown Field Projects**
- Green field: new project, Claude scaffolds from nothing.
- Brown field: existing codebase — Claude has to explore and infer conventions first, which is exactly where a good CLAUDE.md pays for itself.

**2. Plan Mode vs Direct Execution vs Auto Mode**
- Plan mode: Claude researches and writes a plan file without touching source.
- Approving a plan offers: start in auto mode, accept edits, review manually, keep planning, or refine with Ultraplan (browser-based review).
- **Ctrl+G** opens the proposed plan in your text editor so you can hand-edit it *before* approving — the single most useful plan-mode habit to teach.

**3. Session Management**
- `/resume` picker, `--continue`, `--resume`, `--from-pr`.
- `/branch [name]` copies the conversation so far and **switches you into the copy**, leaving the original intact and still in the picker. From the CLI it's a flag: `claude --continue --fork-session`. (There is no `/fork` command.)
- Name sessions with `claude -n <name>` or `/rename` — an AI-generated title is not a resume handle, only a name you set is.
- `/rewind` can resume from before a `/clear`.

**4. Working Across a Monorepo or Large Codebase**
- Nested CLAUDE.md files per package, `worktree.sparsePaths`, `Read` deny rules for generated code, code intelligence plugins, and per-package skills keep Claude focused on the slice you're working in.
- **Where you start Claude decides the rest.** From the repo root you get every file but only the root CLAUDE.md at launch; from a subdirectory you get that subtree plus that directory's and every ancestor's CLAUDE.md.
- CLAUDE.md is inherited from parent directories; **`.claude/settings.json` is not** — project settings load only from your starting directory.

**5. Common Workflows**
- The docs' "Common workflows" page has copy-paste recipes for bug fixes, refactors, and test generation — a good first stop before writing a prompt from scratch.

**6. Hands-On: Planning and Building a Starter Project**
- **Try:** `/plan` a small feature → review with Ctrl+G → approve into acceptEdits → build it.

---

# Module 7 — Extending Claude Code: Skills, Commands, and Hooks

**1. Introduction to Agent Skills (SKILL.md)**
- Skills and custom slash commands are unified — a `SKILL.md`/`.md` file in `.claude/skills/` (recommended) or the older `.claude/commands/` (still works) becomes a `/command-name`.
- Skills can **auto-invoke** based on their description, not only when typed — Claude decides when one's relevant.
- Skills follow the open [Agent Skills](https://agentskills.io) standard, so they're portable — but **only some frontmatter fields are in the standard**. Outside Claude Code only `name`, `description`, `license`, `compatibility`, `metadata`, and `allowed-tools` are accepted; extensions like `argument-hint` or `context: fork` are rejected.
- Skill directories are watched live — adding or editing a `SKILL.md` takes effect mid-session, no restart.

**2. Creating Custom Slash Commands**
- A markdown file with instructions plus a `$ARGUMENTS` placeholder for dynamic input is the simplest form.
- Built-ins worth knowing for reference: `/clear`, `/compact`, `/context`, `/hooks`, `/doctor`.

**3. Automating Actions with Hooks**
- Hooks are **deterministic** — they run every time, no exceptions — unlike skills, which are probabilistic (Claude's judgment call).
- **`PreToolUse` hooks fire before any permission-mode check, in every mode.** A hook returning `permissionDecision: "deny"` blocks the tool even under `bypassPermissions` or `--dangerously-skip-permissions` — the only genuinely unbypassable layer in Claude Code.
- Common events: `PreToolUse` (can block), `PostToolUse` / `PostToolUseFailure`, `UserPromptSubmit`, `Stop` / `StopFailure`, `SessionStart` / `SessionEnd`, `PreCompact` / `PostCompact`, `InstructionsLoaded`, `PermissionRequest`, `SubagentStart` / `SubagentStop`, `ConfigChange`. Most take matchers (tool name, session-start reason, compaction trigger).
- Exit codes: **0** proceeds, **2** blocks with stderr as the reason, anything else is a non-blocking error. Hooks on the same event run in parallel and **deny wins**. A `Stop` hook is overridden after 8 consecutive blocks — check `stop_hook_active` in its input.
- Handler types: command (shell), http, mcp_tool, prompt, agent.
- Classic use: a `PostToolUse` hook on Write/Edit that auto-runs your formatter/linter.

**4. Output Styles**
- Same agent loop, different response "personality" — for adapting Claude Code to non-engineering uses.
- Built-ins: **Default**, **Proactive** (acts immediately, prefers action over planning), **Explanatory** (adds teaching "Insights"), **Learning** (leaves `TODO(human)` markers for you to fill in). Select with `/config` → Output style.
- Custom styles are markdown files in `.claude/output-styles/`. Set `keep-coding-instructions: true` in the frontmatter, or the style strips Claude Code's built-in software-engineering instructions.
- Read once at session start — a change needs `/clear` or a new session to take effect.

**5. Hands-On: Building Your Own Skill**
- **Try:** build a `/security-review` skill that checks for hardcoded secrets, SQL injection risk, and unsanitized input.

---

# Module 8 — The Plugin Ecosystem

**1. What Are Plugins and Why They Matter**
- A plugin bundles skills, subagents, hooks, and MCP servers into one installable unit — the packaging layer above individual extensions.

**2. Discovering and Installing Plugins from Marketplaces**
- The official marketplace `claude-plugins-official` is **added automatically** the first time you start Claude Code interactively — nothing to set up.
- `/plugin marketplace add <repo>`, `/plugin install <name>@<marketplace>`, `/plugin list`. The `/plugin` panel has four tabs: Discover, Installed, Marketplaces, Errors.
- Install scopes: **user** (all your projects), **project** (everyone on the repo, via `.claude/settings.json`), **local** (you, this repo), **managed** (admin-installed, locked). If the summary says `Run /reload-plugins to activate.`, run it.
- Plugins can also load from `.zip` archives or a direct URL.
- Official marketplaces auto-update; **third-party and local ones don't**. Removing a marketplace uninstalls every plugin from it.

**3. Creating and Distributing Your Own Plugin Marketplace**
- You can host your own marketplace to distribute plugins across a team or a community — relevant if you want to ship a "Zero To Hero starter kit" plugin to your own students.

**4. Claude Security Plugin — Scanning Your Codebase for Vulnerabilities**
- Multi-agent vulnerability scanner (public beta from July 21, 2026): maps architecture, threat-models, and suggests patches — you review and apply the findings, it doesn't silently auto-fix.

**5. security-guidance Plugin — Catching Issues as Claude Writes Code**
- Runs automatically once installed, nothing to invoke manually.
- Three review layers: a zero-cost per-edit pattern check, an end-of-turn LLM diff review, and an agentic cross-file review at commit time.
- Non-blocking and advisory. The pattern check runs **after** an edit lands (it's a `PostToolUse` hook) and appends a warning to Claude's context for the next step — it never blocks the write.
- Built entirely on hooks — `SessionStart`, `UserPromptSubmit`, `PostToolUse` on Edit/Write, `Stop`, and `PostToolUse` on Bash filtered to git commit/push. A working example of everything in Module 7.
- Extend it with `.claude/claude-security-guidance.md` (threat model in prose) and `.claude/security-patterns.yaml` (regex rules). Disable layers with `ENABLE_PATTERN_RULES=0`, `ENABLE_STOP_REVIEW=0`, `ENABLE_COMMIT_REVIEW=0`.
- Install: `/plugin install security-guidance@claude-plugins-official`. Available on all plans.

**6. Hands-On: Installing and Configuring a Plugin Stack**
- **Try:** install security-guidance, deliberately write a hardcoded API key, and watch Claude flag and fix it before the turn ends.

---

# Module 9 — MCP (Model Context Protocol) in 2026

**1. What Is MCP and Why It Became the Industry Standard**
- The open standard for connecting agents to external tools and services. Hosts (the app) → clients (connectors) → servers (the services); servers expose **resources**, **prompts**, and **tools**.
- The signal you need one: you're copy-pasting into chat from another tool — an issue tracker, a dashboard, a database.

**2. MCP Architecture: The 2026-07-28 Spec**
- Moved to a **stateless core** (request/response instead of bidirectional/stateful), so servers can run on serverless/edge infrastructure.
- Three opt-in extensions negotiated at initialization: **Tasks** (async long-running ops with polling and durable handles), **MCP Apps** (inline interactive UI), and **Skills over MCP** (agent workflows discovered over MCP).
- Remote servers commonly use OAuth 2.0; Claude Code runs the flow via `/mcp` or `claude mcp login <name>`, with automatic discovery from a `WWW-Authenticate` header.
- The spec treats tool descriptions from untrusted servers as untrusted input — relevant again in Module 12.

**3. Connecting Your First MCP Server**
- `claude mcp add --transport http|sse <name> <url>`, or `--transport stdio <name> -- <command>` — everything after `--` is passed to the server untouched.
- Scopes: **local** (default, `~/.claude.json`, this project, private), **project** (`.mcp.json`, checked in, shared), **user** (all your projects). MCP "local scope" lives in `~/.claude.json`, *not* `.claude/settings.local.json`.
- `claude mcp login` authenticates a configured server from your shell; `claude mcp logout` clears its credentials. Needed for `-p` and SDK runs, where there's no `/mcp` panel to run OAuth in.
- `/mcp` opens the interactive menu for setup and inspection. `claude mcp list` shows per-server health: `✔ Connected`, `! Needs authentication`, `✘ Failed to connect`, `⏸ Pending approval`.
- **Tool search is on by default** — only tool names and server instructions load at startup, so there's no per-server tool cap and adding servers barely costs context. `ENABLE_TOOL_SEARCH=auto` for threshold-based loading instead.

**4. Real-World MCP Example: AWS and Observability Tools**
- Wire up an MCP server for AWS or a monitoring/incident tool so Claude can query real infrastructure state instead of guessing from code alone.

**5. Channels: Pushing Alerts and Webhooks into a Running Session**
- A channel is an MCP server that **pushes events in** rather than waiting to be queried — CI results, chat messages, monitoring alerts — so Claude reacts while you're away. Events only arrive while the session is open.
- **Research preview.** Anthropic auth only (no Bedrock/Google Cloud/Foundry), each plugin needs Bun, and Team/Enterprise orgs must enable `channelsEnabled` first.
- Telegram, Discord, iMessage, plus **fakechat** — a localhost demo with nothing to authenticate, the right one to try first.
- Two independent gates: a per-channel **sender allowlist** (bootstrapped with a pairing code), and the server must be named in `claude --channels plugin:<name>@<marketplace>`. Being in `.mcp.json` is not enough.

**6. Hands-On: Wiring Up an MCP Server**
- **Try:** connect a GitHub or Slack MCP server and ask Claude to list open PRs/issues through it.

---

## PHASE 3: PRODUCTIVE → ADVANCED

# Module 10 — Subagents and Orchestration at Scale

**1. Creating Custom Subagents**
- Focused workers with their own context, tools, and instructions; they report results back to the caller only — they don't talk to each other.
- Good for keeping your main context clean (e.g., a dedicated "doc-fetcher" subagent).

**2. Background Subagents and Agent View**
- Subagents run in the background by default. `claude agents` opens **agent view** — one screen showing every session: what's running, what's blocked on you, what's done.
- Background sessions survive closing the terminal and are resumable via `claude --resume`; each one consumes your subscription quota independently, so watch how many you dispatch at once.

**3. Agent Teams**
- Independent Claude Code sessions that **share a task list and message each other directly** through a mailbox — teammates, not subordinates.
- Experimental and disabled by default. Good for PR review, competing debugging hypotheses, migration planning.
- Doesn't isolate teammates in worktrees by itself — partition work by file ownership so they don't collide.

**4. Dynamic Workflows**
- A script (that Claude writes) holds the orchestration logic instead of a lead agent deciding turn by turn — deterministic control flow for repeatable audits or large migrations.
- Use this when you want the *process* itself, not just the work, to be repeatable.
- Requires v2.1.154+ on any paid plan (on Pro, enable it in `/config`). `/deep-research` ships built in; trigger your own with the `ultracode` keyword, plain language ("use a workflow"), or `/effort ultracode`.
- Watch with `/workflows` — `p` pauses, `x` stops, **`s` saves the run's script** as a reusable `/command`.
- Limits: **16 concurrent agents, 1,000 per run**, no mid-run user input, no filesystem access from the script itself. Default size guideline is `medium` (under 15 agents).
- **Resume replays in start order:** caching stops at the first agent that didn't finish, and everything started after it reruns even if it completed. Fan out across many small agents rather than a few long ones.

**5. Worktrees**
- Give each parallel session its own git checkout so edits never collide.
- A subagent with `isolation: worktree` gets a temporary worktree branched from your **default branch**, not the parent session's `HEAD`, and it's removed automatically if the subagent changed nothing.
- On large repos pair this with `worktree.sparsePaths` (check out only what's needed) and `worktree.symlinkDirectories` (share one `node_modules`).

**6. Cross-Session Messaging**
- `ListAgents`/`SendMessage` let independent sessions — yours, on this machine, on another machine, or on the web — hand off findings directly.
- A message is just text with a reply address — not your full context. If you need the whole conversation moved, resume/fork the session instead of messaging it.

**7. Hands-On: Multi-Agent Codebase Audit**
- **Try:** dispatch three subagents in parallel on the same diff — one for security, one for tests, one for performance — and compare their findings.

---

# Module 11 — Claude Code Across Every Surface

**1. Claude Code Desktop**
- Parallel sessions with git isolation, drag-and-drop pane layout, integrated terminal/file editor, visual diff review.
- **Computer use** (research preview) lets Claude click through native apps on macOS for things only a GUI can verify.
- **iOS Simulator pane** — opens automatically when Claude builds/runs/checks an app; drives the simulator directly (no Accessibility/Screen Recording permissions needed), local sessions only, macOS with Xcode.
- **In-app browser** — Claude can pull up docs, designs, or any site and interact with it the same way it does local dev-server previews.
- **What Desktop can't do:** agent teams are CLI-only (dynamic workflows *do* run in Desktop); no inline code suggestions; Bedrock and Foundry need the CLI or VS Code; computer use isn't on Linux yet; `/permissions` returns `isn't available in this environment`, and `/config` opens Settings while ignoring any argument.

**2. Claude Code on the Web and Routines**
- Cloud sessions, connect a GitHub repo, no local setup required.
- **Routines** — templated cloud agents that fire on a schedule, a GitHub event, or an API call: real autopilot.

**3. Claude Code on Mobile**
- Start, monitor, and steer tasks from your phone; push notifications when a long task finishes or Claude needs input.

**4. Remote Control**
- Continue a local session from your phone, tablet, or any browser via claude.ai/code or the mobile app.

**5. Claude Tag (Slack)**
- Tag @Claude in any channel to delegate a task — the current path for Team/Enterprise Slack workspaces (Pro/Max still use the earlier Claude Code in Slack setup).

**6. Claude in Chrome**
- Generally available on all direct Anthropic plans — test web apps, debug via console logs, automate form filling, extract data straight from the browser.
- Start with `claude --chrome`; `/chrome` shows status and lets you reconnect or pick a browser. Needs extension v1.0.36+.
- **Requires `/login`** — an API key or `claude setup-token` token keeps Chrome integration off even with `--chrome`. Not available on Bedrock/Google Cloud/Foundry, and **not supported in WSL**.
- Claude **shares your browser's login state**, so it works inside anything you're already signed into — and pauses for you at login pages and CAPTCHAs.
- In plan mode, read-only browser calls run without a prompt; clicks, typing, navigation, and GIF recording ask first. Enabling Chrome by default raises context usage since browser tools always load.

---

# Module 12 — CI/CD, Code Review, and Security

**1. Claude Code GitHub Actions**
- `@claude` mentions in PR/issue comments trigger the action, which auto-detects interactive vs. automation mode. Built on the Claude Agent SDK — same engine, packaged for CI.

**2. Claude Code in GitLab CI/CD**
- Same underlying idea, wired into GitLab pipelines for teams not on GitHub.

**3. Automated Code Review and /ultrareview**
- `/code-review` runs locally as a background subagent. **Effort levels trade coverage against confidence** — `low`/`medium` report only high-confidence findings, `high`→`max` broaden coverage and may include uncertain ones. With no level typed, it reuses the last one you typed, even from an earlier session.
- `/code-review ultra` escalates to **ultrareview**, a genuinely separate deeper review that runs in the cloud (`claude ultrareview` is its non-interactive subcommand). It needs a claude.ai account and isn't available on Bedrock/Google Cloud/Foundry or under Zero Data Retention — where it quietly falls back to a local review.
- Naming history: **`/review` is now an alias of `/code-review`**; `/simplify` was its old name and is now a separate cleanup-only review.
- Flags: `--fix` applies findings, `--comment` posts them inline on a PR. A background review's `--fix` edits land **outside checkpoints**, so `/rewind` won't undo them — use git.
- The managed **Code Review** GitHub App posts findings as inline PR comments automatically — multiple agents analyze the diff in parallel, then a verification step filters out false positives.
- Research preview, **Team and Enterprise only**, not available under Zero Data Retention. Roughly **$15–25 per review**, billed via usage credits rather than plan usage. Severity markers: 🔴 Important, 🟡 Nit, 🟣 Pre-existing. The check run always completes **neutral**, so it never blocks a merge.
- Tune it with two files that carry very different weight: **`CLAUDE.md`** is read as project context and its violations are flagged as *nits*; **`REVIEW.md`** is injected into every review agent's system prompt as the **highest-priority** block. `REVIEW.md` is pasted verbatim, so `@` imports aren't expanded — and the local `/code-review` command doesn't read it at all.
- `@claude review` triggers one review; **`@claude review always`** also subscribes the PR to push-triggered reviews. Before a July 2026 change the bare command did both.

**4. Security Fundamentals: Sandbox Environments**
- Escalating isolation levels: sandboxed Bash tool (Module 4) → dev containers/Docker → full VMs — pick based on how untrusted the code or task actually is.

**5. Managing Costs**
- Main levers: token usage, model selection, effort levels, and the prompt-caching habits from Module 4.
- `/usage` breaks down what's driving your plan limits by skill, subagent, plugin, and MCP server.

**6. Hands-On: Wiring Claude Code Into a CI/CD Pipeline**
- **Try:** add a GitHub Actions workflow that runs `/code-review` automatically on every PR open/sync.

---

# Module 13 — Claude Code for Infrastructure, Cloud, and Enterprise
*(Watch this module regardless of which capstone track you're heading toward.)*

**1. Claude Code for Infrastructure as Code: Terraform, Kubernetes, Docker**
- Same gather-act-verify loop applies to infra code as to app code — treat `terraform plan` output or `kubectl diff` as the "test" Claude verifies its own changes against.

**2. Deploying on Amazon Bedrock, Google Cloud's Agent Platform, and Microsoft Foundry**
- Three enterprise deployment routes, each with its own auth/IAM setup — useful when organizational policy requires everything to stay inside one cloud provider.

**3. Self-Hosted Environments**
- Run Claude Code cloud sessions on infrastructure your organization operates (public beta, Team/Enterprise).
- Why orgs adopt it: reach internal-only services/databases without exposing them publicly, pre-install internal compilers/SDKs/CLIs, and keep source code and build artifacts on infrastructure you control.
- **Self-hosting moves session *execution*, not inference.** Model calls still go to `api.anthropic.com` and cannot be routed through Bedrock, Google Cloud, Foundry, or an LLM gateway — prompts, responses, and tool results leave your network. Say this plainly in any compliance conversation.
- All connectivity is **outbound only**; Anthropic never connects into your network. Off by default (an Owner enables it, and Claude Code on the web must be on), unavailable under Zero Data Retention, and Claude Tag / Claude Security / Code Review sessions don't route there yet.
- **A runner serves one user at a time**, locking to the first account it claims — so minimum fleet size is the number of *concurrently active users*, not sessions.

**4. Claude Apps Gateway**
- The product is the **Claude apps gateway** — a self-hosted service between your developers' clients and your model provider. Centralized credentials, per-IdP-group model access, spend limits, SSO sign-in, OTLP telemetry.
- It ships **inside the `claude` binary** — the same executable runs the gateway server: `claude gateway --config gateway.yaml`.
- It's for organizations that must route inference through their own cloud provider (data residency). Without that requirement, Claude Enterprise is often the better fit.

**5. Admin Setup**
- Server-delivered managed settings without needing device-management infrastructure; MCP server allowlists/denylists; policy enforcement (e.g., blocking bypassPermissions mode org-wide).

**6. Observability**
- OpenTelemetry export for traces, metrics, and events; an analytics dashboard for tracking adoption and engineering velocity across a team.

---

## PHASE 4: ADVANCED → HERO

# Module 14 — Building Products with the Agent SDK

**1. Agent SDK Overview**
- Formerly the "Claude Code SDK," now the **Claude Agent SDK** — the same tools, agent loop, and context management as Claude Code, available as a Python/TypeScript library.
- Use it when you're building your *own* product or agent, not just using Claude Code interactively.
- **Python and TypeScript only.** From any other language, run the CLI as a subprocess with `-p --output-format json`.
- Shipping a product on it: **API key auth only** (claude.ai login and rate limits aren't allowed for third-party products), and you may say "Claude Agent" or "Powered by Claude" but **not "Claude Code"**.

**2. The Agent Loop: Message Lifecycle and Architecture**
- Same evaluate → call tools → receive results → repeat loop. A `SystemMessage` with subtype `"init"` starts a run; `"compact_boundary"` fires after compaction.
- `max_turns`/`max_budget_usd` cap the loop — a good default for any production agent so an open-ended prompt can't run away. `max_turns` counts **tool-use turns only**, and the budget cap **covers subagents** (spawning past it fails with `Budget limit reached`).
- Branch on `ResultMessage.subtype` — `success`, `error_max_turns`, `error_max_budget_usd`, `error_during_execution`, `error_max_structured_output_retries`. **The `result` field exists only on `success`.** A single-shot `query()` also *raises* after yielding an error result.

**3. Giving Claude Custom Tools and Structured Outputs**
- Define your own tools via an in-process MCP server so Claude can call your functions and hit your APIs directly.
- Get validated JSON back via JSON Schema, Zod, or Pydantic for anything downstream that needs type-safe data.

**4. Subagents, Skills, and Plugins Inside the SDK**
- Every Claude Code extension concept — subagents, skills, plugins, hooks — is available inside the SDK too. You're not choosing between "Claude Code features" and "the SDK"; you get both.

**5. Hosting the Agent SDK in Production**
- **`query()` spawns a separate `claude` CLI subprocess** that owns the shell, working directory, and JSONL transcripts on local disk. One session = one subprocess. Every hosting decision follows from this.
- Four session patterns: **ephemeral** (container per task), **long-running** (persistent, many sessions each), **hybrid** (ephemeral + a `SessionStore` to hydrate from — the store is required, not optional), and **multi-agent** (several subprocesses in one container).
- Sizing: **1 GiB RAM, 5 GiB disk, 1 CPU per agent** to start; `agents per host = (host RAM − overhead) / per-session RAM ceiling`. **Token cost dominates container cost by an order of magnitude** — a container is ~$0.05/hr.
- Multi-tenant isolation needs `setting_sources=[]` **and** `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1` — auto memory loads regardless of setting sources — plus a per-tenant `cwd` and `CLAUDE_CONFIG_DIR`.
- No top-level session timeout: bound every agent with `max_turns`.

**6. Tie-In: Pairing the Agent SDK with Durable Execution (Temporal)**
- The SDK handles the agent loop; Temporal handles durability, retries, and state across long-running, failure-prone workflows.
- Together they give you an agent that survives crashes and restarts — the underlying pattern behind KubeHealer.

---

# Module 15 — Capstone: Choose Your Track

**1. Capstone Kickoff — Planning with Plan Mode + Auto Mode**
- Start with plan mode, force a review with Ctrl+G, then switch into auto mode/acceptEdits for the actual build — same discipline as Module 6, now on a bigger project.

**2. Track A (General Dev): Full-Stack SaaS Feature**
- Ship one real feature end-to-end — API, UI, and tests — in an existing or starter repo.

**3. Track B (DevOps/SRE): Self-Healing CI/CD or Incident-Response Agent**
- Build an agent that watches a pipeline or alert feed (via channels/MCP) and takes a bounded remediation action behind a human-approval gate.

**4. Build Sprint**
- Track-specific and guided — use subagents, hooks, and skills from earlier modules rather than defaulting to raw prompting for everything.

**5. Capstone Review with /code-review and /ultrareview**
- Run both on your own project before calling it finished — hold yourself to the same bar taught in Module 12.

**6. Presenting and Documenting Your Project with Artifacts**
- Publish the session output as a live, shareable Artifacts page on claude.ai instead of just a static README — a stronger portfolio piece.

---

# Module 16 — Staying Sharp: What's Next

**1. Reading the Weekly Changelog**
- `code.claude.com/docs/en/whats-new` for the curated weekly digest; the full changelog for bug-fix-level detail. Build the habit of skimming it regularly — this course is a snapshot, Claude Code isn't.

**2. Claude's Expanding Surfaces — Where This Is Headed**
- Recap how fast the surface area has grown this year: CLI → IDE → Desktop → Web → Mobile → Slack → Chrome → Agent SDK → Managed Agents. Treat it as a trend, not a finish line.

**3. Building Your Own Learning Loop and Community**
- Watch plugin marketplaces, follow official release announcements, engage with the community building on the same tool.

**4. Course Recap, Certification, and Next Steps**
- Recap the full 16-module arc, and point back to whichever capstone track (A or B) you built as your portfolio piece going forward.

