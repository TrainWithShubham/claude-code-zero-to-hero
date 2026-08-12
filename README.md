# Claude Code: Zero To Hero

Learn Claude Code end to end — from your first session to building products on the Agent SDK. Written for developers, DevOps, cloud, SRE, and platform engineers who want to actually use it, not just watch a demo.

Sixteen modules, four phases, a two-track capstone. Every fact is verified against Anthropic's official documentation, and every module carries the date it was last checked.

**Course:** [trainwithshubham.ai](https://trainwithshubham.ai) · **Language:** English · **Format:** Self-paced · **Verified:** 2026-08-12

---

## Jump in

| | |
|---|---|
| 🚀 **New to Claude Code** | Start at [Module 1](modules/module-01-agentic-era.md) and read in order |
| ⚡ **Already installed** | Skip to [Module 4 — the loop and the tools](modules/module-04-agentic-loop-tools.md) |
| 🔧 **Here for DevOps/SRE** | [Module 13](modules/module-13-infrastructure-cloud-enterprise.md) is the one written for you |
| 🔍 **Looking for one thing** | [Topic index](reference/topics.md) · [Command reference](reference/commands.md) |
| 📋 **Want the short version** | [Quick reference](reference/quick-reference.md) — all 16 modules condensed |
| 🩹 **Something's broken** | [Troubleshooting](reference/troubleshooting.md) — symptom → cause → fix |
| ⚡ **Already good, want sharper** | [Power moves](reference/power-moves.md) — the compound techniques |

---

## The course

### Phase 1 · Zero → Comfortable

> Understand what you're using, and get it running.

| # | Module | What you'll learn |
|---|---|---|
| **01** | [Welcome to the Era of Agentic Coding](modules/module-01-agentic-era.md) | What an agentic loop is, the 2026 model lineup, and which Claude surface to use |
| **02** | [Prompt Engineering for Agentic Work](modules/module-02-prompt-engineering.md) | Writing for an agent instead of a chatbot, and where instructions actually live |
| **03** | [Installing Claude Code Everywhere](modules/module-03-installing-everywhere.md) | Install on any OS, every platform compared, authentication, and the interface |

### Phase 2 · Comfortable → Productive

> The tools, the guardrails, and how to extend both.

| # | Module | What you'll learn |
|---|---|---|
| **04** 🔬 | [The Agentic Loop and Built-In Tools](modules/module-04-agentic-loop-tools.md) | Read, Write, Edit, Grep, Glob, Bash, sandboxing, and the context window |
| **05** 🔬 | [Permissions, Memory, and Configuration](modules/module-05-permissions-memory-config.md) | Permission modes, auto mode, CLAUDE.md, auto memory, settings precedence |
| **06** | [Planning and Executing a Real Project](modules/module-06-planning-real-projects.md) | Plan mode, sessions and branching, checkpoints, and monorepos |
| **07** 🔬 | [Skills, Commands, and Hooks](modules/module-07-skills-commands-hooks.md) | Extending Claude Code — and the one layer that's genuinely enforced |
| **08** | [The Plugin Ecosystem](modules/module-08-plugin-ecosystem.md) | Installing plugins, building a marketplace, and the two security plugins |
| **09** 🔬 | [MCP in 2026](modules/module-09-mcp.md) | Connecting external tools, tool search, and channels |

### Phase 3 · Productive → Advanced

> Many agents, every surface, and the boundaries an organisation sets.

| # | Module | What you'll learn |
|---|---|---|
| **10** 🔬 | [Subagents and Orchestration at Scale](modules/module-10-subagents-orchestration.md) | Subagents, agent teams, dynamic workflows, worktrees, cross-session messaging |
| **11** | [Claude Code Across Every Surface](modules/module-11-every-surface.md) | VS Code, JetBrains, Desktop, web, routines, mobile, Remote Control, Slack, Chrome |
| **12** | [CI/CD, Code Review, and Security](modules/module-12-cicd-review-security.md) | Headless mode, GitHub Actions, the review ladder, and the security story end to end |
| **13** | [Infrastructure, Cloud, and Enterprise](modules/module-13-infrastructure-cloud-enterprise.md) | IaC, third-party providers, self-hosted environments, gateway, admin, observability |

### Phase 4 · Advanced → Hero

> Build with it, then build something with it.

| # | Module | What you'll learn |
|---|---|---|
| **14** 🔬 | [Building Products with the Agent SDK](modules/module-14-agent-sdk.md) | The SDK, the message lifecycle, and hosting agents in production |
| **15** | [Capstone: Choose Your Track](modules/module-15-capstone.md) | Ship a full-stack feature, or build a self-healing ops agent |
| **16** | [Staying Sharp: What's Next](modules/module-16-staying-sharp.md) | Reading the weekly changelog and maintaining your setup |

🔬 = **deep-dive module** — longer, with hands-on exercises, common pitfalls, and key takeaways per chapter.

**Everyone takes Modules 1–14.** Module 15 splits into two tracks so a frontend developer and an SRE each leave with a portfolio piece that matches their job. **Module 13 is not optional on either track** — it's what makes this different from a generic AI coding tutorial.

---

## Reference

Six ways to find something without reading a whole module:

| | |
|---|---|
| [**Topic index**](reference/topics.md) | A–Z concepts → the module that covers them, plus a "I want to…" table |
| [**Command reference**](reference/commands.md) | Every slash command, CLI flag, and environment variable in the course |
| [**Quick reference**](reference/quick-reference.md) | All 16 modules condensed to bullets |
| [**Troubleshooting**](reference/troubleshooting.md) | Symptom → cause → fix. Install, login, config not loading, performance |
| [**Power moves**](reference/power-moves.md) | Compound techniques that combine several modules, plus the anti-patterns |
| [**Syllabus**](reference/syllabus.md) | Full chapter-level outline of all 16 modules |

---

## How to use this

- **Read in order the first time.** Modules build on each other — Module 7 makes much more sense after Module 5.
- **Every module ends with links** to the previous and next one, so you can read straight through.
- **"Try it yourself" blocks assume a practice repo.** See [`labs/`](labs/README.md) for what it needs to contain — or use any project of your own with a failing test in it.
- **Claude Code ships weekly.** Each module carries a `last_verified` date in an HTML comment at the top. Anything version-specific — model IDs, prices, flags, command names — is worth a re-check before you rely on it for something critical. [Module 16](modules/module-16-staying-sharp.md) covers how.

---

## Repository map

```
modules/        The course. Sixteen files, one per module — this is the content
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
