<!-- module: 13 | phase: 3 | format: deep-dive | last_verified: 2026-09-15 -->
# Module 13 — Prompting Strategies and Multi-Agent Architecture

With the URL shortener project behind you, you've felt the limits of a single Claude Code session working alone: it reasons about one thing at a time, and its context window fills up the longer a session runs. This module covers three ways to get past that — writing prompts that hand out clear roles, splitting independent work across sub-agents that run in parallel, and keeping the context window from filling with things that don't matter anymore.

## Chapter 1: Defining Agent Roles and Responsibilities

### Concept

A single Claude Code session with a single, unstructured prompt tends to blur together research, planning, and implementation in one pass. On a small task that's fine. On a larger one — say, reviewing a pull request for both security issues and code style — it helps to split the work into roles, the same way a human team would split a security reviewer from a style reviewer, each with a narrower brief and less to hold in their head at once.

Claude Code supports this directly through **subagents**: specialized configurations with their own system prompt, their own restricted tool access, and (optionally) their own model. You define one as a Markdown file with YAML frontmatter, stored in `.claude/agents/` for a project-level subagent that's checked into the repo, or `~/.claude/agents/` for one you want available across every project. The frontmatter sets `name`, `description`, and optionally `tools` and `model`; the Markdown body underneath becomes the subagent's system prompt, replacing Claude Code's default prompt entirely for that subagent.

```yaml
---
name: security-auditor
description: Audits code for security vulnerabilities
tools: Read, Grep, Bash
model: opus
---

You are a security expert specializing in code audits. When analyzing code:
- Look for common vulnerabilities (SQL injection, XSS, CSRF, and similar issues)
- Check authentication and authorization logic
- Flag sensitive data handling issues
- Explain the risk level and business impact of each finding, not just the finding itself
```

Restricting `tools` matters as much as the prompt. A security auditor that can only `Read` and `Grep` can't accidentally "fix" something it was only asked to flag — it reports, and a separate step (you, or a different subagent with write access) decides what to do about it.

### Try it yourself

In a project you're comfortable editing, create `.claude/agents/style-reviewer.md` with a persona that reviews only for naming, formatting, and consistency — nothing about logic or security. Give it `tools: Read, Grep` so it can't edit files. Then ask Claude Code, in a normal session: "Use the style-reviewer subagent to review the changes in this branch." Notice that the subagent's report comes back as a single result in your main conversation — its own back-and-forth investigating the code doesn't clutter your session's context.

### Common pitfalls

- **Writing a subagent description that's too vague.** Claude Code decides when to invoke a subagent automatically based on its `description` field, so "helps with code" won't get picked up reliably — describe the specific situation that should trigger it.
- **Giving every subagent every tool.** A subagent scoped to review shouldn't have `Edit` or `Bash` access just because it's convenient — the point of a role is that it can't do more than its job requires.
- **Expecting the main session to see the subagent's intermediate reasoning.** It doesn't — only the subagent's final response returns to the conversation that invoked it. If you need to debug what a subagent was doing, that's a separate concern from what it reports back.

### Key takeaways

- Subagents are Markdown files with YAML frontmatter, stored in `.claude/agents/` (project) or `~/.claude/agents/` (personal).
- The frontmatter's `tools` field is a permissions boundary, not just documentation — use it to keep a role honest.
- A subagent's system prompt (the Markdown body) fully replaces the default one, so persona and constraints belong there, not just in the description.

## Chapter 2: Persona Prompting and Parallel Sub-Agents

### Concept

Persona prompting is the technique of writing a role directly into a prompt or a subagent's system prompt — "You are a security expert specializing in code audits," rather than "check this code for security issues." The persona narrows the model's attention to the kind of thing that role would notice, and it shapes tone and structure in the output too: a security auditor's report reads differently from a style reviewer's, even given the same diff.

The bigger productivity gain, though, comes from running multiple subagents **in parallel** on independent pieces of work. If a task naturally splits into pieces that don't depend on each other — reviewing three unrelated modules, researching three different libraries, auditing three microservices — you can dispatch a subagent for each piece in a single instruction, and Claude Code runs them concurrently rather than one after another. You ask for this directly:

```text
Use the auth-researcher subagent to investigate authentication mechanisms.
Use the db-researcher subagent to explore the database layer.
Use the api-researcher subagent to analyze the API endpoints.
Synthesize findings from all three.
```

Each subagent runs with its own isolated context window, does its own tool calls, and returns a result independently. Your main session waits for all of them, then works with the combined output. This is a different mechanism from asking one subagent to do three things in sequence — sequential work in a single subagent shares one context window and finishes only as fast as the slowest step happens after the one before it; parallel subagents each start immediately and finish whenever their own piece of work is done.

The gain shows up most on read-heavy, independent work — research, audits, exploration across unrelated parts of a codebase. It's a poor fit for work with real dependencies: if reviewing module B requires knowing what you found in module A, parallelizing them just means B's subagent starts without the context it needs.

### Try it yourself

Pick a repository with at least three independent areas (three unrelated directories, three services, three libraries a project depends on). Ask Claude Code to use general-purpose subagents to investigate all three in parallel, then summarize the combined findings. Compare how the request reads and how long it takes against asking the same three questions one at a time in a single conversation.

### Common pitfalls

- **Parallelizing work that isn't actually independent.** If one subagent's output should inform another's starting point, running them in parallel throws away that dependency — do those sequentially in a single conversation instead.
- **Over-provisioning subagents for small tasks.** Spinning up three subagents to review three one-line changes costs more in coordination overhead than just doing the review directly.
- **Assuming persona alone fixes weak instructions.** "You are an expert Python reviewer" without concrete review criteria still produces vague output — the persona sets tone and focus, not substance; pair it with specifics about what to check.

### Key takeaways

- Persona prompting narrows focus and shapes output structure by naming a role and its priorities up front, in a system prompt or a one-off instruction.
- Dispatching several subagents in one instruction runs them in parallel, each with an isolated context window; the main session synthesizes their results once they're all back.
- Parallel subagents pay off on independent, read-heavy work — research, audits, cross-cutting exploration — not on work with real sequential dependencies.

## Chapter 3: Managing Context with /compact and Auto-Compact

### Concept

Claude Code sends the full conversation history with every request in a session, so a long session — hours of back-and-forth, dozens of file reads, several rounds of test output — eventually consumes a large share of the context window just carrying its own history forward. Two mechanisms in Claude Code manage this:

- **`/compact`** is a command you run yourself. It summarizes the conversation so far, replacing the detailed history with a condensed version, and frees up space for the work ahead. You can pass instructions with it — `/compact Focus on code samples and API usage` — telling Claude what's worth preserving in the summary and what isn't. Run in a session with no meaningful history yet, `/compact` reports `Not enough messages to compact.` rather than doing anything.
- **Auto-compact** triggers this same summarization automatically once a session's context usage crosses a threshold, so you don't have to babysit context usage yourself. That threshold — the auto-compact window — is configurable through Claude Code's model settings if the default doesn't suit how you work.

Both are different from **`/clear`**, which drops the conversation entirely and starts a fresh session with no summary at all. Use `/compact` when you want continuity — the task is ongoing and older context is still relevant but doesn't need to be replayed in full. Use `/clear` when you're switching to genuinely unrelated work and carrying any of the old context forward would just waste tokens.

Compaction itself isn't free: `/compact` has to read the entire conversation it's about to summarize, so triggering it on a very large context is itself a sizeable request. That's part of why proactive habits — clearing between unrelated tasks, keeping CLAUDE.md concise, delegating verbose operations like test runs to subagents so their raw output doesn't sit in the main conversation — matter more than only reacting to compaction warnings when they appear.

### Try it yourself

Run a real working session until it's substantial — several file edits, a few rounds of test output. Run `/compact Focus on what changed and why, not the full diffs` and read the summary Claude Code produces. Then start a completely unrelated task in the same session and notice whether the summary still gives Claude enough continuity, or whether you'd have been better off with `/clear` instead.

### Common pitfalls

- **Waiting for an auto-compact warning before ever managing context yourself.** By the time you see one, the session has already been carrying a lot of dead weight — proactively clearing or compacting between tasks is cheaper than reacting.
- **Using `/compact` when `/clear` was the right call.** If the next task has nothing to do with the last one, compaction still forces Claude to reason about a summary of irrelevant history; clearing is free and cleaner.
- **Assuming compaction preserves everything important by default.** Without custom instructions, the summarizer makes its own judgment about what to keep — if a specific detail matters (an exact error message, a specific file path), say so in the `/compact` instruction rather than assuming it survives.

### Key takeaways

- `/compact` summarizes conversation history on demand and accepts custom instructions about what to preserve; auto-compact does the same thing automatically once context usage crosses a configurable threshold.
- `/clear` discards history entirely and costs nothing — use it between unrelated tasks instead of compacting context you don't need.
- Compacting a large context is itself an expensive request, so proactive habits (clearing early, keeping CLAUDE.md lean, offloading verbose output to subagents) matter more than relying on compaction alone.

<!-- nav -->
[← Module 12: Advanced Project: URL Shortener](module-12-url-shortener-project.md) | [Module 14: CI/CD and Reliability →](module-14-cicd-and-reliability.md)
