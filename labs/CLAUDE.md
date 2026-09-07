# URL Shortener — Project Guide

Source of truth for Claude Code on this repo. Read before making changes. **(assumed)** marks a decision made without explicit input — flag if wrong, or make your own call and document it here.

## 1. What this is

A full-stack URL shortener: submit a long URL, get back a short code that redirects to it, with click analytics, built to survive more than one concurrent user — not a toy CRUD app.

## 2. Tech stack

| Layer | Choice |
|---|---|
| Backend | Python 3.11+, FastAPI, fully **async** (no sync DB calls in async routes) |
| Database | PostgreSQL, via SQLAlchemy async engine + `asyncpg` |
| Cache / rate-limit store | Redis — cache-aside for redirects, shared counter store for rate limiting |
| Frontend | React (Vite) |
| Migrations | Alembic |

## 3. Architecture

```
React frontend ──HTTP──▶ FastAPI backend ──▶ Redis (cache, rate limit)
                               │
                               ▼
                          PostgreSQL

Dev workflow:
  implement → Testing Agent → Verification Agent → Push Agent
```

`.claude/agents/` defines the three agents referenced above — see §8.

## 4. Repo structure

```
url-shortener/
├── CLAUDE.md
├── .claude/agents/                # testing-agent.md, verification-agent.md, push-agent.md
├── backend/
│   ├── app/
│   │   ├── main.py                # app factory, CORS, request-ID middleware
│   │   ├── config.py              # env var loading (pydantic-settings)
│   │   ├── database.py            # async engine/session
│   │   ├── cache.py               # Redis client
│   │   ├── models.py
│   │   ├── schemas.py
│   │   ├── crud.py
│   │   ├── security.py            # URL safety / SSRF checks, reserved aliases
│   │   ├── logging_config.py
│   │   └── routers/
│   │       ├── shorten.py         # /api/v1/shorten
│   │       ├── redirect.py        # /{short_code}
│   │       └── stats.py           # /api/v1/urls/{short_code}
│   ├── scripts/cleanup_expired.py # cron/systemd-timer job
│   ├── tests/
│   ├── alembic/
│   ├── requirements.txt
│   ├── pyproject.toml             # ruff + mypy config
│   └── .env.example
├── frontend/
│   ├── src/{App.jsx, components/, pages/Home.jsx, api/client.js}
│   ├── package.json
│   └── .env.example
└── docker-compose.yml             # local Postgres + Redis for dev
```

## 5. Core features (MVP)

- Shorten a URL (optional custom alias, optional expiry)
- Redirect a short code to its original URL — cached, atomic click tracking
- View stats for a short code
- Rate-limited, abuse-resistant shorten endpoint
- Reject invalid/unsafe URLs, duplicate custom aliases, reserved words

**Out of scope for MVP (assumed):** auth/user accounts, link editing, QR codes, per-click history (see §12).

## 6. API contract (`/api/v1` prefix)

| Method | Path | Description |
|---|---|---|
| `POST` | `/api/v1/shorten` | Create (or dedupe-return) a short URL |
| `GET` | `/{short_code}` | 302 redirect — unversioned, this is the public-facing URL |
| `GET` | `/api/v1/urls/{short_code}` | Metadata + click stats |
| `GET` | `/api/health` | Liveness + dependency check |

**POST /api/v1/shorten**
```json
// request
{ "url": "https://example.com/very/long/path", "custom_alias": "promo26", "expires_in_days": 30 }
// response 201 (new) or 200 (deduped existing non-alias URL — see idempotency rule below)
{ "short_code": "promo26", "short_url": "https://sho.rt/promo26", "original_url": "...", "expires_at": "2026-10-07T00:00:00Z" }
```

Validation, in order:
1. Scheme must be `http`/`https` only — reject `javascript:`, `data:`, `file:`, etc.
2. Length ≤ 2048 chars.
3. Resolve hostname; reject loopback, link-local, and RFC1918 private ranges, and bare IP literals pointing at the metadata endpoint (`169.254.169.254`) — this is an SSRF/phishing guard, implement in `security.py`, never trust client-supplied hostnames blindly.
4. `custom_alias`, if given, must not be in the reserved-word set: `api, health, admin, static, docs, redoc, openapi, v1, www, assets, favicon.ico`.

Errors: `400` invalid/unsafe URL · `409` alias taken or reserved · `429` rate limited.

**Idempotency rule (explicit, don't leave this undefined):** if `custom_alias` is omitted and a non-expired row already exists for the exact same `original_url` with `is_custom_alias = false`, return that existing short code with `200` instead of creating a duplicate row. Requests with a `custom_alias` always attempt a fresh insert and `409` on conflict — aliases are never deduped against.

**GET /{short_code}**
- Cache-aside: check Redis (`url:{short_code}` → original URL) first. On hit, redirect immediately.
- On miss: query Postgres, write-through to Redis with a TTL **(assumed 1h)**, then redirect.
- Click tracking happens **after** the redirect response is sent, via FastAPI `BackgroundTasks` running a single atomic statement: `UPDATE urls SET click_count = click_count + 1, last_accessed_at = now() WHERE short_code = :code`. Never read-then-write the counter in application code — that's a lost-update race under concurrent hits.
- `404` unknown code · `410` expired (and evict from Redis on expiry).

## 7. Database schema

```sql
CREATE TABLE urls (
  id               BIGSERIAL PRIMARY KEY,
  short_code       VARCHAR(10) UNIQUE NOT NULL,
  original_url     TEXT NOT NULL CHECK (char_length(original_url) <= 2048),
  is_custom_alias  BOOLEAN NOT NULL DEFAULT FALSE,
  created_at       TIMESTAMPTZ NOT NULL DEFAULT now(),
  expires_at       TIMESTAMPTZ,
  click_count      BIGINT NOT NULL DEFAULT 0,
  last_accessed_at TIMESTAMPTZ
);
CREATE INDEX idx_urls_short_code ON urls (short_code);
-- partial index: only used for the idempotency dedupe lookup, and only applies to
-- non-alias rows, so it stays small even as alias rows accumulate
CREATE INDEX idx_urls_original_url_dedupe ON urls (original_url) WHERE is_custom_alias = FALSE;
```

Short codes: 7-char base62 via `secrets`, insert-and-retry on unique-constraint collision. **Trade-off, stated explicitly:** this makes codes opaque/non-enumerable, the right call for a public shortener — the cost is an occasional wasted round-trip on collision (rare at 62^7 keyspace) versus a sequential ID's guaranteed-unique but enumerable codes.

## 8. Agent workflow (`.claude/agents/`)

| Agent | File | Job |
|---|---|---|
| Testing Agent | `testing-agent.md` | Writes/runs pytest + vitest, runs lint/type-check |
| Verification Agent | `verification-agent.md` | Read-only review against this spec + a reliability/security checklist |
| Push Agent | `push-agent.md` | Stages, commits, pushes a branch, opens a PR |

Run them in that order for every change: implement → Testing Agent → Verification Agent → Push Agent. Don't invoke Push Agent unless the other two have both passed — that discipline is on you and Claude Code here, since this starter doesn't wire up automated enforcement (see §13 for what that would take).

## 9. Observability

- Structured (JSON) logging via `structlog` or stdlib `logging` with a JSON formatter.
- Request-ID middleware: read `X-Request-ID` header or generate a `uuid4`, attach to every log line for that request.
- `/api/health`: returns `200` only if `SELECT 1` succeeds against Postgres **and** `PING` succeeds against Redis; `503` otherwise. A health check that doesn't check its dependencies isn't a health check.

## 10. Security & config

- CORS: `CORSMiddleware` with `allow_origins=[FRONTEND_ORIGIN]` from env — never `*`.
- Rate limiting: `slowapi` (or equivalent) backed by `REDIS_URL`, not in-memory — in-memory counters are per-process and silently stop working the moment you run more than one uvicorn/gunicorn worker.
- Secrets: `.env` for local dev only (gitignored, never committed — only `.env.example` is).
- Reserved-alias denylist enforced at creation time (§6).

## 11. Ops

- `backend/scripts/cleanup_expired.py`: hard-deletes rows where `expires_at < now()`, evicts them from Redis. Run daily via cron or a systemd timer. **(assumed** hard delete is acceptable — switch to a `deleted_at` soft-delete if an audit trail is ever needed.)**

## 12. Known limitations (stated explicitly, not silent)

- **Click analytics are aggregate-only** (`click_count`, `last_accessed_at`). No per-click event log (timestamp/referrer/user-agent) in v1 — deliberately deferred, not forgotten.
- **Dedupe lookup** (`idx_urls_original_url_dedupe`) is a plain B-tree on `TEXT`. Fine at this scale; at real scale, hash the URL into a separate indexed column instead of indexing the raw text.
- **No multi-region / read replica story.** Single Postgres + single Redis instance — fine for this assignment, would need connection pooling and read replicas for real production traffic.

## 13. Stretch goals

If the core app is working and passing its own tests, these are the next things a production system would need — good extensions to attempt:
- A GitHub Actions workflow that runs lint, type-check, and tests on every push/PR, with branch protection on `main` requiring it to pass. This turns §8's agent workflow from "discipline" into a gate that can't be skipped.
- `PreToolUse` hooks on the agents in `.claude/agents/` (see Claude Code's subagent docs) so Testing Agent can only edit test files, Verification Agent can't run mutating Bash commands, and Push Agent reruns tests before it's allowed to `git push`.

## 14. Conventions

- Type hints everywhere; Pydantic for all request/response models; no raw SQL string building (SQLAlchemy Core/ORM only).
- `ruff` + `mypy` clean, `eslint`/`prettier` clean.
- Commits: conventional commits (`feat:`, `fix:`, `test:`, `chore:`).
- Never commit `.env` files — only `.env.example`.
