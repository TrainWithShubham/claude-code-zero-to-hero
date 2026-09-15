<!-- module: 12 | phase: 3 | format: deep-dive | last_verified: 2026-09-15 -->
# Module 12 — Advanced Project: URL Shortener

By the end of this module you can set up and build a URL shortener as a hands-on project using the provided architecture, files, and assignment brief.

This module is the capstone-style project for everything up to this point — planning, permissions, skills, sub-agents, hooks, and MCP all show up here at once, applied to one real piece of software instead of isolated examples. The full spec lives in the `labs/` directory of this repo (`labs/README.md` and `labs/CLAUDE.md`); this module walks through the architecture and the assignment mechanics so you understand what you're building and why, before you open that spec and start.

## URL Shortener Architecture and Project Setup

**Concept**

A URL shortener sounds trivial — take a long URL, hand back a short code, redirect anyone who visits the short code back to the original — but building one "production-ready" surfaces almost every reliability concern a real backend has to handle: caching, race conditions, abuse prevention, and observability. That's exactly why it's the project used here: it's small enough to finish in a few sessions, but it isn't a toy once you take the production requirements seriously.

The architecture has three moving pieces:

- **The API service.** A `POST /api/v1/shorten` endpoint takes a long URL and returns a short code, and a `GET /api/v1/urls/{short_code}` endpoint returns stats (click count, creation date) for a given code. This is where input validation lives — including a check that the submitted URL isn't pointing at an internal address, which is a real SSRF (server-side request forgery) risk, not a hypothetical one: a redirect service that will happily "visit" any URL it's given is a foothold for hitting internal infrastructure if you don't guard against it.
- **The redirect service.** A `GET /{short_code}` endpoint is the hot path — it's hit far more often than the shorten endpoint, so it's built around a cache-aside pattern (check a cache like Redis first, fall back to the database on a miss) and updates the click counter atomically rather than with a read-then-write, which would lose counts under concurrent traffic.
- **Storage.** A Postgres database holds the URL records; Redis backs both the redirect cache and rate limiting. The schema and its migration are defined with Alembic, so schema changes are tracked the same way code changes are.

Setting the project up starts with the spec, not the code:

1. Create an empty repository and clone it locally.
2. Copy `labs/README.md`'s referenced `CLAUDE.md` and `.claude/agents/*.md` into that new repo — at the repo root, and under `.claude/agents/` respectively. These aren't example files to read once; they're what Claude Code loads automatically the moment you start a session in that folder, so the project's rules and its three specialised agents (testing, verification, push) are live from the first prompt.
3. Start Claude Code from the repo root and scaffold the structure the spec describes — service layout, a `docker-compose.yml` for local Postgres and Redis — before writing any endpoint logic.

Build it in slices rather than asking for the whole system in one prompt: scaffold, then schema, then the shorten endpoint, then the redirect endpoint, then stats, then the operational pieces (rate limiting, CORS, health checks), then the frontend, then the cleanup script for expired links. Each slice should be small enough to actually review before you move to the next one — a plan you can't check is not a plan, it's a hope.

**Try it yourself**

1. Read `labs/README.md` in full before writing anything, and decide whether you're doing Path A (build the URL shortener exactly as specified) or Path B (use the same spec as a template for a different small app). Either is a valid way to practice the same skill: writing and following a spec Claude Code can build from.
2. Set up the repo and the `.claude/agents/` files exactly as described, then ask Claude Code to scaffold just the first slice — repo structure and `docker-compose.yml` — and review what it produces before asking for anything else.
3. Before writing the shorten endpoint, ask Claude Code to explain back to you, in its own words, why the SSRF check and the atomic click-count update matter. If it can't, that's a sign to slow down and re-read that section of the spec together.

**Common pitfalls**

- Asking for the whole system in one prompt. It's the fastest way to end up with a plausible-looking implementation that quietly skips the SSRF check or does read-then-write click counting — the failure modes this project is designed to teach you to catch.
- Treating the cache-aside pattern as optional because "the database is fast enough locally." The redirect endpoint is the one endpoint every visitor hits; building it without a cache means you're not practicing the pattern the project exists to teach.
- Skipping the migration step and hand-editing the schema. Alembic migrations are how a real team tracks schema history — get in the habit here, not just when a project forces it on you later.

**Key takeaways**

- The project has three parts — an API service, a redirect service, and Postgres/Redis storage — and each one maps to a specific production concern: input safety, cache-aside performance, and durable counting.
- Setup means copying the spec's `CLAUDE.md` and `.claude/agents/*.md` into your own repo before starting Claude Code, not reading them as reference material from this course repo.
- Build in slices and review each one — this project rewards discipline more than speed.

## Assignment Files, README and How to Submit

**Concept**

The assignment isn't graded on code alone — it's graded on whether you can show a working system and explain the decisions you made building it. That means the files you produce matter as much as the endpoints you write.

**The agent workflow is part of the deliverable, not a suggestion.** The spec defines an explicit order: implement a slice, run it past a **testing agent** (writes and runs tests for what you just built), then a **verification agent** (reviews the implementation against the spec and returns a clear pass/fail verdict), then a **push agent** (commits and opens a pull request — it does not merge, and it refuses to run until you've told it the other two agents passed). Naming these steps as agents, rather than trusting yourself to remember them, is the actual lesson: a "just push it" workflow breaks down under real deadline pressure, and a workflow that names its own checks as agents can't quietly skip a step the way a mental checklist can.

**What to submit:**

- A repository link — or a pull request link if your instructor wants to see the agent workflow in action rather than just the final state of the code.
- If you took Path B (a different small app built from the same spec template), a one-line note on what you built instead and why. This isn't a formality: it's evidence you understood the spec well enough to adapt it, not just follow it.
- A short writeup of known limitations. The spec is explicit on this point — if you cut a corner (skipped a stretch goal, left rate limiting basic, didn't wire up the cleanup script as a cron job), write it down rather than leaving it for someone reviewing your code to discover. A stated limitation reads as engineering judgment; a hidden one reads as an oversight.

**Writing the README:** treat it the way you'd treat a README for any real project — what the service does, how to run it locally (the `docker compose` commands, environment variables, how to run migrations), and what's deliberately out of scope. A learner reading your README a week later, without your memory of the session, should be able to get the project running from it alone. That's the same bar this course's own module files are held to, and it's not a coincidence — writing for someone who wasn't in the room is a skill this whole course has been building toward.

**If you finish early**, the spec lists stretch goals — a CI pipeline, and hook-enforced sub-agent permissions. That second one connects directly back to Module 11: instead of trusting that the push agent will always check the other two agents' verdicts, a hook can enforce it structurally, so the rule can't be skipped even by accident.

**Try it yourself**

1. Before you write a line of code, draft the shape of your README — headings only. Filling it in as you go is far easier than reconstructing setup steps from memory afterward.
2. After your first passing slice, run the full sequence — testing agent, verification agent, push agent — even if the slice feels too small to bother. Practicing the sequence on a small, low-stakes change is exactly when you want to find out it's confusing.
3. When you're done, read back your own known-limitations section as if you were the instructor grading it. If something reads as vague ("some edge cases might not be handled"), make it specific ("expired-link cleanup isn't wired to a scheduler — it's a manual script").

**Common pitfalls**

- Running the push agent before the other two have actually passed, or arguing with it when it refuses. That refusal is the whole point — it's the automation equivalent of the discipline this project is trying to build.
- A README that only says "run `docker compose up`" with no mention of environment variables or migrations. If you needed a `.env` file to get it running, your reader will too.
- Treating the known-limitations section as an admission of failure rather than a normal part of engineering communication. Every real system has documented limitations; the absence of one is usually a sign nobody looked closely enough.

**Key takeaways**

- Submission is the repo (or PR) link, a Path B note if applicable, and a written list of known limitations — not just working code.
- The testing agent → verification agent → push agent sequence from the spec is the deliverable's backbone; skipping it defeats the purpose of the exercise even if the final code happens to work.
- Write the README for a reader who wasn't in your session — the same standard this course holds its own modules to.

<!-- nav -->
[← Module 11: Advanced Automation with Sub-Agents and Plugins](module-11-automation-subagents-plugins.md) | [Module 13: Prompting Strategies and Multi-Agent Architecture →](module-13-prompting-strategies-multiagent.md)
