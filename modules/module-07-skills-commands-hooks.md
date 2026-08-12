<!-- module: 7 | phase: 2 | format: deep-dive | last_verified: 2026-08-12 -->

# Module 7 — Extending Claude Code: Skills, Commands, and Hooks

Modules 5 and 6 were about configuring Claude Code. This one is about extending it — teaching it things it doesn't know, and making certain things happen whether it decides to or not.

The distinction that organizes the whole module:

> **CLAUDE.md is advisory. Settings are enforced by the client. Hooks are enforced before anything else runs.**

Skills add knowledge and workflows. Hooks add guarantees. Knowing which one you need is the skill this module actually teaches.

---

## Chapter 1 — Agent Skills (SKILL.md)

**What you'll learn:** what a skill is, where it lives, and how Claude decides to use one.

### The concept

A skill is a directory containing a `SKILL.md`: YAML frontmatter, then instructions.

```markdown
---
name: api-conventions
description: REST API design conventions for our services
---

# API Conventions

- Use kebab-case for URL paths
- Use camelCase for JSON properties
- Always include pagination for list endpoints
- Version APIs in the URL path (/v1/, /v2/)
```

Create a skill when you keep pasting the same instructions into chat, or when a section of CLAUDE.md has grown from a fact into a procedure. **A skill's body loads only when it's used**, so long reference material costs almost nothing until you need it. That's the key difference from CLAUDE.md, which loads in full every session.

### Where skills live

| Level | Path | Available in |
|---|---|---|
| Personal | `~/.claude/skills/<name>/SKILL.md` | All your projects |
| Project | `.claude/skills/<name>/SKILL.md` | This project only |
| Plugin | `<plugin>/skills/<name>/SKILL.md` | Wherever the plugin is enabled |

When names collide, enterprise overrides personal, and personal overrides project. A skill at any level overrides a bundled one — a `code-review` skill in your project replaces the built-in `/code-review`. Plugin skills are namespaced `plugin-name:skill-name`, so they never collide with anything.

Skill directories are **watched live**. Add or edit a `SKILL.md` and Claude Code picks it up within the current session, no restart needed.

### Frontmatter that changes behavior

There are around twenty fields. These are the ones worth knowing:

| Field | Why it matters |
|---|---|
| `description` | How Claude decides to auto-invoke the skill. **`description` and `when_to_use` are truncated together at 1,536 characters** in the skill listing — put the key use case first |
| `when_to_use` | Extra trigger phrases; counts toward the same 1,536-character cap |
| `disable-model-invocation: true` | Manual only, via `/name`. Use for anything with side effects |
| `allowed-tools` | Tools pre-approved for the turn that invokes the skill; the grant clears on your next message |
| `disallowed-tools` | Tools removed while the skill is active |
| `context: fork` | Runs the skill in its own subagent context |
| `paths` | Auto-load only when Claude works with matching files |
| `model` / `effort` | Override for the duration of the skill |

### A portability caveat

Skills follow the [Agent Skills](https://agentskills.io) open standard, so they work across compatible tools — but **only some fields are part of that standard.** Outside Claude Code, the accepted set is `name`, `description`, `license`, `compatibility`, `metadata`, and `allowed-tools`.

Ship a skill using a Claude Code extension field elsewhere and you get:

```
Unexpected key(s) in SKILL.md frontmatter: argument-hint.
Allowed properties are: allowed-tools, compatibility, description, license, metadata, name
```

If portability matters for a skill, stick to the standard fields.

### Try it yourself

Create `~/.claude/skills/commit-style/SKILL.md` with your team's commit message conventions. Then, without invoking it, ask Claude to commit something and see whether it picks the skill up from the description alone.

### Common pitfalls

- **A vague description.** Claude decides from the description; "helps with API stuff" won't trigger reliably. Lead with the concrete use case.
- **Putting a procedure in CLAUDE.md.** If it's multi-step and only sometimes relevant, it's a skill. CLAUDE.md loads every session; a skill loads on demand.
- **Forgetting `disable-model-invocation` on a skill with side effects.** A skill that opens pull requests shouldn't fire because Claude thought it was relevant.

### Key takeaways

- A skill is a `SKILL.md` in a directory; the body loads only when used
- The `description` is the trigger — write it for matching, and keep it under the cap
- Enterprise > personal > project; plugin skills are namespaced

---

## Chapter 2 — Custom slash commands

**What you'll learn:** how commands and skills became the same thing, and how to pass arguments.

### The concept

Custom commands have been merged into skills. A file at `.claude/commands/deploy.md` and a skill at `.claude/skills/deploy/SKILL.md` both create `/deploy` and work the same way. Existing `commands/` files keep working.

Skills are the recommended form because they add a directory for supporting files, frontmatter for controlling invocation, and the ability for Claude to load them automatically.

```markdown
---
name: fix-issue
description: Fix a GitHub issue
argument-hint: [issue-number]
disable-model-invocation: true
---

Analyze and fix the GitHub issue: $ARGUMENTS.

1. Use `gh issue view` to get the issue details
2. Understand the problem described in the issue
3. Search the codebase for relevant files
4. Implement the fix
5. Write and run tests to verify it
6. Create a descriptive commit message and open a PR
```

Run it with `/fix-issue 1234`.

- `$ARGUMENTS` inserts everything passed after the command
- The `arguments` frontmatter field defines named positional arguments for `$name` substitution
- `argument-hint` drives the autocomplete hint

Built-ins worth knowing for contrast: `/clear`, `/compact`, `/context`, `/hooks`, `/doctor`, `/memory`, `/code-review`.

### Try it yourself

Write a `/standup` skill that runs `git log --since=yesterday --author=$(git config user.email)` and summarizes what you did. Set `disable-model-invocation: true` so it only runs when you ask.

### Common pitfalls

- **Expecting `$ARGUMENTS` to split into separate values.** It inserts the whole argument string. Use named `arguments` for positional parsing.

### Key takeaways

- Commands and skills are one mechanism; skills are the richer form
- `$ARGUMENTS` for the whole string, named `arguments` for positional
- `disable-model-invocation: true` keeps a side-effecting command manual

---

## Chapter 3 — Automating actions with hooks

**What you'll learn:** the one mechanism in Claude Code that is genuinely enforced.

### The concept

Hooks are user-defined shell commands that Claude Code runs at specific lifecycle points. Unlike skills, which are a judgment call, **hooks run every time**.

And this is the fact that makes them the foundation of anything policy-related:

> **`PreToolUse` hooks fire before any permission-mode check, in every permission mode, including `dontAsk`. A hook returning `permissionDecision: "deny"` blocks the tool even in `bypassPermissions` mode or with `--dangerously-skip-permissions`.**

Nothing else in Claude Code can make that claim. If a rule must hold regardless of what a developer sets locally, it is a hook.

### The events

| Group | Events |
|---|---|
| Session lifecycle | `SessionStart`, `SessionEnd`, `Setup` |
| Prompt and turn | `UserPromptSubmit`, `Stop`, `StopFailure`, `MessageDisplay` |
| Tools | `PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `PostToolBatch` |
| Permissions | `PermissionRequest`, `PermissionDenied` |
| Subagents and teams | `SubagentStart`, `SubagentStop`, `TeammateIdle`, `TaskCreated`, `TaskCompleted` |
| Context | `PreCompact`, `PostCompact`, `InstructionsLoaded` |
| Environment | `WorktreeCreate`, `WorktreeRemove`, `CwdChanged`, `ConfigChange` |

Most events support **matchers**, so a hook only fires on what you care about:

- `PreToolUse`, `PostToolUse` — match on tool name: `Bash`, `Edit|Write`, `mcp__.*`
- `SessionStart` — `startup`, `resume`, `clear`, `compact`, `fork`
- `PreCompact` / `PostCompact` — `manual`, `auto`
- `InstructionsLoaded` — `session_start`, `nested_traversal`, `path_glob_match`, `include`, `compact`

`UserPromptSubmit`, `Stop`, and several others have no matcher — they fire on every occurrence.

### Exit codes

Your script tells Claude Code what to do by what it writes and how it exits:

| Exit | Meaning |
|---|---|
| **0** | No decision. Proceed |
| **2** | **Block.** Write the reason to stderr; on most events it's fed back to Claude so it can adjust |
| Anything else | Non-blocking error. The action proceeds; you get a `hook error` notice |

Some events can't be blocked — on `SessionStart` and `Setup`, exit 2 shows stderr to the user and execution continues anyway.

### Two behaviors that will bite you

**Hooks on the same event run in parallel, and deny wins.** If a logging hook exits 0 and a guardrail hook exits 2, the command is blocked *and* the log entry is still written, because the logging hook already ran.

**A `Stop` hook is overridden after it blocks eight times in a row.** Your script must read the `stop_hook_active` field from its JSON input and exit early when it's `true`, or you've built an infinite loop.

### Handler types

| Type | What it does |
|---|---|
| `command` | Runs a shell script. The workhorse |
| `http` | POSTs to an endpoint |
| `mcp_tool` | Calls an MCP tool |
| **`prompt`** | **A Claude model evaluates the condition** |
| **`agent`** | **A subagent evaluates it** |

The last two are the answer to "I need a rule, but it requires judgment a regex can't make." A `prompt` hook can decide whether a commit message is descriptive enough; a regex can't.

### What people actually build

- **Auto-format after edits** — `PostToolUse` matching `Edit|Write`, running your formatter
- **Block edits to protected files** — `PreToolUse`, exit 2 with a reason
- **Desktop notification when Claude needs input** — `Notification`
- **Re-inject context after compaction** — `PostCompact`
- **Audit configuration changes** — `ConfigChange`
- **Auto-approve specific permission prompts** — `PermissionRequest`

You don't have to write them by hand. Ask: *"Write a hook that runs eslint after every file edit"* or *"Write a hook that blocks writes to the migrations folder."* Browse what's registered with `/hooks`.

### Try it yourself

Ask Claude to write you a `PostToolUse` hook that runs your project's formatter after every edit. Then check it with:

```
/hooks
```

### Common pitfalls

- **Using a hook where a skill would do.** Hooks are for guarantees, not guidance. A hook that runs on every edit costs time on every edit.
- **Writing a `Stop` hook without the `stop_hook_active` check.** Infinite loop until Claude Code overrides it.
- **Assuming a non-zero exit blocks.** Only exit 2 blocks. Anything else is a non-blocking error and the action proceeds.

### Key takeaways

- Hooks are deterministic and run before permission checks — the only true enforcement layer
- Exit 0 proceeds, exit 2 blocks, anything else is a non-blocking error
- Parallel hooks on one event all run, and deny wins
- `prompt` and `agent` handlers cover rules that need judgment

---

## Chapter 4 — Output styles

**What you'll learn:** how to change Claude's voice and format without touching what it knows.

### The concept

Output styles modify the system prompt directly, changing role, tone, and default response format on every turn. Use one when you keep re-prompting for the same voice, or when Claude isn't doing software engineering at all.

Built-in styles:

- **Default** — the standard software-engineering system prompt
- **Proactive** — executes immediately, makes reasonable assumptions instead of pausing for routine decisions, prefers action over planning
- **Explanatory** — adds educational "Insights" while completing tasks
- **Learning** — collaborative; leaves `TODO(human)` markers in your code for you to implement

Select with `/config` → **Output style**, or set it directly:

```json
{ "outputStyle": "Explanatory" }
```

Custom styles are markdown files in `~/.claude/output-styles/` or `.claude/output-styles/`:

```markdown
---
name: Diagrams first
description: Lead every explanation with a diagram
keep-coding-instructions: true
---

When explaining code, architecture, or data flow, start with a Mermaid diagram
showing the structure, then explain in prose.
```

### Two things that catch people out

**Without `keep-coding-instructions: true`, a custom style strips Claude Code's built-in software-engineering instructions** — how to scope changes, write comments, and verify work. Correct for a writing assistant, a footgun if you only wanted diagrams.

**Output style is read once at session start.** A change takes effect after `/clear` or in a new session.

Styles apply to the main conversation only. A subagent runs its own system prompt, so styles don't reach it — except a fork, which inherits the parent's.

### Try it yourself

Switch to **Learning** with `/config`, run `/clear`, and ask Claude to build a small feature. Watch for the `TODO(human)` markers.

### Common pitfalls

- **Changing the style and not seeing a difference.** Run `/clear` — it's read at session start.
- **Using an output style for project conventions.** That's CLAUDE.md. Styles are about voice and format.

### Key takeaways

- Four built-ins; Explanatory and Learning are genuinely useful while learning
- `keep-coding-instructions: true` unless Claude isn't coding
- Read at session start, main conversation only

---

## Chapter 5 — Hands-on: building your own skill

### Build a `/security-review` skill

Create `.claude/skills/security-review/SKILL.md`:

```markdown
---
name: security-review
description: Review the current diff for hardcoded secrets, injection risk, and unsanitized input. Use when the user asks for a security review before committing.
allowed-tools: Read Grep Glob Bash(git diff:*)
---

Review the current diff for:
- Hardcoded secrets, API keys, or credentials
- SQL, command, or template injection risk
- Unsanitized user input reaching a sink
- Authentication or authorization gaps

For each finding, give the file, the line, why it's a problem, and a suggested fix.
Report findings only — do not fix anything unless asked.
```

Run it with `/security-review`, then try triggering it by description alone: *"can you check this diff for security problems before I commit?"*

### Then feel the difference between guidance and enforcement

This exercise is the point of the whole module. Do all three steps.

1. **Guidance.** Add to CLAUDE.md: `Never edit files in migrations/.` Ask Claude to edit one. It'll usually refuse.
2. **Guidance under pressure.** Switch to `bypassPermissions` and ask again. Notice that compliance is a judgment call, not a guarantee.
3. **Enforcement.** Add a `PreToolUse` hook that denies writes to `migrations/`, and try again — still in `bypassPermissions`. **Blocked.**

Three layers, one demo. Once you've seen step 3 succeed where step 2 didn't, you'll never again put a rule that matters in a markdown file.

---
<!-- nav -->

[← Planning and Executing a Real Project](module-06-planning-real-projects.md) · [All modules](../README.md#the-course) · [The Plugin Ecosystem →](module-08-plugin-ecosystem.md)
