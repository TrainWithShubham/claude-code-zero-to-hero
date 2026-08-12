<!-- module: 12 | phase: 3 | last_verified: 2026-08-12 -->

# Module 12 — CI/CD, Code Review, and Security

Everything so far has been Claude Code working with you. This module is about Claude Code working without you — in CI, on pull requests, and inside the boundaries you set.

---

## 1. Claude Code GitHub Actions

### Setup

```
/install-github-app
```

This installs the Claude GitHub App, saves your credential as a repository secret, pushes a branch with the workflow files, and opens a PR ready to create. You need admin access to the repository and the [GitHub CLI](https://cli.github.com) authenticated with `gh auth login`.

The credential is one of two secrets:

- **`ANTHROPIC_API_KEY`** — a key from the Claude Console
- **`CLAUDE_CODE_OAUTH_TOKEN`** — a subscription token from `claude setup-token`, available on Pro, Max, Team, and Enterprise

In the workflow, pass whichever you used to the matching input: `anthropic_api_key` or `claude_code_oauth_token`.

### Interactive vs automation mode

The action detects the mode from your workflow — the switch is simply whether you supply a `prompt`:

| Mode | Trigger | Results appear |
|---|---|---|
| **Interactive** | No `prompt` — waits for `@claude` in a comment, review, or new issue | As a comment on the triggering issue or PR |
| **Automation** | A `prompt` is supplied — runs on any GitHub event | In the workflow run log |

A minimal interactive workflow:

```yaml
name: Claude Code
on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
jobs:
  claude:
    if: contains(github.event.comment.body, '@claude')
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
      issues: write
      id-token: write
      actions: read
    steps:
      - uses: actions/checkout@v6
        with:
          fetch-depth: 1
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

The non-boilerplate parts: `id-token: write` is required for the action's default GitHub App authentication, `actions: read` lets Claude read CI results, and the `if` keeps runners from starting on comments that don't mention `@claude`.

### Who can trigger a run

Two checks run on the triggering actor before Claude starts, and the run fails if either rejects it:

- **Write access** — the triggering user must have write access to the repository. Add exceptions with `allowed_non_write_users`.
- **Human actor** — bot actors are rejected unless listed in `allowed_bots`. This is what stops Claude from triggering itself in a loop.

### Useful inputs

| Input | Purpose |
|---|---|
| `prompt` | Instructions, as plain text or a skill invocation like `/code-review` |
| `claude_args` | Any CLI argument — `--max-turns 5`, `--model claude-sonnet-5`, `--allowedTools` |
| `trigger_phrase` | Defaults to `@claude` |
| `plugin_marketplaces` / `plugins` | Install plugins before the run |
| `use_bedrock` / `use_vertex` / `use_foundry` | Route inference through your own cloud account |

### For an organization

Install the app once at the organization level, store the secret as an organization Actions secret, and either add the workflow per repository or define it once as a reusable workflow.

Use an **API key** rather than an OAuth token for a shared secret — an OAuth token is tied to the subscription of whoever ran `claude setup-token`.

To avoid a long-lived secret entirely, use **workload identity federation**: the action exchanges the workflow's GitHub OIDC token for Claude API access through a Console service account.

> **The GitHub App's permission set is shared across every Claude GitHub feature** — the Action, Code Review, and web auto-fix. It covers Actions, Checks, Contents, Discussions, Issues, Members, Metadata, Pull requests, Repository hooks, Statuses, and Workflows, and **GitHub doesn't let you accept a subset.** If your organization needs only what the Action uses, create a custom GitHub App with Contents, Issues, and Pull requests — but Code Review and web auto-fix still require the official app.

### Upgrading from beta

If your workflows reference `@beta`: change it to `@v1`, remove the `mode` input (it's auto-detected now), rename `direct_prompt` to `prompt`, and move `max_turns` and `model` into `claude_args`.

---

## 2. Claude Code in GitLab CI/CD

The same idea wired into GitLab pipelines, for teams not on GitHub. For self-hosted GitHub, see GitHub Enterprise Server.

---

## 3. Automated code review

There are three distinct things here, and they're easy to conflate.

| | What it is | Where it runs |
|---|---|---|
| **`/code-review`** | A command in your session | Locally, as a background subagent |
| **`/code-review ultra`** | Escalates to **ultrareview** | In the cloud |
| **Code Review** | A managed GitHub App | Anthropic infrastructure, on your PRs |

### `/code-review` locally

Reviews your branch's commits ahead of upstream plus uncommitted changes. Pass a target to review something else — a file path, PR number, branch name, or `main...my-feature`.

Flags:
- `--fix` applies findings to your working tree
- `--comment` posts them as inline PR comments
- `--post` preselects posting an `ultra` review's findings to the PR

**Effort levels trade coverage against confidence**, which is more specific than "quick to deep":

> At `low` and `medium`, the review reports only the findings it's most confident in, so you see fewer false positives. `high` through `max` broaden coverage and may include findings the review is less sure about.

If you don't type a level, it **reuses the last level you typed — even from an earlier session** — and shows a notice saying so. `ultra` neither uses nor updates that memory.

⚠️ **A background review's `--fix` edits land outside your session's checkpoints, so `/rewind` won't undo them.** Use git. When the review runs in the foreground, `/rewind` works normally.

### Ultrareview

`/code-review ultra` runs a deeper review **in the cloud**, with its own scope: your current branch against the repository's default branch, plus uncommitted and staged changes. `claude ultrareview` is the non-interactive subcommand, and `claude -p '/code-review ultra'` launches one from a script.

**Ultrareview requires a claude.ai account.** It's unavailable on Amazon Bedrock, Google Cloud, and Microsoft Foundry, and for organizations with Zero Data Retention. When unavailable, `/code-review ultra` quietly runs a local review instead.

> **Naming history worth knowing:** `/review` is now an alias of `/code-review` — before v2.1.223 it was a separate single-pass PR review. `/simplify` was `/code-review`'s own name before v2.1.147; since v2.1.154 it's a separate **cleanup-only** review that applies fixes without hunting for bugs.

### The managed Code Review app

Multiple agents analyze the diff and surrounding code in parallel, a verification step checks candidates against actual behavior to filter false positives, and results are deduplicated, ranked, and posted as inline comments.

**Availability and cost:**
- **Research preview, Team and Enterprise plans only.** Not available with Zero Data Retention.
- **$15–25 per review**, scaling with PR size. Billed through **usage credits** — it does **not** count against your plan's included usage.
- Averages 20 minutes.

**Severity markers:** 🔴 Important (a bug to fix before merging) · 🟡 Nit (minor, non-blocking) · 🟣 Pre-existing (a bug that was already there).

**Three trigger modes per repository:** once after PR creation, after every push (most reviews, highest cost), or manual.

**The check run always completes with a neutral conclusion, so it never blocks a merge.** To gate on findings, read the machine-readable severity counts out of the check run in your own CI:

```bash
gh api repos/OWNER/REPO/check-runs/CHECK_RUN_ID \
  --jq '.output.text | split("bughunter-severity: ")[1] | split(" -->")[0] | fromjson'
```

**Manual triggers:**

| Command | Effect |
|---|---|
| `@claude review` | A single review, **without** subscribing to future pushes |
| `@claude review always` | A review, **and** subscribes the PR to push-triggered reviews |
| `@claude review once` | Same as the bare command |

⚠️ **This changed in July 2026** — `@claude review` used to subscribe the PR. If you relied on that, use `always`.

### Tuning: CLAUDE.md vs REVIEW.md

These are weighted very differently:

| File | How it's used |
|---|---|
| **`CLAUDE.md`** | Read as project context. Newly introduced violations are flagged as **nits** |
| **`REVIEW.md`** | Injected into **every review agent's system prompt as the highest-priority instruction block** |

Two things that catch people out:

- **`REVIEW.md` is pasted verbatim**, so `@` import syntax is not expanded and referenced files are not read. Put the rules directly in the file.
- **The local `/code-review` command does not read `REVIEW.md`.** Only the managed service does.

What `REVIEW.md` is actually for:

```markdown
# Review instructions

## What Important means here
Reserve Important for findings that would break behavior, leak data, or block
a rollback. Style, naming, and refactoring suggestions are Nit at most.

## Cap the nits
Report at most five Nits per review. If you found more, say "plus N similar
items" in the summary instead of posting them inline.

## Do not report
- Anything CI already enforces: lint, formatting, type errors
- Generated files under `src/gen/` and any `*.lock` file

## Always check
- New API routes have an integration test
- Log lines don't include email addresses, user IDs, or request bodies
```

Keep it short. A long `REVIEW.md` dilutes the rules that matter.

---

## 4. Security fundamentals

The isolation ladder from Module 4 still applies: **sandboxed Bash tool → dev containers or Docker → full VMs**, picked by how untrusted the code or task actually is.

What's worth doing here is assembling the security story the course has been building across modules, because none of these pieces makes sense alone:

| Layer | The guarantee |
|---|---|
| **Hooks** (Module 7) | A `PreToolUse` deny holds in **every** permission mode, including `bypassPermissions` and `--dangerously-skip-permissions`. The only unbypassable layer |
| **Managed settings** (Module 5) | Enforced by the client; user and project settings can't override |
| **Sandbox** (Module 4) | OS-level. But **writes are bounded and reads default to the whole machine** — configure `sandbox.credentials` |
| **MCP** (Module 9) | The spec treats tool descriptions from untrusted servers as **untrusted input** |
| **Inter-agent messages** (Module 10) | A teammate can't approve a prompt on your behalf, and a denial can't be relayed around |
| **CLAUDE.md** | Advisory only. Not a security control |

The one-line version to leave learners with: **if a rule matters, it belongs in a hook or managed settings — never in a markdown file.**

---

## 5. Managing costs

`/usage` breaks down what's driving your plan limits by skill, subagent, plugin, and MCP server.

The main levers, most of them from earlier modules:

- **Model selection** — Haiku for volume, per-subagent `model:` overrides
- **Effort levels** — the single biggest dial
- **Prompt-caching habits** (Module 4) — pick a model at session start, don't edit CLAUDE.md mid-task
- **`--max-turns`** in `claude_args` for CI runs
- **Workflow size guideline** (Module 10) — default `medium`, under 15 agents
- **Concurrency controls and timeouts** in your workflows

Three separate cost surfaces appear in this module, and it's worth naming them so nobody is surprised by a bill:

| Surface | Billed as |
|---|---|
| GitHub Actions runs | GitHub Actions minutes **plus** tokens (or subscription usage with an OAuth token) |
| Managed Code Review | **Usage credits, separately** — not against plan usage |
| Local `/code-review` and ultrareview | Your normal plan usage |

---

## 6. Hands-on: wiring Claude Code into a pipeline

**Try:** run `/install-github-app`, merge the workflow PR, then open an issue and comment `@claude implement this`.

**Try:** run `/code-review high` locally before pushing, then compare what it found against what the PR review reports.

**Try:** add a `REVIEW.md` that caps nits at five and skips generated files. Open a PR that would previously have produced a wall of style comments, and see the difference.

**A distinction worth making explicitly:** for automatic reviews on every PR you don't need a workflow file at all — that's the managed Code Review app. Write a workflow when you want to control the prompt, model, and triggers yourself.

---
<!-- nav -->

[← Claude Code Across Every Surface](module-11-every-surface.md) · [All modules](../README.md#the-course) · [Infrastructure, Cloud, and Enterprise →](module-13-infrastructure-cloud-enterprise.md)
