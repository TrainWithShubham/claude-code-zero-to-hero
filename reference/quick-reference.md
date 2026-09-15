# Claude Code: Zero To Hero
### Master Cheatsheet — All 15 Modules

A quick-reference companion for the full course. Each topic gets a bit more than a bare cheatsheet — enough to actually understand the "why," not just the command. Facts here are checked against Anthropic's official Claude Code docs as of September 2026; Claude Code ships weekly, so version-specific details (exact numbers, flag names) are worth a quick re-check before you rely on them for something critical.

---

## PHASE 1: ZERO → COMFORTABLE

# Module 1 — Understanding Claude and How It Works

**1. From AI to Generative and Agentic AI**
- Rule-based AI → machine learning (learns patterns from data) → generative AI (LLMs generate new content, one token at a time) → agentic AI (an LLM plus the ability to act: call tools, observe results, decide what's next, in a loop).
- A chatbot generates a response and stops. An agent generates, executes, checks whether it worked, and adjusts. Claude Code is an agent in this sense.

**2. Claude's Model Lineup**
- **Haiku 4.5** — fastest, 200K context, best for lightweight/high-volume tasks.
- **Sonnet 5** — default balance of speed and intelligence, 1M-token context.
- **Opus 5** — complex agentic coding and enterprise work, 1M-token context.
- **Fable 5.1** — demanding reasoning / long-horizon agentic tasks, slower, 1M-token context.
- All current models take text + image input, produce text, support tool use. "Adaptive thinking" replaces manually toggled extended thinking — the model decides how much internal reasoning to spend, steered by an effort setting.
- Model IDs are pinned and change on a published schedule — look them up at build time, don't memorize.

**3. The Product Ecosystem**
- **claude.ai** — consumer chat interface (web + mobile).
- **Claude Developer Platform (API)** — programmatic integration; the foundation everything else is built on.
- **Claude Code** — this course's focus; same engine across Terminal (CLI), IDE extensions (VS Code, JetBrains), Desktop app, and Web (claude.ai/code).
- **Claude Agent SDK** — build your own agents on the same tools/orchestration (Module 14).
- Also integrates with CI/CD (GitHub Actions, GitLab), Slack, and Chrome.

---

# Module 2 — Prompt Engineering Foundations

**1. Prompts and Tokens**
- A prompt is everything sent to the model: your instruction plus system instructions, file contents, and conversation history.
- A token is roughly a word-chunk, not a whole word. Code, punctuation, and non-English text tokenize less efficiently than plain English.
- Context window = max tokens per request (prompt + files + history + output share the budget). Current Claude models: up to 1M tokens (Sonnet 5, Opus 5, Fable 5.1), 200K (Haiku 4.5). A bigger window isn't permission to fill it — irrelevant content still costs money and can distract the model.

**2. Structuring and Iterating**
- Give clear, direct instructions with format, scope, and constraints stated explicitly.
- Provide context ("why"), not just commands.
- Use few-shot examples for consistent formatting (commit messages, JSON shapes).
- Use structure (XML tags or markdown sections) to separate context, instructions, and examples in longer prompts.
- Break large tasks into smaller checkable steps.
- Treat prompting as a loop — iterate on the actual output rather than expecting a perfect first prompt.
- Name a role/perspective when it helps ("review this as a senior backend engineer focused on security").

**3. Getting Concise Output, Avoiding Mistakes**
- Ask directly for length/format ("two sentences," "only code, no explanation," "no markdown headers").
- Avoid vague asks ("make this better") when you want a specific outcome — state the outcome.
- Don't paste more than the task needs; don't repeat full context every turn if history already has it.
- Watch for self-contradictory asks ("be extremely thorough" + "under 50 words").
- A wasted round trip costs more than a slightly longer prompt — a little extra up-front detail is usually worth it.

---

# Module 3 — Understanding Agentic AI Systems

**1. How LLMs, Tools and Memory Make an Agent**
- The LLM is the reasoning engine — it can only generate text describing an intended action; it never executes anything itself.
- **Tool use (function calling)** turns text into action: the model picks a tool and arguments, the harness actually executes it, the result feeds back as new input.
- **Memory** gives the loop continuity: ordinary context-window history, plus durable layers like CLAUDE.md (written by you) and auto memory (written by Claude, persists across sessions).
- The **perceive → act → observe → decide** loop is what makes something "agentic," not tool access alone — a single request/response is just a chatbot.
- Claude Code is a concrete instance: Claude reasons, built-in tools (read/edit files, run bash, search) act, CLAUDE.md + auto memory persist context, and the loop is what you watch run — read, edit, test, adjust — with check-ins governed by your permission mode.
- "Agentic" is a spectrum: one tool call before replying is thinly agentic; chaining dozens of tool calls and coordinating subagents (introduced in Module 9) is agentic in a much deeper sense.

---

# Module 4 — Getting Started with Claude Code

**1. What Claude Code Is and Where to Use It**
- Terminal-native agentic coding tool; also available as IDE extensions (VS Code, JetBrains), a Desktop app (with a Cowork tab), Web (claude.ai/code), and CI/CD (GitHub Actions, GitLab).
- Install: macOS/Linux/WSL — `curl -fsSL https://claude.ai/install.sh | bash`; Windows PowerShell — `irm https://claude.ai/install.ps1 | iex`; also via Homebrew or WinGet. Verify with `claude --version`.
- Native installer updates itself automatically; **Homebrew casks don't** — run `brew upgrade claude-code` manually.
- On native Windows without Git for Windows, Claude Code falls back to PowerShell as its shell tool instead of Bash.

**2. Logging In and Choosing a Model**
- Login options: Claude subscription (Pro/Max/Team/Enterprise — recommended), Claude Console (API credits, auto-creates a "Claude Code" workspace), or an enterprise cloud provider (Bedrock, Google Cloud's Agent Platform, Microsoft Foundry).
- `ANTHROPIC_API_KEY` set in the environment skips the login prompt entirely and asks you to approve the key instead.
- Default model by account: Opus for Max/Enterprise, Sonnet for Pro. Switch with `/model` (interactive picker) or `--model` at startup. In the picker: **Enter** saves as your default for future sessions, **`s`** switches for this session only.

**3. Local Models and Ollama**
- Ollama (a separate, third-party tool) documents its own integration at `docs.ollama.com/integrations/claude-code` — it runs a local server that mimics the Anthropic API shape Claude Code already speaks.
- Quick path: `ollama launch claude`. Manual path: set `ANTHROPIC_AUTH_TOKEN=ollama`, `ANTHROPIC_API_KEY=""`, `ANTHROPIC_BASE_URL=http://localhost:11434`, then run `claude --model <name>`.
- Free once a model is downloaded (no per-token billing, nothing leaves your machine), but expect reduced feature support (web search, some tool controls) versus a hosted session.

---

# Module 5 — Claude Code Setup, Configuration and Memory

**1. Settings Files and CLAUDE.md**
- Settings precedence, **highest to lowest**: managed settings (`managed-settings.json`/MDM/console) → command line (`claude --settings`) → project local (`.claude/settings.local.json`, gitignored) → shared project (`.claude/settings.json`, committed) → user (`~/.claude/settings.json`).
- `settings.json` holds `permissions.allow` / `permissions.deny` / `permissions.ask` rules and `permissions.defaultMode`.
- CLAUDE.md is instructions, not enforcement — it shapes behavior as context; only `permissions.deny` actually blocks something regardless of Claude's judgment.
- `/context` shows what settings/memory files actually loaded; `/init` generates a starting CLAUDE.md from your codebase.

**2. Permission Modes**
- Six modes: **default/Manual** (reads only), **acceptEdits** (reads + edits + common filesystem commands like `mkdir`/`touch`/`mv`/`cp`/`sed`/`rm`/`rmdir`), **plan** (reads, plus classifier-approved commands when auto is available), **auto** (everything, background safety classifier reviews risky actions), **dontAsk** (reads + pre-approved only, denies the rest), **bypassPermissions** (everything, no checks — isolated environments only).
- `auto` is the built-in starting mode for terminal/VS Code sessions on Pro, Max, and Team; Enterprise and Console API keys start in Manual.
- `Shift+Tab` cycles modes in the CLI; the mode indicator does it in VS Code/Desktop.

**3. CLAUDE.md and Auto Memory**
- CLAUDE.md — you write it (standards, build commands, conventions). Locations: `./CLAUDE.md` or `./.claude/CLAUDE.md` (team, committed), `~/.claude/CLAUDE.md` (personal, all projects), `CLAUDE.local.md` (personal, gitignored, project-specific).
- Auto memory — Claude writes it automatically from your corrections and session learnings. Stored per-project at `~/.claude/projects/<project>/memory/`, indexed by `MEMORY.md`, on by default. Machine-local, not shared with teammates.
- Keep CLAUDE.md under ~200 lines; push per-area detail into `.claude/rules/`. `/memory` browses, edits, or toggles auto memory.

---

## PHASE 2: COMFORTABLE → PRODUCTIVE

# Module 6 — Built-in Tools and Working with Codebases

**1. Core Tools**
- **Read** — reads a file's contents. **Write** — creates or fully overwrites a file. **Edit** — targeted exact-string-replacement change to an existing file (Claude's default for bug fixes). **Bash** — shell commands (tests, installs, git, anything without a dedicated tool). **Grep** — searches file contents. **Glob** — finds files by name pattern.
- Claude doesn't pre-load your whole repo — it searches (Grep/Glob) before reading and editing, same as a human would. Giving specific paths/function names in your prompt skips a search step.
- In Manual mode, Read/Grep/Glob run freely (read-only); Write/Edit/most Bash prompt. In `acceptEdits`, Write/Edit plus a specific list of filesystem Bash commands run without prompting — everything else in Bash still asks.

**2. Reviewing and Fixing Bugs**
- Scope a review request ("review the new API handlers for missing input validation") rather than a bare "review my code."
- Bug-fix loop: describe the concrete symptom → Claude searches (Grep/Glob) → reads context (Read) → fixes (Edit) → runs tests (Bash) if present.
- Git operations work conversationally: "what files have I changed?", "commit with a descriptive message," "show me the last 5 commits."
- Always read the diff before trusting a fix, especially in `acceptEdits`/`auto` mode.

---

# Module 7 — Planning a Complete Project With Claude Code

**1. Defining Requirements, Features, Tech Stack**
- Plan mode (`Shift+Tab` or `claude --permission-mode plan`): Claude can read files and search the web, but cannot edit or run changing commands — it must present a plan first.
- Ask for three distinct outputs: requirements (constraints the project must satisfy), features (what a user experiences), and a tech stack — plus the reasoning behind stack choices, not just the choices.
- Small projects benefit from plan mode too — an ungoverned agent overbuilds (adds a database, auth, a build pipeline nobody asked for) just as readily on a small project as a large one.

**2. Reviewing and Refining the Plan**
- A first plan is a draft — review it for gaps between what you asked for and what it silently assumed (data storage, external services).
- Ask for targeted revisions to one section rather than a full regeneration.
- Approving the plan is what lets Claude exit plan mode and start editing — treat that as a deliberate moment.

---

# Module 8 — Executing and Building the Project

**1. Starting the Build and Tracking Progress**
- After approving a plan: Manual mode reviews everything, `acceptEdits` auto-approves edits + common filesystem commands, `auto` hands routine review to a background classifier. Switch with `Shift+Tab` or start with `--permission-mode`.
- A markdown task file (e.g. `TASKS.md`) is a working pattern, not a built-in feature — ask Claude to maintain one as a resumable checkpoint across a multi-turn build. Spot-check "done" items against actual code.

**2. Running, Testing and Debugging**
- Claude can run your dev server, build, and tests directly via Bash — no need to switch to a separate terminal.
- Ask for tests on the pieces most likely to break silently, close to when each piece is built (not all at the end).
- When something fails: ask Claude to explain the failure (what was expected vs. what happened) before asking for a fix.
- Paste the real error/stack trace — never a paraphrase of the symptom.

---

# Module 9 — Slash Commands, Skills and Sub-Agents

**1. Skills**
- An Agent Skill is a markdown file with YAML frontmatter. The `description` field loads into context every turn (cheap); the full skill body loads only when invoked (the expensive part), which is the point of skills as a mechanism.
- Sources: bundled with Claude Code (`/code-review`, `/doctor`), your own project skills (`.claude/skills/<name>/SKILL.md`) or personal skills (`~/.claude/skills/<name>/SKILL.md`), or installed from a marketplace via `/plugin` (e.g. `anthropics/claude-plugins-official`).
- Invoke by typing `/skill-name`, or Claude invokes it automatically when your request matches the description — unless `disable-model-invocation: true` restricts it to manual use (use this for anything with side effects, like a deploy skill).

**2. Slash Commands and Intro to Sub-Agents**
- Slash commands span setup (`/init`, `/permissions`), in-session (`/plan`, `/effort`), review (`/diff`, `/code-review`), and session management (`/clear`, `/resume`). A custom slash command **is** a skill — same frontmatter, intended for manual invocation.
- Sub-agents run a task in an isolated context window and report back only a summary — for work whose intermediate detail (logs, search noise) you don't need to keep.
- Defined as markdown + YAML frontmatter in `.claude/agents/` (project) or `~/.claude/agents/` (personal): `name`, `description` (drives auto-delegation), optional `tools` and `model`.
- Built-in sub-agents: **Explore** (fast read-only search), **Plan** (context-gathering in plan mode), **General-purpose** (broader multi-step work).

---

# Module 10 — MCP Foundations

**1. MCP Clients, Servers and External Tools**
- MCP (Model Context Protocol) is the open standard connecting an agent to external systems without bespoke integration code per tool. Claude Code is the **client**; an **MCP server** exposes tools (and sometimes resources/prompts) that Claude can call like built-in tools.
- Transports: **stdio** (local process Claude Code spawns), **HTTP** (remote, often OAuth), **SSE** (Server-Sent Events, remote).
- Scopes: **local** (default, this machine/project only), **project** (`.mcp.json`, committed, team-shared), **user** (all your projects).
- Never assume a server is trustworthy — it runs with whatever real access it's configured with.

**2. Finding and Installing an MCP Server**
- Add: `claude mcp add --transport http <name> <url>` (optionally `--header "Authorization: Bearer <token>"`), `claude mcp add <name> -- <command> [args...]` for stdio, or `claude mcp add-json <name> '<json>'` for other transports. `--scope project|user` (default: local).
- Manage: `claude mcp list`, `claude mcp get <name>`, `claude mcp remove <name>`, `claude mcp login/logout <name>`. `/mcp` shows live status inside a session.
- Find servers via Anthropic's connector directory (claude.ai/directory), vendor docs, or vetted community GitHub repos — check maintenance activity before trusting one with credentials.

**3. Worked Examples**
- **AWS Documentation** — AWS maintains an official, open-source MCP server family in `awslabs/mcp` (including a broader "AWS Knowledge" server) — reasonably trustworthy for read access to public docs; check the repo for current server names/commands.
- **Hacker News** — community-maintained servers exist, but **no single official one**; the public HN API needs no auth so the risk surface is small, but still verify the specific project's maintenance before use.

---

## PHASE 3: PRODUCTIVE → ADVANCED

# Module 11 — Advanced Automation with Sub-Agents and Plugins

**1. Hooks**
- Fire on lifecycle events: `PreToolUse` (before a tool call — can block it), `PostToolUse` (after success), `SessionStart`/`SessionEnd`, `UserPromptSubmit`, `Stop`, plus more specialized events (subagents, file changes, worktrees, model switches).
- Configured as JSON: a `matcher` (which tool/condition) plus handlers with a `type` — `command` (local script), `http`, `mcp_tool`, `prompt`, or `agent`.
- Live in `.claude/settings.json` (shared), `.claude/settings.local.json` (personal), `~/.claude/settings.json` (all your projects), or a plugin's `hooks/hooks.json`.
- A command hook gets structured JSON on stdin; exit code **2** reliably blocks the action, or return JSON with a reason.

**2. Plugins**
- Bundle skills, agents, hooks, and MCP servers into one shareable unit via a `.claude-plugin/plugin.json` manifest at the plugin root — everything else (`skills/`, `commands/`, `agents/`, `hooks/hooks.json`, `.mcp.json`, `.lsp.json`) sits at the plugin root too, **never** nested inside `.claude-plugin/`.
- Plugin skills are namespaced: a `hello` skill in plugin `my-toolkit` becomes `/my-toolkit:hello`.
- Test locally with `claude --plugin-dir ./my-plugin`; `/reload-plugins` picks up edits without restarting.
- Distributed via marketplaces: official `claude-plugins-official` (auto-registered on first interactive run), community `claude-community`, or a private team marketplace.
- A hook shipped inside a plugin travels with it and updates for everyone who installs it — unlike one stuck in personal `settings.local.json`.

**3. Specialised Background Agents**
- Standing automation, not one-off delegation: a testing agent, verification agent, or release-notes agent that always runs the same check.
- A plugin's root `settings.json` can set a default `agent` for the whole session. A `monitors/monitors.json` in a plugin runs a long-lived command (tailing logs, polling a status endpoint) and streams output to Claude as notifications.

---

# Module 12 — Advanced Project: URL Shortener

**1. Architecture and Setup**
- Three pieces: **API service** (`POST /api/v1/shorten`, `GET /api/v1/urls/{short_code}` — validates input including an SSRF check on submitted URLs), **redirect service** (`GET /{short_code}`, cache-aside with Redis, atomic click-count updates to avoid lost counts under concurrency), and **storage** (Postgres + Redis, schema migrations via Alembic).
- Setup: copy the spec's `CLAUDE.md` and `.claude/agents/*.md` (from `labs/README.md` / `labs/CLAUDE.md` in this repo) into your own new repo before starting Claude Code there.
- Build in slices: scaffold → schema → shorten endpoint → redirect endpoint → stats → operational pieces (rate limiting, CORS, health checks) → frontend → cleanup script. Review each slice before moving on.

**2. Assignment Workflow and Submission**
- Required agent sequence per slice: **testing agent** (writes/runs tests) → **verification agent** (checks implementation against spec, pass/fail) → **push agent** (commits + opens a PR, refuses to run until told the other two passed; never merges).
- Submit: repo/PR link, a one-line note if you built a Path B variant, and a written list of known limitations.
- README should cover what the service does, how to run it locally (docker compose commands, env vars, migrations), and what's out of scope.
- Stretch goals: a CI pipeline, and hook-enforced sub-agent permissions (structurally guaranteeing the push agent can't skip the check).

---

# Module 13 — Prompting Strategies and Multi-Agent Architecture

**1. Defining Agent Roles**
- Subagents: markdown + YAML frontmatter in `.claude/agents/` (project) or `~/.claude/agents/` (personal) — `name`, `description`, optional `tools`, `model`. The markdown body fully replaces the default system prompt for that subagent.
- Restricting `tools` is a real permissions boundary, not documentation — e.g. a security auditor with only `Read`/`Grep` can flag but not "fix" anything.
- Only a subagent's final response returns to the calling session — its intermediate reasoning doesn't clutter your context.

**2. Persona Prompting and Parallel Sub-Agents**
- Persona prompting: write the role directly into the prompt/system prompt ("You are a security expert...") to narrow focus and shape output structure.
- Dispatch multiple subagents in one instruction to run them **in parallel**, each with its own isolated context window — the main session synthesizes results once all are back.
- Parallel subagents pay off on independent, read-heavy work (research, audits across unrelated areas); a poor fit when one subagent's output should inform another's starting point.

**3. Managing Context**
- `/compact` summarizes conversation history on demand and accepts custom instructions ("`/compact` Focus on code samples and API usage"); reports `Not enough messages to compact.` on a short session.
- Auto-compact triggers the same summarization automatically once context usage crosses a configurable threshold.
- `/clear` discards history entirely with no summary — use it between unrelated tasks; use `/compact` when the task is ongoing and old context still matters.
- Compacting a large context is itself an expensive request — proactive habits (clear early, keep CLAUDE.md lean, offload verbose output to subagents) beat reacting to compaction warnings.

---

# Module 14 — CI/CD and Reliability

**1. GitHub Repository and Automated Checks**
- Official GitHub integration ships as a GitHub Action (`anthropics/claude-code-action`). `@claude` mentions in PR/issue comments trigger analysis, changes, and pushed commits.
- Fastest setup: `/install-github-app` (needs `gh` authenticated + repo admin access) — installs the GitHub App, sets up an auth secret (`ANTHROPIC_API_KEY` or `CLAUDE_CODE_OAUTH_TOKEN`), and opens a PR with the workflow file.
- **Automation mode** — give the workflow a fixed `prompt` input so it fires on any GitHub event (PR, push, schedule) without needing a mention; distinct from the separate, zero-setup **Code Review** feature that reviews every PR automatically.
- Never commit an API key into a workflow file — reference `${{ secrets.ANTHROPIC_API_KEY }}`. Commits made with the default `GITHUB_TOKEN` don't trigger further workflow runs.

**2. Production Database and Deployment**
- Claude Code has no documented Vercel-specific integration — treat "deploy to Vercel" as a general coding task: it can run the Vercel CLI (`vercel link`, `vercel env`, `vercel deploy`, `vercel --prod`) with your approval.
- Production database needs its own connection string as an environment variable (never committed), a migration step before the app depends on it, and separation from local/preview databases.

**3. CI/CD and Reliability Basics (SRE)**
- **CI** — every change is merged frequently and verified automatically (build, test, lint, review stages; failing any blocks the merge). **CD** — extends that pipeline to automatically ship to staging/production.
- **SLI** (Service Level Indicator) — a measured metric (e.g. % of requests returning within 300ms). **SLO** (Service Level Objective) — your internal target for an SLI (e.g. 99.9% within 300ms over 30 days). **SLA** (Service Level Agreement) — an external, often contractual commitment, looser than the SLO.
- The gap between SLO and the ceiling your SLIs could hit is your **error budget** — the unreliability you're allowed to spend on deploys/experiments/risk before threatening the SLA.
- CI/CD protects the SLO: automated tests/reviews catch regressions before they ship; automated deploys reduce manual error; a fast rollback path matters as much as the forward path.

---

## PHASE 4: ADVANCED → HERO

# Module 15 — Beyond Claude Code: Careers, Tools and What's Next

- This module is the course's close, not a separate capstone — the URL shortener (Modules 12 and 14) is the portfolio piece.
- Turn the project into evidence: push the repo publicly, write a README explaining architecture and decisions, be ready to walk through what Claude Code did versus what you reviewed and changed.
- In interviews, talk about the workflow (scoping tasks, when you used plan mode, how you split work across subagents, how you caught wrong turns) — not just the output.
- The human role moves up a level, it doesn't disappear: you decide what "correct" looks like, review every diff, choose architecture, and set the guardrails (permissions, hooks, CLAUDE.md) — Claude Code executes inside lines you draw.
- The same patterns (persona prompting, parallel subagents, context management, CI/CD) generalize beyond web projects — to infrastructure automation, data pipelines, DevOps/SRE workflows.
- Build a habit of checking `code.claude.com/docs/en/whats-new` regularly — Claude Code ships weekly, and this course (like any course) is a snapshot, not a finished picture.
