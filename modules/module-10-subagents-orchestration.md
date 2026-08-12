<!-- module: 10 | phase: 3 | format: deep-dive | last_verified: 2026-08-12 -->

# Module 10 — Subagents and Orchestration at Scale

Everything so far has been one Claude working in one conversation. This module is about many of them.

## The organizing question: who holds the plan?

Four mechanisms can run a multi-step task. They differ in who decides what runs next.

| | Subagents | Skills | Agent teams | Workflows |
|---|---|---|---|---|
| **What it is** | A worker Claude spawns | Instructions Claude follows | A lead supervising peer sessions | A script the runtime executes |
| **Who decides what runs next** | Claude, turn by turn | Claude, following the prompt | The lead, turn by turn | **The script** |
| **Where intermediate results live** | Claude's context window | Claude's context window | A shared task list | **Script variables** |
| **What's repeatable** | The worker definition | The instructions | The team definition | **The orchestration itself** |
| **Scale** | A few per turn | Same as subagents | A handful of long-running peers | **Dozens to hundreds per run** |
| **Interruption** | Restarts the turn | Restarts the turn | Teammates keep running | **Resumable in the same session** |

Hold onto the middle row. With subagents, skills, and agent teams, **Claude is the orchestrator** and every intermediate result lands in a context window. A workflow moves the plan into code, so the script holds the loop and the branching, and Claude's context holds only the final answer.

---

## Chapter 1 — Creating custom subagents

**What you'll learn:** the worker primitive everything else builds on.

### The concept

A subagent is a specialized worker with its own context window, system prompt, tool access, and permissions. Claude delegates to it, it works independently, and it returns **only its summary**.

Reach for one when a side task would flood your main conversation with search results, logs, or file contents you'll never reference again. Define a *custom* one when you keep spawning the same kind of worker with the same instructions.

Subagents help you:

- **Preserve context** — exploration stays out of the main conversation
- **Enforce constraints** — limit which tools a worker can use
- **Reuse configurations** across projects
- **Control costs** — route work to a faster, cheaper model

### Where they live

| Location | Scope |
|---|---|
| `.claude/agents/` | This project — check into version control |
| `~/.claude/agents/` | All your projects |

> As of v2.1.198, `/agents` no longer opens an interactive creation wizard. Running it prints a reminder to ask Claude or edit `.claude/agents/` directly. The file locations and frontmatter are unchanged.

The easiest way to make one is to ask:

```
Create a personal code-improver subagent in ~/.claude/agents/ that scans for
dead code and unused exports. Give it read-only tools and run it on Haiku.
```

### Frontmatter

| Field | Required | Notes |
|---|---|---|
| `name` | Yes | Lowercase and hyphens. Can't contain `:` — that's reserved for plugin scoping |
| `description` | Yes | How Claude decides to delegate |
| `tools` | No | Inherits everything if omitted. **If no entry resolves to a real tool, the subagent fails to launch** |
| `model` | No | `sonnet`, `opus`, `haiku`, `fable`, a full ID, or `inherit`. **Defaults to `inherit`** |
| `skills` | No | Preloads full skill *content* into context, not just the description |
| `mcpServers` | No | Servers available to this subagent |
| `memory` | No | `user`, `project`, or `local` — gives the subagent its own persistent memory |
| `background` | No | Claude chooses when unset, and **runs subagents in the background by default** as of v2.1.198 |
| `effort` | No | Overrides the session effort level |
| `isolation` | No | `worktree` runs it in a temporary git worktree |

Two of these deserve attention.

**`model` is a cost lever.** A doc-fetcher or a file-lister doesn't need Opus. Routing routine subagents to Haiku is one of the cheapest wins available.

**`isolation: worktree`** gives the subagent an isolated copy of the repository, **branched from your default branch rather than the parent session's `HEAD`**. The worktree is automatically cleaned up if the subagent makes no changes. This is what lets several subagents edit files in parallel without colliding.

### Try it yourself

Ask Claude to create a read-only research subagent, then use it:

```
Use a subagent to investigate how our authentication system handles token
refresh, and whether we have existing OAuth utilities I should reuse.
```

Watch your main context stay clean while the subagent reads a dozen files.

### Common pitfalls

- **Listing tools that don't exist.** The subagent won't launch, and the error names the bad entries.
- **Using a subagent for something you could do in three tool calls.** Every delegation costs a round trip and a re-briefing.
- **Expecting subagents to talk to each other.** They report to the caller only. If they need to coordinate, you want agent teams.

### Key takeaways

- A subagent is a worker with its own context that returns only a summary
- `.claude/agents/` for the project, `~/.claude/agents/` for you
- `model: haiku` for routine work, `isolation: worktree` for parallel edits

---

## Chapter 2 — Background subagents and agent view

**What you'll learn:** how to keep track when several things are running at once.

### The concept

Subagents run in the background by default, so Claude keeps working while they do. That's efficient, and it creates a new problem: knowing what's running.

```bash
claude agents
```

Agent view is one screen showing every session: what's running, what's blocked on you, and what's done.

Background sessions **survive closing the terminal** and are resumable with `claude --resume`. Each consumes your subscription quota independently, so watch how many you dispatch at once.

### Try it yourself

Dispatch two or three research subagents, then open agent view in another terminal and watch them progress.

### Key takeaways

- Background is the default; `claude agents` is how you see everything
- Background sessions outlive the terminal and are resumable
- Each one draws on your quota independently

---

## Chapter 3 — Agent teams

**What you'll learn:** when workers need to talk to each other, not just report back.

### Teams vs subagents

| | Subagents | Agent teams |
|---|---|---|
| **Context** | Own window; results return to the caller | Own window; fully independent |
| **Communication** | Report to the main agent only | **Teammates message each other directly** |
| **Coordination** | Main agent manages all work | **Shared task list with self-coordination** |
| **Best for** | Focused tasks where only the result matters | Work requiring discussion and challenge |
| **Token cost** | Lower — results summarized back | **Higher — each teammate is a full Claude instance** |

Teammates are peers, not subordinates. You can also message any teammate directly without going through the lead.

### Enabling it

Agent teams are **experimental and disabled by default**:

```json
{ "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" } }
```

Without that variable no team forms, no team directories are written, and Claude won't spawn or propose teammates.

### Starting one

Describe the task and the teammates in natural language:

```text
I'm designing a CLI tool that tracks TODO comments across a codebase. Spawn
three teammates to explore this from different angles: one on UX, one on
technical architecture, one playing devil's advocate.
```

Claude spawns them, populates a shared task list, and synthesizes findings when they finish. Claude may sometimes use subagents instead — they appear in the same panel, so if you specifically want a team, ask again explicitly.

### Display modes

- **In-process** (the default) — all teammates in your main terminal. Arrow keys to select, `Enter` to view and message, `x` to stop, `Ctrl+T` for the task list. Works in any terminal.
- **Split panes** — one pane each. Requires **tmux or iTerm2 with the `it2` CLI**, and is **not supported in VS Code's integrated terminal, Windows Terminal, or Ghostty**.

```json
{ "teammateMode": "auto" }
```

### Architecture

| Component | Role |
|---|---|
| **Team lead** | The main session; spawns teammates and coordinates |
| **Teammates** | Separate Claude Code instances working assigned tasks |
| **Task list** | Shared work items that teammates claim and complete |
| **Mailbox** | JSON file per agent at `~/.claude/teams/{team}/inboxes/{agent}.json` |

Task claiming uses file locking, so two teammates can't grab the same task. Dependencies resolve automatically — completing a blocking task unblocks its dependents.

### Sizing and steering

**Start with 3–5 teammates.** Token costs scale linearly, coordination overhead grows, and returns diminish. With 15 independent tasks, 3 teammates is a good start — three focused teammates often outperform five scattered ones. Aim for 5–6 tasks per teammate.

**Teammates don't inherit the lead's `/model`** by default. Set **Default teammate model** in `/config`, or name the model in your spawn prompt. They *do* inherit the lead's effort level.

**Give them context in the spawn prompt.** Teammates load CLAUDE.md, MCP servers, and skills like any session, but **they do not inherit the lead's conversation history.**

**Avoid file conflicts** by partitioning work so each teammate owns different files. Teams don't isolate teammates in worktrees on their own.

### Quality gates with hooks

A direct callback to Module 7 — three hook events exist specifically for teams, each blockable with exit 2:

- `TeammateIdle` — fires when a teammate is about to go idle; block to send feedback and keep it working
- `TaskCreated` — block to prevent creation
- `TaskCompleted` — block to prevent a premature completion

### Limitations

It's experimental, and these will bite:

- **`/resume` and `/rewind` do not restore in-process teammates.** After resuming, the lead may try to message teammates that no longer exist.
- **Task status can lag** — teammates sometimes fail to mark tasks complete, blocking dependents.
- **No nested teams.** Only the lead manages the team.
- **One team per session**, and the lead is fixed for the session's lifetime.
- **Permissions are set at spawn** from the lead's mode.

### Try it yourself

```text
Users report the app exits after one message instead of staying connected.
Spawn 5 teammates to investigate different hypotheses. Have them talk to each
other to try to disprove each other's theories, like a scientific debate.
```

The debate structure is the mechanism. Sequential investigation anchors on the first plausible theory; independent investigators actively trying to refute each other surface the real root cause more reliably.

### Key takeaways

- Peers that message each other and share a task list, not subordinates
- Off by default — `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`
- 3–5 teammates, partitioned by file ownership, context in the spawn prompt
- Experimental: no teammate restore on resume, task status can lag

---

## Chapter 4 — Dynamic workflows

**What you'll learn:** moving the plan out of a context window and into a script.

### The concept

A dynamic workflow is a JavaScript script that orchestrates subagents at scale. **Claude writes the script** for the task you describe, and a runtime executes it in the background while your session stays responsive.

Reach for one when a task needs more agents than one conversation can coordinate, or when you want the orchestration itself codified as something you can read and rerun.

Because the plan is code, a workflow can apply a repeatable **quality pattern**, not just more agents: independent agents adversarially reviewing each other's findings before anything is reported, or drafting a plan from several angles and weighing them.

**Availability:** Claude Code v2.1.154+, on all paid plans, plus Amazon Bedrock, Google Cloud, and Microsoft Foundry. On Pro, turn it on from the Dynamic workflows row in `/config`.

### Start with the bundled one

```text
/deep-research What changed in the Node.js permission model between v20 and v22?
```

It fans searches across several angles, fetches and cross-checks sources, votes on each claim, and returns a cited report with claims that didn't survive cross-checking filtered out.

### Three ways to trigger one

- Include the keyword **`ultracode`** in a prompt
- Ask in plain language — "use a workflow"
- Set **`/effort ultracode`**, and Claude plans a workflow for every substantive task in the session

> **A security detail worth teaching:** the `ultracode` keyword is an opt-in **only in a prompt you type yourself.** It does not start a workflow from `-p`, from a scheduled task, from an Agent SDK prompt not stamped as human input, or from a webhook payload or pull request comment relayed into the conversation.

### Watching and saving

```text
/workflows
```

The progress view shows each phase with agent counts, token totals, and elapsed time.

| Key | Action |
|---|---|
| `Enter` / `→` | Drill into a phase, then an agent, to read its prompt and result |
| `f` | Filter agents by status |
| `p` | Pause or resume the run |
| `x` | Stop the selected agent, or the whole run |
| `r` | Restart a running agent |
| **`s`** | **Save the run's script as a command** |

Saved workflows go to `.claude/workflows/` (shared with the repo) or `~/.claude/workflows/` (just you), and run as `/<name>` in future sessions. Plugin workflows are namespaced `/plugin-name:workflow-name`.

### Limits

| Constraint | Why |
|---|---|
| **16 concurrent agents** (fewer on limited CPUs) | Bounds local resource use |
| **1,000 agents total per run** | Prevents runaway loops |
| No mid-run user input | Only permission prompts pause a run |
| No filesystem or shell access from the script | Agents do that; the script coordinates |
| No module loading — `import()` fails before the run starts | The body is plain JavaScript |

**Size guideline** — how many agents Claude aims for. Default is **`medium`** (under 15). `small` is under 5, `large` under 50, `unrestricted` lets Claude size it. Set via `/config` or the `workflowSizeGuideline` setting.

### The resume rule, which is subtle

If you stop a run you can resume it, and completed agents usually return cached results. But:

> Cached results stop at the first agent that didn't finish, and **every agent that started after that one runs again, even if it completed.**

Four agents A, B, C, D start in order and you stop while B is running. On resume: A is cached; B reruns because it never finished; **C and D rerun too**, because they started after B — even though both completed.

The practical consequence: **a workflow that fans work across many small agents preserves more progress than one built around a few long agents.**

Resume works only within the same session. Exit Claude Code mid-run and the next session starts fresh.

### Cost

A run can use meaningfully more tokens than doing the same work in conversation. Gauge it by running on a small slice first — one directory instead of the repo.

A run scheduling more than 25 agents, or projecting past 1.5 million tokens, shows a **`Large workflow`** warning in the task panel. It's advisory and doesn't pause anything.

### Try it yourself

```text
use a workflow to audit every route handler under src/routes/ for missing
authentication checks, and adversarially verify each finding before reporting it
```

Then open `/workflows`, drill into a phase, and read what one agent actually did.

### Common pitfalls

- **Stopping a run mid fan-out.** Expensive, per the resume rule.
- **Expecting mid-run sign-off.** There's no user input during a run. For sign-off between stages, run each stage as its own workflow.
- **Forgetting to check `/model` before a large run.** Every agent uses the session model unless the script routes otherwise.

### Key takeaways

- The script holds the plan, so Claude's context holds only the answer
- `/deep-research` is bundled; `ultracode` or plain language starts your own
- 16 concurrent, 1,000 total, default guideline `medium` (<15 agents)
- Resume replays in start order — fan out across small agents

---

## Chapter 5 — Worktrees

**What you'll learn:** how parallel agents edit files without colliding.

A git worktree gives a session or subagent its own checkout, so edits never collide with the main tree.

For subagents, `isolation: worktree` in the frontmatter creates a temporary worktree **branched from your default branch rather than the parent session's `HEAD`**, and removes it automatically when the subagent made no changes.

The settings from Module 6 matter more here, because every isolated subagent gets a checkout:

- **`worktree.sparsePaths`** — check out only the directories a task needs. All worktrees in a session share the same list, so include every path any subagent needs.
- **`worktree.symlinkDirectories`** — symlink `node_modules` back to the main repo instead of duplicating it per worktree.

### Key takeaways

- Worktrees are how parallel edits stay isolated
- `isolation: worktree` branches from the default branch, not the parent's HEAD
- Pair with `sparsePaths` and `symlinkDirectories` on large repos

---

## Chapter 6 — Cross-session messaging

**What you'll learn:** letting independent sessions hand findings to each other.

`ListAgents` and `SendMessage` let separate sessions — yours on this machine, on another machine, or on the web — pass findings directly, without a team.

**A message is just text with a reply address, not your full context.** If you need the whole conversation moved, resume or branch the session instead of messaging it.

### The security model

This is the part worth teaching, because it answers a question learners will have:

> Claude Code tells the receiving agent the message came from **another Claude session, not from you**. A teammate can't approve a permission prompt or supply consent on your behalf, and a teammate that was denied an action **can't relay it to another teammate to bypass the check**.

In auto mode the classifier adds two more checks: it treats a relayed approval claim as untrusted input rather than confirmation from you, and it reviews every inter-agent message before delivery — a message it blocks never reaches the recipient.

So: no, you cannot chain agents to escape permissions.

### Key takeaways

- `ListAgents` and `SendMessage` connect independent sessions
- A message carries text and a reply address, not context
- Inter-agent messages can never launder consent or bypass a denial

---

## Chapter 7 — Hands-on: a multi-agent codebase audit

**Try (subagents):** dispatch three subagents in parallel on the same diff — one for security, one for tests, one for performance — and compare their findings.

**Try (workflow):** run the same audit as a workflow:

```text
use a workflow to audit every route handler under src/routes/ for missing
authentication checks, and adversarially verify each finding before reporting it
```

Compare the two. The subagent version is faster to start; the workflow version gives you a script you can read, save with `s`, and rerun on every branch forever.

**Try (team):** enable agent teams and run the competing-hypotheses debate from Chapter 3 on a real bug. Watch teammates message each other in the agent panel.

**Try (cost):** before a big run, check `/model`, and run the workflow on one directory before the whole repository.

---
<!-- nav -->

[← MCP in 2026](module-09-mcp.md) · [All modules](../README.md#the-course) · [Claude Code Across Every Surface →](module-11-every-surface.md)
