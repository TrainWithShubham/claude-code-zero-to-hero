<!-- module: 15 | phase: 4 | last_verified: 2026-08-12 -->

# Module 15 — Capstone: Choose Your Track

Everyone reaches this module having learned the same things. From here the work splits, so a frontend developer and an SRE each leave with a portfolio piece that matches the job they actually hold.

Pick one track and build it properly. Module 13 applies to both.

---

## 1. Capstone kickoff — planning with plan mode and auto mode

### Start by being interviewed

Before you write a prompt, have Claude pull the requirements out of you:

```text
I want to build [brief description]. Interview me in detail using the AskUserQuestion tool.
Ask about technical implementation, UI/UX, edge cases, concerns, and tradeoffs.
Don't ask obvious questions, dig into the hard parts I might not have considered.
Keep interviewing until we've covered everything, then write a complete spec to SPEC.md.
```

Then **start a fresh session to execute the spec.** Two reasons this is the right opener for a project this size:

- A written `SPEC.md` **survives compaction**; conversation history doesn't
- The new session begins with clean context focused entirely on implementation

The most useful specs name the files and interfaces involved, state what's out of scope, and end with an end-to-end verification step that proves the feature works.

### Then plan, edit, and approve

Enter plan mode with `Shift+Tab`, ask for an implementation plan, and press **`Ctrl+G`** to open it in your editor and change it before approving. Approve into `acceptEdits` or auto mode for the build.

Same discipline as Module 6, on a bigger project.

---

## 2. Track A (general dev): ship a full-stack SaaS feature

Ship one real feature end to end — API, UI, and tests — in an existing or starter repo.

What this track exercises from earlier modules:

| Module | Applied as |
|---|---|
| 5 | A tight `CLAUDE.md`, plus path-scoped rules if the repo has distinct areas |
| 6 | Plan mode, named sessions, `/branch` when you try a second approach |
| 7 | At least one project skill for a workflow you repeat |
| 8 | A code-intelligence plugin so Claude sees type errors immediately after edits |
| 11 | **Claude in Chrome** — build, open localhost, read the console, fix |

That last row is what makes Track A visibly more than "I used an AI to write code." The build → verify-in-a-real-browser → fix loop is the whole thesis of the course, running end to end.

---

## 3. Track B (DevOps/SRE): a self-healing CI/CD or incident-response agent

Build an agent that watches a pipeline or an alert feed and takes a **bounded remediation action behind a human-approval gate**.

### Choose how the event reaches the agent

There are four options and they behave very differently:

| Approach | Trigger | Claude runs on | Notes |
|---|---|---|---|
| **Channels** (Module 9) | Webhook, Telegram, Discord | Your machine, CLI | Research preview. **Anthropic auth only — not on Bedrock, Google Cloud, or Foundry.** Needs Bun, and Team/Enterprise orgs must enable it |
| **Routines** (Module 11) | Schedule, API POST, GitHub event | Anthropic cloud | Belongs to **an individual account** and acts as **you** — commits and PRs carry your identity |
| **GitHub Actions** (Module 12) | Any GitHub event | A CI runner | Best when the trigger already lives in GitHub |
| **Agent SDK** (Module 14) | Whatever you build | Your infrastructure | Most control, most to operate |

Check the constraints before you design around one.

### The approval gate has a correct answer

Put the gate in a **`PreToolUse` hook**, not a CLAUDE.md instruction.

CLAUDE.md is advisory. A hook fires **before any permission-mode check, in every mode**, including `bypassPermissions`. For an agent that can touch production, that difference is the entire design. This is the assessable decision in Track B.

### Bound the blast radius

- `permissions.deny` rules for anything the agent must never touch
- `max_turns` or `max_budget_usd` if it runs unattended
- A **verification signal** — `terraform plan`, `kubectl diff`, a health check — that the agent must read before declaring success

---

## 4. Build sprint

Use what you built in earlier modules rather than defaulting to raw prompting. A finished capstone should be able to check off:

- [ ] A `CLAUDE.md` under 200 lines that passes *"would removing this cause Claude to make mistakes?"*
- [ ] At least one **skill** you wrote
- [ ] At least one **hook** — and for Track B, the approval gate is a hook
- [ ] A **verification signal** Claude can run itself
- [ ] Sessions **named** with `claude -n`, and `/branch` used at least once
- [ ] `max_turns` or a budget on anything unattended
- [ ] Subagents or a workflow used for something genuinely parallel, not for its own sake

---

## 5. Capstone review

Run the same review ladder you'd run on production code:

1. **`/code-review high`** locally before pushing — high effort broadens coverage
2. **`/code-review ultra`** for the deep cloud pass
3. **`/security-review`**, or the **Claude Security plugin** for a multi-agent scan that produces patches you apply yourself

### Check availability before you promise it

- **Ultrareview** requires a claude.ai account and is unavailable on Bedrock, Google Cloud, and Foundry — where `/code-review ultra` quietly falls back to a local review
- The managed **Code Review** GitHub App is **Team and Enterprise only**, so most individual learners won't have it. Use `/code-review` and the Claude Security plugin instead
- The **Claude Security plugin** needs a paid plan (it uses dynamic workflows; enable them in `/config` on Pro) and `python3` 3.9.6+

Hold yourself to the bar from Module 12: fix what the review finds, then re-run it.

---

## 6. Presenting and documenting with Artifacts

Publish the result as a live, interactive page on claude.ai instead of a static README. It's a stronger portfolio piece, and it's a link you can send.

```text
Make an artifact that walks through this project: the architecture, the key
decisions, and an annotated diff of the most interesting change.
```

Claude writes an HTML or Markdown file in your project, asks permission the first time, publishes it, and prints the URL. `Ctrl+]` reopens the most recent artifact. Republishing updates the same URL and creates a new version.

### Availability

| Requirement | Detail |
|---|---|
| **Plan** | Pro, Max, Team, or Enterprise |
| **Auth** | Must be signed in with `/login`. **API keys, gateway tokens, and cloud-provider credentials cannot publish** |
| **Provider** | Anthropic API only — **not Bedrock, Google Cloud, or Foundry** |
| **Version** | CLI v2.1.183+ or desktop v1.13576.0+ |
| **Contexts** | **Off by default in the Agent SDK, GitHub Actions, and MCP-server contexts** — relevant if Track B publishes from an agent |

### Sharing

A new artifact is **private to you**.

- **On Pro and Max, a public link is the only sharing option** — which is exactly what you want for a portfolio piece
- On **Team and Enterprise** you can share within the organization, assign **editor** roles, and publish publicly only after an Owner enables external sharing

### Live data, and the catch

An artifact can call **MCP connectors each time someone opens it**, so a dashboard shows current data instead of a snapshot. Each viewer's calls run through **their own** connected accounts, and each viewer approves access first — so include a fallback message naming the connector a section needs.

> ⚠️ **A connector-backed artifact cannot be shared to a public link on any plan.** For a public portfolio piece, keep it static.

### Page constraints

| Constraint | Effect |
|---|---|
| No external requests | A strict CSP blocks external scripts, styles, fonts, and images. Everything is inlined or embedded as data URIs |
| No backend | Static page. It can't store form input or authenticate viewers |
| Single page | Relative links don't resolve; use in-page anchors |
| File types | `.html`, `.htm`, or `.md` |
| Size | 16 MiB rendered, maximum |

A styled page costs more output tokens than terminal text. Prefer SVG or HTML/CSS for diagrams over embedded raster images, which are the usual reason a publish fails on size.

### Brand it from CLAUDE.md

Claude applies a built-in design skill, and it looks for a design system in your project first. Record one and every artifact matches:

```markdown
## Design system

- Colors: primary #1a4d8f, accent #f59e0b, surface #f8fafc
- Typography: Inter for body, JetBrains Mono for code
- Spacing: 8px scale, 6px border radius
```

Your design system outranks Claude's own choices, and your prompt outranks both.

That's a fitting last exercise: the file you learned to write in Module 5 is what makes your final deliverable look like yours.

---
<!-- nav -->

[← Building Products with the Agent SDK](module-14-agent-sdk.md) · [All modules](../README.md#the-course) · [Staying Sharp: What's Next →](module-16-staying-sharp.md)
