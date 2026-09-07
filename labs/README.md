# Lab: Build a Production-Ready URL Shortener with Claude Code

## What you're doing

You've been given a spec (`CLAUDE.md`) and three subagent definitions (`.claude/agents/`) for a URL shortener — not the code. Your job is to get Claude Code to build it, using the agent workflow described in the spec, and end up with something that's actually production-minded: caching, rate limiting, input safety, atomic counters, the works.

You have two options:

- **Path A — Build it as specified.** Follow `CLAUDE.md` exactly and produce a working URL shortener.
- **Path B — Take inspiration, build your own.** Use `CLAUDE.md` as a *template* for a different small app idea (a paste bin, a poll app, a bookmark manager — your call). Copy its structure — architecture, API contract, schema, security/reliability sections, agent workflow — and rewrite the content for your idea. The skill being tested is writing a spec Claude Code can build from and running the agent workflow, not this specific app.

Either way, the deliverable is a working repo plus a short writeup (see §5).

## 1. Prerequisites

- [Claude Code](https://claude.com/product/claude-code) installed and logged in
- A GitHub repo (empty, yours) and the [`gh` CLI](https://cli.github.com/) authenticated — `push-agent` uses it to open PRs
- Python 3.11+, Node 20+
- Docker (for local Postgres + Redis) — or install both natively if you prefer

## 2. Setup

```bash
# 1. Create and clone your repo
gh repo create url-shortener --private --clone
cd url-shortener

# 2. Copy in the two files/folders you were given
#    CLAUDE.md            → repo root
#    .claude/agents/*.md  → repo root, under .claude/agents/

# 3. Start Claude Code from the repo root
claude
```

Claude Code reads `CLAUDE.md` and everything in `.claude/agents/` automatically the moment you start a session in this folder — you don't need to tell it those files exist.

## 3. Building it — suggested order

Don't ask Claude Code to "build the whole thing" in one shot. Work through it in slices, and run the agent workflow (§4) after each one:

1. Scaffold the repo structure from `CLAUDE.md` §4, plus `docker-compose.yml` for local Postgres + Redis.
2. Database schema + Alembic migration (§7).
3. `POST /api/v1/shorten` — validation, safety checks, idempotency (§6).
4. `GET /{short_code}` — cache-aside redirect + atomic click tracking (§6).
5. `GET /api/v1/urls/{short_code}` — stats.
6. Rate limiting, CORS, `/api/health` (§9–10).
7. Frontend — a form that hits `/shorten` and shows the result.
8. Ops script for expired-link cleanup (§11).

After each slice, run the agent workflow before moving to the next one — don't let five features pile up untested.

## 4. The agent workflow — how to actually use it

`CLAUDE.md` §8 defines the order: **implement → Testing Agent → Verification Agent → Push Agent.** In Claude Code, invoke a subagent by name in plain language:

```
Use the testing-agent to write and run tests for what we just built
```

Read its verdict before moving on. If it fails, fix the implementation and rerun it — don't skip ahead.

```
Use the verification-agent to review this against CLAUDE.md
```

Same deal: if `VERDICT: FAIL`, fix the listed issues first.

```
Use the push-agent to commit and open a PR
```

`push-agent` will refuse if you haven't told it the other two passed — that's intentional, read `push-agent.md` to see why. It opens a PR, it doesn't merge — that's still your call, do it from GitHub once you're happy.

This isn't busywork: the whole point of the exercise is experiencing why a "just push it" workflow breaks down, and how naming the checks as agents makes them a repeatable habit instead of something you remember to do only when you feel like it.

## 5. What "production ready" means here

Don't just get the happy path working. `CLAUDE.md` calls out specific things a rushed implementation gets wrong — re-read these before you consider a feature done:

- §6 — idempotency rule, URL safety validation (this is an SSRF guard, not busywork)
- §6 — atomic click-count updates, not read-then-write
- §9–10 — health check that actually checks its dependencies, Redis-backed rate limiting (not in-memory), CORS locked to your frontend's origin
- §12 — known limitations are *stated*, not hidden. If you cut a corner, write it down instead of leaving it for someone else to discover.

If you finish early: §13 lists stretch goals (CI pipeline, hook-enforced subagent permissions) that turn the workflow in §4 from "discipline" into something that can't be skipped.

## 6. Submitting

Share your repo link (or your PR link if your instructor wants to see the workflow in action, not just the final code). If you took Path B, include a one-line note on what you built instead and why.

## Troubleshooting

- **Claude Code doesn't seem to know about `CLAUDE.md` or the agents** — make sure you started `claude` from the repo root, and that `.claude/agents/*.md` weren't renamed.
- **`push-agent` can't open a PR** — run `gh auth login` and confirm `gh repo view` works from inside the repo.
- **Postgres/Redis connection errors** — check `docker compose ps`, and that your `.env` matches `.env.example`.
