# Power moves

The techniques that only make sense once you've seen the whole course. Each one combines things from different modules — which is why none of them live in a single module.

Nothing here is new material. It's the same features, used together.

---

## The three laws everything else follows from

1. **Give it a way to verify.** Without a check it can run, "looks done" is the only signal available — and *you* become the verification loop.
2. **Context is the constraint.** Performance degrades as the window fills. Almost every technique below is about getting the right information in and keeping the wrong information out.
3. **Guidance is not enforcement.** `CLAUDE.md` is advisory. Settings are enforced by the client. Hooks fire regardless. If a rule actually matters, it's a hook.

---

## Context moves

### Read `/context` before you `/compact`

`/compact` is the reflex. `/context` is the diagnosis. It shows what's *actually* filling the window — system prompt, tools, MCP tools, subagents and their source, memory files, skills, conversation.

Half the time the answer isn't "compact the conversation," it's "an MCP server is loading 40 tools I'm not using" or "my `CLAUDE.md` is 600 lines." Compacting doesn't fix either.

→ [Module 4](../modules/module-04-agentic-loop-tools.md)

### Compact with a focus, not blindly

```
/compact keep only the plan and the diff
```

Bare `/compact` summarises everything with equal weight, which is how you lose the one detail you needed. Naming what to keep turns a lossy operation into a deliberate one.

→ [Module 4](../modules/module-04-agentic-loop-tools.md)

### Use `/btw` for anything you don't want in the history

A side question whose answer never enters the conversation. "What's the syntax for a bash trap again?" doesn't need to occupy context for the rest of the session — and won't distract the next twenty turns.

→ [Module 2](../modules/module-02-prompt-engineering.md)

### Push big reads into a subagent

A subagent runs in its own context window and returns only its conclusion. Reading a 4,000-line log to find one stack trace costs you the whole log in the main conversation — or costs you nothing, if a subagent reads it and reports back.

This is also the fix for `Autocompact is thrashing`: the file that keeps refilling the window shouldn't be in the main window at all.

→ [Module 10](../modules/module-10-subagents-orchestration.md) · [Troubleshooting](troubleshooting.md#autocompact-is-thrashing)

### `/clear` between unrelated tasks, not `/compact`

Compaction preserves a summary of work you're finished with. If the next task shares nothing with the last one, that summary is pure cost — and worse, a source of confused cross-contamination. Clear it.

→ [Module 2](../modules/module-02-prompt-engineering.md)

---

## Loop moves

### Plan first on anything you'd review as a PR

Plan mode separates "decide what to do" from "do it." The value isn't safety — it's that reviewing a plan costs you thirty seconds and reviewing a wrong implementation costs you twenty minutes.

Rule of thumb: if the change would need a PR review, it needs a plan.

→ [Module 6](../modules/module-06-planning-real-projects.md)

### Branch the conversation instead of restarting it

```
/branch try-the-other-approach
```

You've spent forty turns building context and now want to try something different. `/branch` copies the conversation and switches you into the copy — the original stays intact. Restarting throws away the expensive part.

→ [Module 6](../modules/module-06-planning-real-projects.md)

### `/rewind` beats `git checkout` for a bad turn

`/rewind` restores code *and* conversation from an earlier point. `git checkout` restores the code and leaves Claude convinced it already made the change.

→ [Module 6](../modules/module-06-planning-real-projects.md)

### Give it a finish line with `/goal`

`/goal` keeps Claude working until a completion condition holds, rather than until it decides it's done. Pair it with something machine-checkable — a passing test suite, a clean `terraform plan`, a green build — and law #1 takes care of itself.

→ [Module 2](../modules/module-02-prompt-engineering.md)

---

## Configuration moves

### Promote a rule from `CLAUDE.md` to a hook the second it's violated twice

The escalation ladder, in order of strength:

| Layer | Strength |
|---|---|
| `CLAUDE.md` | Advisory — Claude usually follows it |
| Permission rules | Enforced by the client |
| Hooks | Fire regardless of what Claude decides |

If you've written "never edit files in `/generated`" in `CLAUDE.md` and it's happened twice, writing it a third time in bold won't help. It's a deny rule or a `PreToolUse` hook.

→ [Module 5](../modules/module-05-permissions-memory-config.md) · [Module 7](../modules/module-07-skills-commands-hooks.md)

### Let `/doctor` trim your `CLAUDE.md`

`CLAUDE.md` grows. Every line dilutes the others, and a lot of what accumulates is content Claude can derive from the codebase anyway — the framework, the directory layout, the test command that's already in `package.json`. `/doctor` finds that content and proposes cutting it.

→ [Module 5](../modules/module-05-permissions-memory-config.md)

### Deny-read your secrets, don't rely on Claude avoiding them

A `Read` deny rule on `.env` is the difference between "Claude has been asked not to look" and "Claude cannot look." It also stops the file's contents leaking through IDE selection sharing, which is the path people forget.

→ [Module 5](../modules/module-05-permissions-memory-config.md) · [Module 11](../modules/module-11-every-surface.md)

### Restate critical instructions when delegating to Explore or Plan

The built-in Explore and Plan agents **skip `CLAUDE.md`**. Your conventions don't reach them. Put anything that matters into the delegating prompt itself.

→ [Module 10](../modules/module-10-subagents-orchestration.md) · [Troubleshooting](troubleshooting.md#claude-is-ignoring-my-claudemd)

---

## Cost and speed moves

### Effort level is the biggest single dial

```
/effort low        # mechanical edits, formatting, renames
/effort high       # the genuinely hard debugging
```

Bigger than model choice for most tasks. Most work does not need `max`, and running everything at `max` is how a plan limit disappears by Wednesday.

→ [Module 10](../modules/module-10-subagents-orchestration.md)

### Protect the prompt cache

Two habits, both cheap:

- **Pick a model at session start** and stay on it. Switching invalidates the cache.
- **Don't edit `CLAUDE.md` mid-task.** It sits near the front of the context, so changing it invalidates everything cached behind it.

→ [Module 4](../modules/module-04-agentic-loop-tools.md)

### Find out what's actually costing you

```
/usage
```

Breaks your plan usage down by skill, subagent, plugin, and MCP server. The answer is often one MCP server loading a large tool list on every single turn.

→ [Module 12](../modules/module-12-cicd-review-security.md)

---

## Automation moves

### Always `--bare` in CI

```bash
claude --bare -p "..." --allowedTools "Read"
```

Without it, a CI run loads whatever hooks, plugins, MCP servers, and memory happen to exist on the runner — so the same commit can produce different results on different machines. `--bare` skips discovery entirely, and Anthropic's docs say it will become the default for `-p` eventually.

→ [Module 12](../modules/module-12-cicd-review-security.md#1-headless-mode--claude-code-without-a-terminal)

### Fail the build when your tooling didn't load

A plugin that silently fails to load turns a security review into a run that finds nothing and exits `0` — the worst possible failure, because it looks like success.

Read `plugin_errors` and `mcp_server_errors` from the `system/init` event in `stream-json`. Both keys are omitted entirely when there are no errors, so a CI gate can just fail on a non-empty array.

→ [Module 12](../modules/module-12-cicd-review-security.md)

### Pipe context in instead of granting permission to fetch it

```bash
git diff main | claude -p "review this for security issues"
```

Piping the diff means Claude never needs Bash permission to run `git diff`. Every permission you don't grant is one you don't have to reason about.

→ [Module 12](../modules/module-12-cicd-review-security.md)

### Mind the space in prefix rules

```
Bash(git diff *)     ✅  matches "git diff HEAD"
Bash(git diff*)      ❌  also matches "git diff-index"
```

One character, meaningfully different blast radius.

→ [Module 12](../modules/module-12-cicd-review-security.md)

---

## Debugging moves

### `--safe-mode` first, always

```bash
claude --safe-mode
```

Disables every customisation — `CLAUDE.md`, skills, plugins, hooks, MCP servers, custom commands and agents — while keeping auth, models, built-in tools, and permissions working. One command tells you whether the problem is *yours* or *Claude Code's*. Do this before reading any documentation.

→ [Troubleshooting](troubleshooting.md#start-here-the-four-commands)

### Ask Claude about Claude Code

It has built-in access to its own documentation. For "does this flag exist," "what changed in this version," or "why isn't my hook firing," asking directly is usually faster than searching — and it can read your actual config while it answers.

→ [Module 16](../modules/module-16-staying-sharp.md)

---

## Anti-patterns

Things that feel like power moves and aren't.

| Looks smart | Why it isn't |
|---|---|
| Running everything at `max` effort | Costs a multiple for tasks where it changes nothing. Match effort to difficulty |
| A 500-line `CLAUDE.md` | Every line dilutes the others. Run `/doctor` and cut what Claude can derive |
| `--dangerously-skip-permissions` as a default | You've removed the layer that catches the one command you'd have stopped. Use `dontAsk` with real allow rules instead |
| Twenty MCP servers connected "just in case" | Their tool definitions occupy context on every turn, in every session. Connect what the task needs |
| Writing a rule in `CLAUDE.md` a third time | It didn't work twice. Make it a hook |
| Spawning agents for a task one agent can do | Parallelism costs coordination. It pays off on genuinely independent work, not on a linear task |
| `/compact` on autopilot | Diagnose with `/context` first. Sometimes the answer is `/clear`, and sometimes it's an MCP server |

---

**Related:** [Topic index](topics.md) · [Command reference](commands.md) · [Quick reference](quick-reference.md) · [Troubleshooting](troubleshooting.md)
