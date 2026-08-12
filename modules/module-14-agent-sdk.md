<!-- module: 14 | phase: 4 | format: deep-dive | last_verified: 2026-08-12 -->

# Module 14 — Building Products with the Agent SDK

Every module so far has been about *using* Claude Code. This one is about building on it: the same tools, the same agent loop, the same context management, as a library in your own process.

---

## Chapter 1 — Agent SDK overview

**What you'll learn:** what the SDK is, when to reach for it, and the rules that apply if you ship a product on it.

### The concept

The Claude Agent SDK gives you the tools, [agent loop](#chapter-2--the-agent-loop), and context management that power Claude Code, programmable in **Python and TypeScript**.

> The SDK is a library for Python and TypeScript only. To drive the same agent loop from another language, run the CLI as a subprocess with `-p` and `--output-format json`.

That's the answer for Go, Java, or Rust codebases — you don't get a native library, you get a well-behaved subprocess with structured output.

### Which one do I actually need?

| If you're… | Use | Why |
|---|---|---|
| Building an agent without implementing the tool loop yourself | **Agent SDK** | Runs the loop in your own process |
| Doing interactive development or one-off terminal tasks | **Claude Code CLI** | Built for daily interactive use |
| Calling the API directly and writing the loop yourself | **Client SDK** | Direct API access; you own the loop |
| Running long or async agents without managing a sandbox | **Managed Agents** | Hosted REST API — Anthropic runs the agent *and* the sandbox |

### Everything from Claude Code is here

| Capability | What you get |
|---|---|
| Built-in tools | Read, Write, Edit, Bash, Glob, Grep, WebSearch, WebFetch, ToolSearch |
| Hooks | Custom code at lifecycle points |
| Subagents | Focused workers with their own context |
| MCP | External tools and data sources |
| Permissions | Which tools run automatically, which need approval |
| Sessions | Resume and fork |
| Skills, commands, memory | Load from `.claude/` and `~/.claude/` |
| Plugins | Load bundled extensions by local path |

### Two rules if you ship a product on it

- **Authentication:** *"Unless previously approved, Anthropic does not allow third party developers to offer claude.ai login or rate limits for their products, including agents built on the Claude Agent SDK."* Use API key authentication.
- **Branding:** "Claude Agent", "Claude" inside an Agents menu, and "*YourAgentName* Powered by Claude" are allowed. **"Claude Code" and "Claude Code Agent" are not**, nor is Claude Code-branded ASCII art or visuals that mimic it. Your product keeps its own branding.

### Key takeaways

- Python and TypeScript library; other languages drive the CLI as a subprocess
- Agent SDK for your own loop, Managed Agents when you don't want to host a sandbox
- API key auth only for third-party products, and you can't call it "Claude Code"

---

## Chapter 2 — The agent loop

**What you'll learn:** the message lifecycle, and how to stop the loop safely.

### The cycle

1. **Receive prompt** — the SDK yields a `SystemMessage` with subtype `"init"` carrying session metadata
2. **Evaluate and respond** — Claude replies with text, tool calls, or both. The SDK yields an `AssistantMessage`
3. **Execute tools** — the SDK runs them and yields a `UserMessage` with the results
4. **Repeat** — steps 2 and 3 cycle until Claude produces a response with no tool calls
5. **Return result** — a final `AssistantMessage`, then a `ResultMessage` with text, usage, cost, and session ID

**A turn is one round trip.** Tool execution happens without yielding control back to your code.

### The five message types

| Type | Emitted |
|---|---|
| `SystemMessage` | Lifecycle. Subtypes: `init`, `compact_boundary`, `informational`, `worker_shutting_down` |
| `AssistantMessage` | After each Claude response, including the final text-only one |
| `UserMessage` | After each tool execution, and for inputs you stream mid-loop |
| `StreamEvent` | Only with partial messages enabled |
| `ResultMessage` | End of the loop |

### Reading the result correctly

`ResultMessage.subtype` is the primary signal, and **the `result` field only exists on `success`**:

| Subtype | What happened | `result` present |
|---|---|---|
| `success` | Finished normally | ✅ |
| `error_max_turns` | Hit the turn limit | ❌ |
| `error_max_budget_usd` | Hit the spend limit | ❌ |
| `error_during_execution` | An error interrupted the loop | ❌ |
| `error_max_structured_output_retries` | No valid structured output within the retry limit | ❌ |

All subtypes still carry `total_cost_usd`, `usage`, `num_turns`, and `session_id`, so you can track cost and resume even after a failure.

Two behaviors that will surprise you:

- **A single-shot `query()` raises after yielding an error result.** Wrap the loop in a try block if your code needs to continue.
- **`ResultMessage` isn't necessarily the last message.** Trailing system events such as `prompt_suggestion` can arrive after it — iterate to completion rather than breaking on the result.

### Bounding the loop

| Option | Controls | Default |
|---|---|---|
| `max_turns` / `maxTurns` | Maximum **tool-use** round trips | No limit |
| `max_budget_usd` / `maxBudgetUsd` | Maximum cost before stopping | No limit |

**Set a budget by default in production.** Without limits the loop runs until Claude decides it's done, which is fine for a well-scoped task and open-ended on "improve this codebase."

**The budget cap covers subagents.** Their spend counts toward the total, and once the cap is reached, spawning another fails with `Budget limit reached` and background subagents are stopped.

### Automatic compaction

When context approaches the limit the SDK compacts automatically and emits `subtype: "compact_boundary"`.

**Compaction replaces older messages with a summary, so instructions from early in the conversation may not survive.** Persistent rules belong in CLAUDE.md — loaded via `settingSources` and re-injected every request — not in the initial prompt. You can also add a summarization-instructions section to CLAUDE.md; the compactor reads it like any other context.

### Try it yourself

```python
async for message in query(
    prompt="Find and fix the bug causing test failures in the auth module",
    options=ClaudeAgentOptions(
        allowed_tools=["Read", "Edit", "Bash", "Glob", "Grep"],
        setting_sources=["project"],
        max_turns=30,
        effort="high",
    ),
):
    if isinstance(message, ResultMessage):
        if message.subtype == "success":
            print(message.result)
        else:
            print(f"Stopped: {message.subtype}")
```

### Common pitfalls

- **Reading `message.result` without checking the subtype.** It's only there on `success`.
- **Not wrapping a single-shot `query()` in a try block.** It raises after an error result.
- **Putting persistent rules in the initial prompt.** Compaction can drop them; CLAUDE.md survives.

### Key takeaways

- Five message types; `init` starts a run, `ResultMessage` ends it
- Branch on `ResultMessage.subtype` before reading `result`
- `max_turns` counts tool-use turns; `max_budget_usd` covers subagents too

---

## Chapter 3 — Custom tools and structured outputs

**What you'll learn:** how to give Claude your own functions and get typed data back.

Beyond the built-in tools you can connect external services with MCP servers, define your own with custom tool handlers, and load project skills via setting sources.

**Structured outputs** give you validated JSON for anything downstream that needs type-safe data. When no valid output is produced within the retry limit, the loop ends with `error_max_structured_output_retries`.

**One performance detail:** read-only built-ins like `Read`, `Glob`, and `Grep` run concurrently when Claude requests several at once; state-changing tools like `Edit`, `Write`, and `Bash` run sequentially. **Custom tools default to sequential** — set `readOnlyHint` in the tool's annotations to opt into parallel execution.

### Key takeaways

- Custom tool handlers for your functions, MCP for external services
- Structured outputs return validated JSON, with a dedicated error subtype
- Custom tools are sequential unless you set `readOnlyHint`

---

## Chapter 4 — Subagents, skills, and plugins inside the SDK

**What you'll learn:** that you don't choose between Claude Code's features and the SDK.

Every extension concept from earlier modules works here: subagents, skills, commands, hooks, plugins, MCP, permissions, and memory.

**The switch is `settingSources`.** Skills, commands, and CLAUDE.md load from `.claude/` and `~/.claude/` only when you opt in:

```python
setting_sources=["project"]
```

That single option is why an SDK agent can follow the same conventions as your team's Claude Code sessions. It also becomes a **security control** in Chapter 5.

**Hooks run in your application process, not inside the agent's context window**, so they cost no context. A `PreToolUse` hook that rejects a call prevents it from executing and Claude receives the rejection instead — same guarantee as Module 7.

**Subagents keep context lean.** Each starts with a fresh conversation and only its final response returns to the parent as a tool result, so the main agent's context grows by a summary rather than the whole transcript.

### Key takeaways

- Everything from Claude Code is available; `settingSources` is the opt-in
- Hooks cost no context and can still block tools
- Subagents return summaries, not transcripts

---

## Chapter 5 — Hosting the Agent SDK in production

**What you'll learn:** the one architectural fact everything else follows from.

### The subprocess model

> When your code calls `query()`, the SDK **spawns a separate `claude` CLI subprocess** and talks to it over stdio. That subprocess owns the shell, the working directory, and the JSONL session transcripts on local disk.

**One agent session maps to one subprocess.** Running N concurrent sessions means N process trees, each with its own transcript file. By default they inherit your application's working directory, so pass `cwd` per `query()` when sessions need separate filesystems.

Hosting this is not like hosting a stateless API wrapper. Everything below is a consequence.

### State that lives on local disk

None of it survives a container restart, a scale-down, or a move to another node:

| State | Default location |
|---|---|
| Session transcripts | `~/.claude/projects/`, or under `CLAUDE_CONFIG_DIR` |
| CLAUDE.md memory files | `~/.claude/CLAUDE.md` and the session's working directory |
| Working-directory artifacts | The session's working directory |

A **`SessionStore` adapter** mirrors transcripts to durable storage (S3, Redis, Postgres, or your own). It mirrors **transcripts only** — memory files and working-directory artifacts need their own strategy. When a batch can't be delivered, the SDK drops it, emits a `mirror_error` system message, and continues, so alert on those if durability matters.

### Four session patterns

| Pattern | Shape | Fits |
|---|---|---|
| **Ephemeral** | A container per task, destroyed when it completes | Bug fix, extraction, translation, media transforms |
| **Long-running** | Persistent containers, often many sessions each | Email triage, Slack bots, site builders |
| **Hybrid** | Ephemeral containers that hydrate from a `SessionStore` | Work spanning hours or days with idle gaps |
| **Multi-agent** | Several SDK subprocesses in one container | Agents collaborating in a shared environment |

For the hybrid pattern a **`SessionStore` is required, not optional** — shutting down without one loses the transcript with the container.

### Sizing

- **1 GiB RAM, 5 GiB disk, 1 CPU per agent** is a reasonable starting point — a floor, not a ceiling
- Concurrency on a host is bounded by RAM:

```text
agents per host = (host RAM − overhead) / (per-session RAM ceiling)
```

Measure the ceiling by running a representative session at your target length under real tool load and recording peak RSS. For long-running containers, pin each session to a container with consistent hashing on `sessionId`.

### Cost

> **Anthropic token cost typically dominates container infrastructure cost by an order of magnitude or more.**

A minimally provisioned container runs roughly **$0.05 per hour**; a single long agent session can spend dollars in tokens. Optimize the tokens, not the container.

### Multi-tenant isolation

Default behavior reads settings and CLAUDE.md from the filesystem. In a shared container serving multiple tenants, that **leaks one tenant's context into another's session**. Four options together:

```python
options=ClaudeAgentOptions(
    cwd=tenant_dir,              # per-tenant working directory
    setting_sources=[],          # no filesystem settings
    env={
        "CLAUDE_CONFIG_DIR": config_dir,          # per-tenant config
        "CLAUDE_CODE_DISABLE_AUTO_MEMORY": "1",   # see below
    },
)
```

> ⚠️ **`CLAUDE_CODE_DISABLE_AUTO_MEMORY=1` is separately required.** Auto memory at `~/.claude/projects/<project>/memory/` loads into the system prompt **regardless of `settingSources`**. Setting `settingSources: []` alone does not isolate tenants.

Add per-tenant egress rules at your proxy so a compromised tenant can't exfiltrate through another's outbound policy.

### Auth and secrets

- **Anthropic API** — the subprocess reads `ANTHROPIC_API_KEY` from its environment. Supply from a secret manager, or set `ANTHROPIC_BASE_URL` to a proxy that injects the key outside the container
- **Inbound** — authenticate at a gateway in front of the agent. The agent should receive pre-authenticated requests
- **Outbound tools** — keep tool credentials out of the agent environment; let a proxy inject them after the request leaves

### Observability

The SDK inherits OpenTelemetry configuration from the environment, so the Module 13 variables apply unchanged — set them at the container level and every `query()` exports.

### Known limitations

| Limitation | What to do |
|---|---|
| **No top-level session timeout** | Bound with `maxTurns` |
| Memory growth over long sessions | Cap session length or recycle subprocesses |
| Large parallel subagent fan-outs hit rate limits | Batch instead of one wide dispatch |
| No per-subagent wall-clock deadline | Cap each with `maxTurns` in its `AgentDefinition` |

### Key takeaways

- One session equals one `claude` subprocess with local state — that's the whole hosting story
- Four patterns; hybrid requires a `SessionStore`
- 1 GiB / 5 GiB / 1 CPU to start; tokens dominate cost
- Multi-tenancy needs `settingSources: []` **and** `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`

---

## Chapter 6 — Pairing the Agent SDK with durable execution

**What you'll learn:** why long-running agents want a durability layer underneath them.

Read Chapter 5's limitations as a list of failure modes:

- **No top-level session timeout** — a session doesn't end on its own
- **Local-disk state** that doesn't survive a restart, scale-down, or node move
- **Memory growth** over long sessions
- **Error results terminate the process** on a single-shot query

Those are exactly the problems durable execution engines exist to solve. The division of labour:

| Layer | Owns |
|---|---|
| **Agent SDK** | The agent loop — planning, tool calls, context management |
| **Durable execution (Temporal)** | Durability, retries, and state across long-running, failure-prone work |

The seam is already in the SDK's design: **`SessionStore`** exists because the SDK expects transcripts to outlive any single container. A durable workflow engine is the natural owner of that state, plus the retry and timeout semantics the SDK deliberately doesn't provide.

Together you get an agent that survives crashes and restarts, which is the pattern behind KubeHealer.

---
<!-- nav -->

[← Infrastructure, Cloud, and Enterprise](module-13-infrastructure-cloud-enterprise.md) · [All modules](../README.md#the-course) · [Capstone: Choose Your Track →](module-15-capstone.md)
