<!-- module: 9 | phase: 2 | format: deep-dive | last_verified: 2026-09-15 -->
# Module 9 — Slash Commands, Skills and Sub-Agents

The portfolio project from Modules 7 and 8 is built and running. This module steps back from that project to cover three ways of making Claude Code itself more efficient to work with: Agent Skills, slash commands, and a first look at sub-agents — the delegation model this course returns to in more depth later.

## Chapter 1: Finding and Using Skills to Improve the Portfolio

### Concept

An Agent Skill is a markdown file with YAML frontmatter that packages instructions, reference material, or a workflow for Claude Code to use. A skill's `description` field tells Claude when it's relevant, and Claude Code loads that description into context on every turn — but loads the full skill content only when the skill actually gets invoked. That's the point of skills as a mechanism: you get reusable, on-demand instructions without paying the context cost of loading them all the time.

Skills come from three places. Bundled skills ship with Claude Code itself — things like `/code-review` and `/doctor`. You can write your own project skills in `.claude/skills/<skill-name>/SKILL.md` (shared with anyone who clones the repo) or personal skills in `~/.claude/skills/<skill-name>/SKILL.md` (available across all your projects). And you can install skills other people published, through the `/plugin` menu or `/plugin install <skill>@<marketplace>`, from a marketplace such as the official `anthropics/claude-plugins-official` one.

A skill can be invoked two ways: you type `/skill-name` directly, or Claude invokes it on its own when your request matches its description — unless the skill sets `disable-model-invocation: true`, which restricts it to manual use only (useful for anything with side effects, like a deploy skill you never want triggered by accident).

### Try it yourself

Look for a skill that fits a real gap in the portfolio project — for example, a web quality or accessibility audit skill you could run against the finished site, or a code-review skill to run before you consider a feature done. Browse what's available with `/plugin`, install one, and run it against your portfolio project directly (either by typing its command or by asking Claude Code a question that should trigger it automatically). Then write a small project skill of your own in `.claude/skills/` — something as simple as a checklist for what "done" means for a portfolio feature (tested, responsive on mobile, no console errors) — and confirm Claude Code picks it up when you ask a matching question.

### Common pitfalls

- **Writing a skill's description too vaguely for Claude to know when to use it.** The description field is what triggers automatic invocation — a vague one like "helps with code" means Claude either never fires it or fires it on the wrong requests. Be specific about what the skill does and when.
- **Installing a marketplace skill without reading what it does first.** A skill runs with whatever tool access its frontmatter grants it; installing one blindly is no different from running an unfamiliar script.
- **Putting side-effecting logic in a skill without `disable-model-invocation: true`.** If a skill can deploy, commit, or send something externally, an automatically-triggered invocation is a real risk. Lock those down to manual use only.

### Key takeaways

- Skills are markdown files with frontmatter — bundled with Claude Code, written by you in `.claude/skills/` or `~/.claude/skills/`, or installed from a marketplace via `/plugin`.
- The `description` field controls automatic invocation; only full content, not the description, is context-expensive, and only loads when the skill actually runs.
- Use `disable-model-invocation: true` on any skill with side effects you don't want triggered by an inferred match.

## Chapter 2: Slash Commands and Intro to Sub-Agents

### Concept

Slash commands are the shorthand you've already been using — `/plan`, `/model`, `/context`, `/compact` — and Claude Code has a full set organized by workflow stage: setup commands like `/init` and `/permissions`, in-session commands like `/plan` and `/effort`, review commands like `/diff` and `/code-review`, and session management commands like `/clear` and `/resume`. Under the hood, a custom slash command is just a skill you invoke by name — writing your own command means writing a skill file, so everything from Chapter 1 about frontmatter and file location applies directly. Where a skill's automatic side benefits from a good description, a command you'll always invoke by hand can set `disable-model-invocation: true` deliberately, since you never need Claude to guess when to run it.

Sub-agents are Claude Code's mechanism for handing a task to a separate, isolated agent instead of doing it in your main conversation. The idea is containment: if a task would flood your context with search results, log output, or files you'll never need to reference again, a sub-agent does that work in its own context window and reports back only a summary. You define one as a markdown file with YAML frontmatter in `.claude/agents/` (project-scoped) or `~/.claude/agents/` (personal, available everywhere), giving it a `name`, a `description` that tells Claude Code when to delegate to it, and optionally a restricted `tools` list and a specific `model`. Claude Code also ships built-in sub-agents you'll recognize from earlier modules — Explore for fast read-only search, Plan for gathering context during plan mode, and General-purpose for broader multi-step work.

This module only introduces the concept. Deciding when a task genuinely benefits from isolation, designing a sub-agent's tool restrictions deliberately, and orchestrating several sub-agents together are covered in full once this course reaches multi-agent architecture.

### Try it yourself

Run `/help` to see the full list of built-in slash commands and pick two or three you haven't used yet — try `/diff` to review your portfolio project's uncommitted changes. Then write one custom slash command as a project skill: something specific to this project, like a command that checks whether every project entry in your JSON data file has the fields the grid component expects. Finally, ask Claude Code to create a simple, read-only sub-agent in `.claude/agents/` — for example, one restricted to `Read`, `Grep`, and `Glob` that reviews your portfolio's file structure and flags anything inconsistent — and invoke it by name to see it run in isolation from your main session.

### Common pitfalls

- **Confusing a custom slash command with something structurally different from a skill.** They're the same mechanism — a command is a skill you plan to invoke directly. Writing one means writing SKILL.md frontmatter, not learning a separate system.
- **Giving a new sub-agent broad tool access by default.** The value of isolating a task into a sub-agent is partly about constraining what it can do. Start narrow — read-only tools for anything exploratory — and widen only if the sub-agent actually needs more.
- **Delegating a task to a sub-agent when you'll need its intermediate output anyway.** Sub-agents are for work whose details you don't need to keep — logs, search results, file contents you won't reference again. If you'll want to see exactly what it found, that's a reason to keep the work in your main session instead.

### Key takeaways

- Slash commands span setup, in-session control, review, and session management; a custom one is written exactly like a skill, just intended for manual invocation.
- Sub-agents run a task in an isolated context and return only a summary, defined as markdown files with frontmatter in `.claude/agents/` or `~/.claude/agents/`, with `tools` and `model` as the main levers for scoping one down.
- This module is an introduction — designing and orchestrating sub-agents deliberately is a later module's territory, not this one's.

<!-- nav -->
[← Module 8: Executing and Building the Project](module-08-executing-and-building.md) | [Module 10: MCP Foundations →](module-10-mcp-foundations.md)
