# Claude Code: Zero To Hero

Learn Claude Code end to end — from your first session to building products on the Agent SDK. Written for developers, DevOps, cloud, SRE, and platform engineers who want to actually use it, not just watch a demo.

Sixteen modules across four phases, with a two-track capstone. Every module is verified against Anthropic's official documentation and carries the date it was last checked.

**Course:** [trainwithshubham.ai](https://trainwithshubham.ai) · **Language:** English · **Format:** Self-paced

---

## Start here

| If you… | Go to |
|---|---|
| Have never used Claude Code | [Module 1 — Welcome to the Era of Agentic Coding](docs/modules/module-01-agentic-era.md) |
| Have it installed and want to go deeper | [Module 4 — The Agentic Loop and Built-In Tools](docs/modules/module-04-agentic-loop-tools.md) |
| Run infrastructure and want the DevOps path | [Module 13 — Infrastructure, Cloud, and Enterprise](docs/modules/module-13-infrastructure-cloud-enterprise.md) |
| Just want a lookup table | [Quick reference](docs/quick-reference.md) |

---

## The course

### Phase 1 — Zero → Comfortable

| # | Module | What you'll learn |
|---|---|---|
| 1 | [Welcome to the Era of Agentic Coding](docs/modules/module-01-agentic-era.md) | What an agentic loop is, the 2026 model lineup, and which Claude surface to use |
| 2 | [Prompt Engineering for Agentic Work](docs/modules/module-02-prompt-engineering.md) | Writing for an agent instead of a chatbot, and where instructions actually live |
| 3 | [Installing Claude Code Everywhere](docs/modules/module-03-installing-everywhere.md) | Install on any OS, every platform compared, auth, and the interface |

### Phase 2 — Comfortable → Productive

| # | Module | What you'll learn |
|---|---|---|
| 4 | [The Agentic Loop and Built-In Tools](docs/modules/module-04-agentic-loop-tools.md) | Read, Write, Edit, Grep, Glob, Bash, sandboxing, and the context window |
| 5 | [Permissions, Memory, and Configuration](docs/modules/module-05-permissions-memory-config.md) | Permission modes, auto mode, CLAUDE.md, auto memory, and settings precedence |
| 6 | [Planning and Executing a Real Project](docs/modules/module-06-planning-real-projects.md) | Plan mode, sessions and branching, checkpoints, and monorepos |
| 7 | [Skills, Commands, and Hooks](docs/modules/module-07-skills-commands-hooks.md) | Extending Claude Code — and the one layer that's genuinely enforced |
| 8 | [The Plugin Ecosystem](docs/modules/module-08-plugin-ecosystem.md) | Installing plugins, building a marketplace, and the two security plugins |
| 9 | [MCP in 2026](docs/modules/module-09-mcp.md) | Connecting external tools, tool search, and channels |

### Phase 3 — Productive → Advanced

| # | Module | What you'll learn |
|---|---|---|
| 10 | [Subagents and Orchestration at Scale](docs/modules/module-10-subagents-orchestration.md) | Subagents, agent teams, dynamic workflows, worktrees, cross-session messaging |
| 11 | [Claude Code Across Every Surface](docs/modules/module-11-every-surface.md) | Desktop, web, routines, mobile, Remote Control, Slack, and Chrome |
| 12 | [CI/CD, Code Review, and Security](docs/modules/module-12-cicd-review-security.md) | GitHub Actions, the review ladder, and the security story end to end |
| 13 | [Infrastructure, Cloud, and Enterprise](docs/modules/module-13-infrastructure-cloud-enterprise.md) | IaC, third-party providers, self-hosted environments, gateway, admin, observability |

### Phase 4 — Advanced → Hero

| # | Module | What you'll learn |
|---|---|---|
| 14 | [Building Products with the Agent SDK](docs/modules/module-14-agent-sdk.md) | The SDK, the message lifecycle, and hosting agents in production |
| 15 | [Capstone: Choose Your Track](docs/modules/module-15-capstone.md) | Ship a full-stack feature, or build a self-healing ops agent |
| 16 | [Staying Sharp: What's Next](docs/modules/module-16-staying-sharp.md) | Reading the weekly changelog and maintaining your setup |

Everyone takes Modules 1–14. **Module 15 splits into two tracks** so a frontend developer and an SRE each leave with a portfolio piece that matches their job. **Module 13 is not optional on either track** — it's what makes this different from a generic AI coding tutorial.

---

## How to use this

- **Every module stands alone**, but they build on each other. Read in order the first time.
- **"Try it yourself" blocks assume a practice repo.** See [`labs/`](labs/README.md) for what it needs to contain.
- **Modules 4, 5, 7, 9, 10, and 14 are deep-dives** — longer, with hands-on exercises and common pitfalls per chapter. The rest are denser reference notes.
- **Claude Code ships weekly.** Each module carries a `last_verified` date in an HTML comment at the top. Anything version-specific — model IDs, prices, flags, command names — is worth a re-check before you rely on it for something critical.

---

## Repository map

| Path | What's in it |
|---|---|
| [`docs/modules/`](docs/modules/) | **The course.** Sixteen module files — this is the content |
| [`docs/syllabus.md`](docs/syllabus.md) | The canonical 16-module, 4-phase structure with chapter breakdowns |
| [`docs/quick-reference.md`](docs/quick-reference.md) | Condensed lookup across all modules, for when you know what you're after |
| [`labs/`](labs/) | Practice repo specification for the hands-on exercises |
| [`CLAUDE.md`](CLAUDE.md) | Instructions for Claude Code when working in this repo |

---

## Contributing

Corrections are welcome, especially version drift — a command that was renamed, a flag that changed, a limit that moved.

Before opening a PR that touches module content:

1. **Verify against official docs.** Start at [`code.claude.com/docs/llms.txt`](https://code.claude.com/docs/llms.txt) for the index and [`whats-new`](https://code.claude.com/docs/en/whats-new) for recent changes. Third-party recap posts have been unreliable.
2. **Update the `last_verified` date** in the module's HTML comment.
3. **Keep the three sources consistent** — a change to a module usually needs the same change in `docs/quick-reference.md` and sometimes `docs/syllabus.md`.
4. **No instructor notes in learner files.** Those go in `docs/instructor/`.

See [`CLAUDE.md`](CLAUDE.md) for the full content conventions.

## License

[MIT](LICENSE) © TrainWithShubham. Use it, teach from it, adapt it — attribution appreciated.
