# Claude Code: Zero To Hero

Learn Claude Code end to end — from what an AI agent actually is to shipping and deploying a real project with it. Written for developers, DevOps, cloud, SRE, and platform engineers who want to actually use it, not just watch a demo.

Fifteen modules, four phases, one project that carries through the second half of the course. Every fact is verified against Anthropic's official documentation, and every module carries the date it was last checked.

**Course:** [trainwithshubham.ai](https://trainwithshubham.ai) · **Language:** English · **Format:** Self-paced · **Verified:** 2026-09-15

---

## Jump in

| | |
|---|---|
| 🚀 **New to Claude Code** | Start at [Module 1](modules/module-01-claude-and-how-it-works.md) and read in order |
| ⚡ **Already know the agent basics** | Skip to [Module 4 — getting started](modules/module-04-getting-started.md) |
| 🔧 **Here for DevOps/SRE** | [Module 14](modules/module-14-cicd-and-reliability.md) is the one written for you |
| 🔍 **Looking for one thing** | [Topic index](reference/topics.md) · [Command reference](reference/commands.md) |
| 📋 **Want the short version** | [Quick reference](reference/quick-reference.md) — all 15 modules condensed |
| 🩹 **Something's broken** | [Troubleshooting](reference/troubleshooting.md) — symptom → cause → fix |
| ⚡ **Already good, want sharper** | [Power moves](reference/power-moves.md) — the compound techniques |

---

## The course

### Phase 1 · Zero → Comfortable

> Understand what you're using before you touch a keyboard.

| # | Module | What you'll learn |
|---|---|---|
| **01** | [Understanding Claude and How It Works](modules/module-01-claude-and-how-it-works.md) | How Claude got here — rule-based systems to ML to generative to agentic AI — and Claude's current model and product lineup |
| **02** | [Prompt Engineering Foundations](modules/module-02-prompt-engineering-foundations.md) | Prompts, tokens, and the context window; structuring and iterating on prompts; getting concise output and avoiding common mistakes |
| **03** | [Understanding Agentic AI Systems](modules/module-03-agentic-ai-systems.md) | How an LLM, tool use, and memory combine into an agent — the loop every later module builds on |

### Phase 2 · Comfortable → Productive

> Install it, configure it, and build a first real project with it.

| # | Module | What you'll learn |
|---|---|---|
| **04** 🔬 | [Getting Started with Claude Code](modules/module-04-getting-started.md) | What Claude Code is and where to run it, logging in and choosing a model, running models locally and free with Ollama |
| **05** 🔬 | [Claude Code Setup, Configuration and Memory](modules/module-05-setup-configuration-memory.md) | Settings files and CLAUDE.md, Claude Code's modes, and guiding a project with CLAUDE.md and memory |
| **06** 🔬 | [Built-in Tools and Working with Codebases](modules/module-06-built-in-tools-codebases.md) | Bash and file-operation tools, and reviewing code and fixing bugs with Claude Code |
| **07** 🔬 | [Planning a Complete Project With Claude Code](modules/module-07-planning-a-project.md) | Defining requirements, features, and a tech stack in plan mode; reviewing and refining the plan |
| **08** 🔬 | [Executing and Building the Project](modules/module-08-executing-and-building.md) | Starting the build and tracking progress, then running, testing, and debugging the result |
| **09** 🔬 | [Slash Commands, Skills and Sub-Agents](modules/module-09-slash-commands-skills-subagents.md) | Finding and using Agent Skills, writing slash commands, and a first look at sub-agents |

### Phase 3 · Productive → Advanced

> Connect Claude Code to more tools, automate it, and build something bigger.

| # | Module | What you'll learn |
|---|---|---|
| **10** 🔬 | [MCP Foundations](modules/module-10-mcp-foundations.md) | MCP clients and servers, finding and installing an MCP server, AWS Docs and Hacker News MCP workflows |
| **11** | [Advanced Automation with Sub-Agents and Plugins](modules/module-11-automation-subagents-plugins.md) | Hooks, plugins, and automation agents that run work in the background |
| **12** 🔬 | [Advanced Project: URL Shortener](modules/module-12-url-shortener-project.md) | Architecture and setup for a URL shortener project, plus the assignment files, README, and how to submit |
| **13** 🔬 | [Prompting Strategies and Multi-Agent Architecture](modules/module-13-prompting-strategies-multiagent.md) | Defining agent roles, persona prompting and parallel sub-agents, and managing context with `/compact` |

### Phase 4 · Advanced → Hero

> Ship the project for real, then take stock of where this is going.

| # | Module | What you'll learn |
|---|---|---|
| **14** 🔬 | [CI/CD and Reliability](modules/module-14-cicd-and-reliability.md) | Wiring up a GitHub repository and automated checks, a production database and Vercel deployment, and core CI/CD and SRE vocabulary (SLI, SLO, SLA) |
| **15** | [Beyond Claude Code: Careers, Tools and What's Next](modules/module-15-beyond-claude-code.md) | Turning the project into portfolio evidence, and the human role in an agentic workflow |

🔬 = **deep-dive module** — longer, with hands-on exercises, common pitfalls, and key takeaways per chapter.

**Everyone takes all 15 modules in order.** Modules 7 through 9 build one portfolio project; Modules 12 through 14 build and ship a second, larger one — a URL shortener — so by the end of the course you have two working, deployed pieces of evidence, not just notes.

---

## Reference

Six ways to find something without reading a whole module:

| | |
|---|---|
| [**Topic index**](reference/topics.md) | A–Z concepts → the module that covers them, plus a "I want to…" table |
| [**Command reference**](reference/commands.md) | Every slash command, CLI flag, and environment variable in the course |
| [**Quick reference**](reference/quick-reference.md) | All 15 modules condensed to bullets |
| [**Troubleshooting**](reference/troubleshooting.md) | Symptom → cause → fix. Install, login, config not loading, performance |
| [**Power moves**](reference/power-moves.md) | Compound techniques that combine several modules, plus the anti-patterns |
| [**Syllabus**](reference/syllabus.md) | Full chapter-level outline of all 15 modules |

---

## How to use this

- **Read in order the first time.** Modules build on each other — Module 8 assumes the project you scoped in Module 7, and Module 14 assumes the project you built in Module 12.
- **Every module ends with links** to the previous and next one, so you can read straight through.
- **"Try it yourself" blocks assume a practice repo.** See [`labs/`](labs/README.md) for what it needs to contain — or use any project of your own with a failing test in it.
- **Claude Code ships weekly.** Each module carries a `last_verified` date in an HTML comment at the top. Anything version-specific — model IDs, prices, flags, command names — is worth a re-check before you rely on it for something critical. [Module 15](modules/module-15-beyond-claude-code.md) covers how to stay current.

---

## Repository map

```
modules/        The course. Fifteen files, one per module — this is the content
reference/      Topic index, commands, quick reference, troubleshooting, power moves, syllabus
labs/           Practice repo specification for the hands-on exercises
CLAUDE.md       Instructions for Claude Code when working in this repo
```

---

## Contributing

Corrections are welcome, especially **version drift** — a command that was renamed, a flag that changed, a limit that moved. That's the most valuable kind of PR this repo can get.

Before opening one that touches module content:

1. **Verify against official docs.** Start at [`code.claude.com/docs/llms.txt`](https://code.claude.com/docs/llms.txt) for the index and [`whats-new`](https://code.claude.com/docs/en/whats-new) for recent changes. Third-party recap posts have been unreliable.
2. **Update the `last_verified` date** in the module's HTML comment.
3. **Keep the references in sync** — a change to a module usually needs the same change in `reference/quick-reference.md`, and sometimes in `reference/commands.md`.
4. **Don't state what you can't verify.** If a fact isn't in the official docs, cut it rather than shipping it.

See [`CLAUDE.md`](CLAUDE.md) for the full content conventions.

---

## License

[MIT](LICENSE) © TrainWithShubham. Use it, teach from it, adapt it — attribution appreciated.
