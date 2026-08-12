<!-- module: 16 | phase: 4 | last_verified: 2026-08-12 -->

# Module 16 — Staying Sharp: What's Next

## Start with a real example

Earlier in this course, the note on session management said `/fork` copies your conversation into a new background session while you keep working in the original.

That was **true when it was written.** The Week 29 digest announced exactly that.

Today the command is **`/branch`**, and the behavior is different: it copies the conversation and **switches you into the copy**, leaving the original intact. The `--fork-session` flag survived; the slash command didn't.

Nothing was wrong. It aged, in under a month.

That's what this module is about. Not "check the docs sometimes" — the concrete fact that a correct note becomes a wrong note on a weekly cadence, and what to do about it.

---

## 1. Reading the weekly changelog

Two sources, doing different jobs:

| Source | What it's for |
|---|---|
| [`code.claude.com/docs/en/whats-new`](https://code.claude.com/docs/en/whats-new) | The **weekly digest** — features most likely to change how you work, with runnable code, a short demo, and a link to full docs |
| The full changelog | Every bug fix and minor improvement |

The digest is **one page per week** — `/whats-new/2026-w32` — tagged with the version range it covers, like `v2.1.220–v2.1.224`. Each entry leads with one headline feature and follows with an "also this week" paragraph covering three or four more.

### The habit, stated precisely

Skim one page a week, and **check the digest's version tag against your own `claude --version`.**

That second half matters. If you installed through Homebrew, WinGet, apt, dnf, or apk, **your install doesn't auto-update** — you may be several digests behind, reading about features you don't have. Native installs update in the background. This is the Module 3 decision coming due.

### When something in your setup breaks

Work in this order:

1. `claude --version` — are you on the version the docs describe?
2. `claude doctor` — install and settings diagnostics without starting a session
3. `/context` — did the file you're debugging actually load?
4. The week's digest, then the changelog for bug-level detail

---

## 2. Claude's expanding surfaces — where this is headed

Rather than guessing, read the trajectory out of the digests themselves. Two patterns are visible.

### Features graduate on a predictable path

Auto mode is the clearest case:

| Week | Milestone |
|---|---|
| 13 (March) | Lands in research preview |
| 21 (May) | Available on the Pro plan |
| 23 (June) | Available on Bedrock, Google Cloud, and Microsoft Foundry |
| 32 (August) | **Becomes the default** permission mode for new Pro, Max, and Team sessions |

Research preview → wider plans → third-party providers → default.

That pattern is useful because it lets you **predict**. Channels and self-hosted environments are in preview today. Dynamic workflows moved from a Week 22 announcement to being available on every paid plan. If a preview feature fits your work, it's worth building familiarity before it becomes the default.

### The surface area keeps widening

CLI → IDE extensions → Desktop → Web → Mobile → Slack → Chrome → Agent SDK, with cross-session messaging added in Week 32. Each of these was a digest entry, not a rewrite of the product.

**Models move too.** Sonnet 5 arrived in Week 27, Opus 5 in Week 30. Anything version-specific — model IDs, context windows, prices, effort levels — has a shelf life measured in weeks.

Treat all of it as a trend, not a finish line.

---

## 3. Building your own learning loop

The useful version of this chapter isn't motivational. Everything you need, you already built earlier in the course.

### Automate the check

Set up a **routine** (Module 11) on a weekly schedule:

> Read the latest weekly digest at code.claude.com/docs/en/whats-new. For each change, check whether anything in my setup — CLAUDE.md, skills, hooks, plugins, or scripts — is now outdated. Report what changed and what I should update. If nothing is affected, say so.

Docs drift is a canonical routine use case, so this runs with the grain of the tool.

### Prune what you've stopped using

Your setup accumulates. Four ways to find the dead weight:

| Tool | Finds |
|---|---|
| `/usage` | What's actually driving your plan limits, by skill, subagent, plugin, and MCP server |
| `/plugin` → **Installed** → **Not used recently** | Plugins untouched for two weeks across 10+ sessions, still costing startup and context |
| `OTEL_LOG_TOOL_DETAILS=1` + the `skill_activated` event | Skills nobody invokes |
| `/doctor` | Proposes trims for a bloated `CLAUDE.md` — cuts what Claude can derive from the codebase, keeps pitfalls and rationale |

### Re-tune after model releases

Prompts written for an older model can actively reduce output quality on a newer one. Step-by-step scaffolding, "think step by step," and workarounds for limitations that no longer exist all fall into this category.

The docs say it directly in the monorepo guide: **revisit your instructions after major model releases.** A rule that forced single-file refactors can be deleted once the limitation it worked around is gone.

### Stay connected

Watch plugin marketplaces for new capability you'd otherwise build yourself, follow official release announcements, and engage with the community building on the same tool.

**The framing to leave learners with:** your Claude Code setup is something you *maintain*, not something you configure once.

---

## 4. Course recap, certification, and next steps

### The arc in four sentences

| Phase | Modules | The idea |
|---|---|---|
| Zero → Comfortable | 1–3 | Claude Code is an LLM, plus tools, plus a loop |
| Comfortable → Productive | 4–9 | That loop is extended by tools and skills, and constrained by permissions and hooks |
| Productive → Advanced | 10–13 | The same loop, many at once, inside boundaries an organization sets |
| Advanced → Hero | 14–16 | The loop as a library you build products on |

### The three things worth keeping

If a learner forgets everything else, these three carry the most weight:

**1. Give it a way to verify.** Claude stops when the work looks done. Without a check it can run, "looks done" is the only signal available — and you become the verification loop. A test suite, a build exit code, `terraform plan`, a screenshot comparison. Anything that returns pass or fail.

**2. Context is the constraint.** Performance degrades as the context window fills. Almost every technique in this course — subagents, skills that load on demand, path-scoped rules, `/clear` between tasks, tool search — is really an answer to the same question: how do I get the right information in and keep the wrong information out?

**3. Guidance is not enforcement.** CLAUDE.md is advisory. Settings are enforced by the client. Hooks fire before any permission check, in every mode, including `bypassPermissions`. If a rule actually matters, it belongs in a hook — not a markdown file asking nicely.

### Where to go next

Point back to whichever capstone track you built and keep extending it. The two most useful directions:

- **Deepen the setup** — turn the workflows you repeat into skills, the rules you rely on into hooks, and the whole thing into a plugin your team can install
- **Build something on the SDK** — the Agent SDK is the same loop you've spent sixteen modules learning, in your own process

And keep the weekly habit. This course is a snapshot. Claude Code isn't.
