<!-- module: 1 | phase: 1 | last_verified: 2026-08-12 -->

# Module 1 — Welcome to the Era of Agentic Coding

## 1. What Generative AI and an LLM actually are

- An LLM is a model trained on an enormous amount of text to predict what comes next. That's the whole mechanism.
- "Generative" means it produces new content rather than sorting existing content into buckets — the difference between writing you a function and telling you whether an email is spam.
- **An LLM by itself cannot do anything.** It reads text and emits text. It can't open your file, run your test, or check whether its own suggestion compiles.
- That's why pasting code into a chat window sometimes produces a confident fix referencing a function you deleted last month. The model isn't being careless — it has no way to look.
- Give the same model tools — read a file, run a command, search a directory — and it stops guessing and starts checking. That jump is what this course is about.

**Hold onto this:** Claude Code is an LLM, plus tools, plus a loop. Every confusing thing about it becomes obvious once you see it that way.

---

## 2. Claude's model lineup in 2026

| Model | Model ID | Context window | API price (in / out per MTok) |
|---|---|---|---|
| Claude Fable 5 | `claude-fable-5` | 1M | $10 / $50 |
| Claude Mythos 5 | `claude-mythos-5` | 1M | $10 / $50 |
| Claude Opus 5 | `claude-opus-5` | 1M | $5 / $25 |
| Claude Sonnet 5 | `claude-sonnet-5` | 1M | $3 / $15 |
| Claude Haiku 4.5 | `claude-haiku-4-5` | 200K | $1 / $5 |

- **Context windows are not uniform.** Everything in the 5 generation gives you a 1M-token context window by default. Haiku 4.5 gives you 200K. That 5× gap is why Haiku is the wrong pick for "read my whole monorepo" work no matter how cheap it looks.
- **Fable 5 is the top tier, not Opus.** Claude Fable 5 is the most capable widely released model; Opus 5 sits below it at half the price. Claude Mythos 5 matches Fable's capabilities, pricing, and API behavior but is only reachable through Project Glasswing.
- **Fable 5 carries stricter safety classifiers** and is explicitly not intended for research biology or most cybersecurity work — requests in those areas can come back refused. Worth knowing if your work is security-adjacent.
- **Sonnet 5** is the default model on Pro, Team Standard, and Enterprise seats, with adaptive thinking on by default.
- **The prices above are API prices.** On a Pro or Max subscription you never see them. They start mattering when you build with the Agent SDK (Module 14) or run Claude Code through Bedrock. Learn the ratios, not the dollars — Haiku is roughly 5× cheaper than Opus on both input and output.

**Rule of thumb:** pick by task difficulty and budget, not by "biggest is always best." Haiku for high-volume simple work, Sonnet for daily driving, Opus for genuinely hard problems. There's a second dial beyond model choice — effort levels — covered in Module 12.

---

## 3. Claude.ai vs Claude API vs Claude Code vs Claude Cowork vs Agent SDK

Five products with "Claude" in the name is genuinely confusing. Cut it with two questions: **who's the user, and what does it act on?**

| Surface | User | Acts on |
|---|---|---|
| **Claude.ai** | A human, conversationally | Nothing — it talks |
| **Claude API / Platform** | Your code | Whatever you build |
| **Claude Code** | A developer or engineer | Your codebase, terminal, cloud |
| **Claude Cowork** | A non-developer | Folders, docs, decks, spreadsheets |
| **Agent SDK** | Your product's code | Whatever you give it |
| **Claude Tag** | Your team, in Slack | Whatever the tagged task needs |

- The one people get wrong: **Claude Code and the Agent SDK are the same engine.** The SDK is Claude Code's own agent loop packaged as a Python/TypeScript library so you can build your own product on it.
- You're not choosing between "Claude Code features" and "SDK features" — the SDK ships subagents, skills, plugins, and hooks too. Module 14 covers it.

---

## 4. What makes an AI system "agentic"

> **Agentic = a loop of gather context → take action → verify results, repeating until done.**

- It's not "has tools." Tools alone give you a function-caller.
- What makes it agentic is that **each result feeds the next decision.** The model reads a test failure, and that failure determines what it edits next. Nobody scripted that branch.
- Contrast with a **workflow**: a deterministic script — step 1, step 2, step 3, same path every time, no judgment in the loop.
- Claude Code gives you both. The interactive loop is agentic; Dynamic Workflows (Module 10) let you script the orchestration when you want the *process* itself to be repeatable.

This is the single most important concept in the course. Every later module either extends this loop (tools, skills, MCP), constrains it (permissions, hooks, sandboxes), or runs many of them at once (subagents, agent teams).

---

## 5. Where Claude Code fits — for DevOps, Cloud, SRE, and Platform engineers

Most Claude Code material assumes you're building a web app. Here's the reframe for infrastructure work.

Go back to the loop: gather → act → **verify**. For application code, the verify step is usually the test suite. For infrastructure, you already have equivalents — you just may not think of them that way:

- `terraform plan` is a verify signal
- `kubectl diff` is a verify signal
- `ansible --check` is a verify signal
- A failing health check after a rollout is a verify signal

**Claude Code works well on infrastructure precisely when you give it one of these to check itself against.** Point it at a Terraform module with no way to run `plan` and it's guessing from syntax. Give it `plan` output and it closes the loop the same way it does with a failing unit test.

That's the framing for the whole DevOps track: your job isn't to describe the change perfectly up front — it's to make sure the agent has a way to tell whether it worked.

It sits alongside your terminal, IDE, CI system, and cloud console. It doesn't replace them; it drives them.

---

## 6. Course roadmap and the two capstone tracks

| Phase | Modules | Arc |
|---|---|---|
| Zero → Comfortable | 1–3 | Fundamentals, prompting for agents, install everywhere, first session |
| Comfortable → Productive | 4–9 | The agentic loop and built-in tools, permissions & config, real projects, Skills/Hooks/Plugins, MCP |
| Productive → Advanced | 10–13 | Multi-agent orchestration, every surface, CI/CD & security, infra/cloud/enterprise |
| Advanced → Hero | 14–16 | Agent SDK, capstone, staying current |

- Everyone takes Modules 1–14.
- **Module 15 splits into two tracks.** Track A ships a full-stack SaaS feature end to end. Track B builds a self-healing CI/CD or incident-response agent. Two tracks, so a frontend developer and an SRE each walk out with a portfolio piece that matches the job they actually hold.
- **Module 13 is not optional, even on Track A.** It's the module that makes this different from every other Claude Code tutorial.

---
<!-- nav -->

_Start of course_ · [All modules](../README.md#the-course) · [Prompt Engineering for Agentic Work →](module-02-prompt-engineering.md)
