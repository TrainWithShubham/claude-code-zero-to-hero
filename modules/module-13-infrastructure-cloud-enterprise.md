<!-- module: 13 | phase: 3 | last_verified: 2026-08-12 -->

# Module 13 — Claude Code for Infrastructure, Cloud, and Enterprise

Watch this module regardless of which capstone track you're heading toward. It's the one that makes this course different from a generic AI coding tutorial.

---

## 1. Claude Code for Infrastructure as Code

Go back to the loop from Module 1: gather context → take action → **verify results**. For application code the verify step is usually a test suite. Infrastructure already has equivalents — you just may not think of them that way:

| Signal | Plays the role of |
|---|---|
| `terraform plan` | A test suite |
| `kubectl diff` | A test suite |
| `ansible --check` | A test suite |
| `docker build` exit code | A build |
| A failing health check after a rollout | An integration test |

**Claude Code works well on infrastructure precisely when you give it one of these to check itself against.** Point it at a Terraform module with no way to run `plan` and it's pattern-matching on syntax. Give it `plan` output and it closes the loop the same way it does with a failing unit test.

Your job isn't to describe the change perfectly up front. It's to make sure the agent has a way to tell whether it worked.

### Sandboxing matters more here

For infra work, Bash *is* the tool — so re-read the Module 4 warning before you let Claude run cloud CLIs:

> Sandbox **writes** are confined to your working directory. **Reads default to the entire computer**, including `~/.aws/credentials` and `~/.ssh/`.

Configure `sandbox.credentials` or `denyRead` before a session that will touch a cloud provider.

---

## 2. Deploying on Amazon Bedrock, Google Cloud, and Microsoft Foundry

Three enterprise deployment routes, each with its own auth and IAM setup, for organizations whose policy requires everything to stay inside one cloud provider. Setup was covered in Module 3.

What's worth knowing here is **what you give up**, because it's scattered across the documentation:

| Feature | On Bedrock / Google Cloud / Foundry |
|---|---|
| Claude in Chrome | ❌ Not available |
| Channels | ❌ Not available |
| Ultrareview | ❌ Not available — `/code-review ultra` falls back to a local review |
| Self-hosted environments | ❌ Inference can't route through them |
| Auto mode | ✅ In the `Shift+Tab` cycle, but **not the starting mode**. Sonnet 5, Opus 4.7+, and Fable 5 only |
| Dynamic workflows | ✅ Available |
| Claude starting `/code-review` on its own | ❌ You type it yourself |

Check this list before promising a capability in an architecture review.

---

## 3. Self-hosted environments

A self-hosted environment executes Claude Code **cloud sessions** on infrastructure your organization operates. If your team only uses terminal and IDE sessions, there's nothing here to configure — those always run on the developer's machine.

### How it works

Three parts:

| Term | What it is |
|---|---|
| **Environment** | A named destination, created in claude.ai admin settings, grouping a set of runners |
| **Runner** | A long-lived process on your hosts that claims and executes sessions — the same idea as a self-hosted CI runner |
| **Session** | One Claude Code task a developer started |

A developer picks your environment from the session-start picker. Anthropic's control plane places the session on your environment's queue, a runner claims it, clones the repository, and spawns a Claude Code process on your host.

**Every connection is outbound HTTPS. Anthropic never connects into your network.** The runner polls `api.anthropic.com` for work (which doubles as its heartbeat), and each session child holds its own event stream and inference calls.

### Why organizations adopt it

- **Network access** — sessions run inside your network and reach internal services, databases, and registries without exposing them publicly
- **Custom tooling** — pre-install compilers, SDKs, and internal CLIs in your runner image so every session starts ready to build
- **Compliance** — repository checkouts and build artifacts stay on infrastructure you control

### The caveat that reframes the chapter

> **Model inference still uses the Anthropic API.** It cannot be routed through Amazon Bedrock, Google Cloud, Microsoft Foundry, or an LLM gateway.

Self-hosting moves **session execution** into your network. It does not move the control plane, and it does not move inference. Repository checkouts, build artifacts, secrets, and files a session creates stay on your machines — **but the conversation itself, including prompts, responses, and tool results, goes to `api.anthropic.com`.**

State that plainly. It's the difference between a compliance story that survives review and one that doesn't.

### Gating facts

- **Public beta on Team and Enterprise, off by default.** An Owner turns on *Allow self-hosted environments* on the Cloud environments admin page, which requires Claude Code on the web to be enabled first.
- **Not available with Zero Data Retention.**
- **Claude Tag, Claude Security, and Code Review sessions don't route to self-hosted environments yet.**
- Sessions check out repositories from GitHub.
- Billing is unchanged — sessions consume your organization's usage exactly as Anthropic-hosted ones do.

### A capacity-planning fact

**A runner serves one user at a time.** The first session a runner claims locks it to that user's account, and it then runs only that user's sessions up to its configured capacity. That's how checked-out code never mixes between users.

**So the minimum fleet size is the number of users you expect to be active at once** — not the number of concurrent sessions. Size accordingly, or run the autoscaling orchestrator, which starts runners as sessions queue and lets each exit when its work finishes.

---

## 4. Claude apps gateway

**The product is called the Claude apps gateway.** It's a self-hosted service that sits between your developers' Claude Code clients and your model provider.

Developers sign in with your **corporate identity provider** instead of holding API keys or cloud credentials. The gateway:

- Holds the upstream credential, so no developer does
- Enforces **model access and managed settings by IdP group**
- Applies **spend limits**
- Relays usage telemetry to your own observability stack

The detail worth remembering: **it ships inside the `claude` binary.** The same executable that runs Claude Code on a laptop runs the gateway server.

```bash
claude gateway --config gateway.yaml
```

Developers connect by pointing at the gateway URL, which you can distribute through managed settings. A signed-in gateway session outranks every other credential — including `CLAUDE_CODE_USE_BEDROCK` and friends.

### When something else fits better

Straight from the docs, and worth repeating to anyone reaching for a gateway reflexively:

> The Claude apps gateway is designed for organizations that must, or prefer to, route inference through their own cloud provider — for example to meet data residency requirements. **If you don't have that requirement**, and want features like SCIM provisioning or Claude Code on web and mobile, **Claude Enterprise may be a better fit.**

---

## 5. Admin setup

Server-delivered managed settings without needing device-management infrastructure, MCP server allowlists and denylists, and policy enforcement.

The controls that have come up across this course, in one place:

| Goal | Setting |
|---|---|
| Block `bypassPermissions` org-wide | Managed settings |
| Remove auto mode from the cycle | `disableAutoMode: "disable"` |
| Turn off dynamic workflows | `disableWorkflows: true` |
| Restrict which channel plugins can run | `allowedChannelPlugins` (with `channelsEnabled`) |
| Control MCP server access | `deniedMcpServers`, allowlists |
| Lock login to one organization | `forceLoginMethod`, `forceLoginOrgUUID` |
| Enforce a version range | `requiredMinimumVersion`, `requiredMaximumVersion` |
| Push organization-wide instructions | `claudeMd` in managed settings |
| Enforce sandbox isolation | `sandbox.enabled` |
| Enable plugins for everyone | `enabledPlugins`, `extraKnownMarketplaces` |

Two reminders from Module 5 that matter most at this scale: **managed settings outrank CLI flags**, and **permission rules merge across scopes rather than override**.

And the layering principle: use **settings for technical enforcement** and **CLAUDE.md for behavioral guidance**. Settings are enforced by the client regardless of what Claude decides.

---

## 6. Observability

### OpenTelemetry export

Configured entirely through environment variables:

```bash
export CLAUDE_CODE_ENABLE_TELEMETRY=1
export OTEL_METRICS_EXPORTER=otlp          # console, otlp, prometheus, none
export OTEL_LOGS_EXPORTER=otlp             # console, otlp, none
export OTEL_EXPORTER_OTLP_PROTOCOL=grpc
export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
export OTEL_EXPORTER_OTLP_HEADERS="Authorization=Bearer your-token"
```

**Metrics:**

| Metric | Unit |
|---|---|
| `claude_code.session.count` | count |
| `claude_code.lines_of_code.count` | count |
| `claude_code.cost.usage` | USD |
| `claude_code.token.usage` | tokens |
| `claude_code.active_time.total` | seconds |
| `claude_code.commit.count` / `claude_code.pull_request.count` | count |
| `claude_code.code_edit_tool.decision` | count |

**Events:** `user_prompt`, `assistant_response`, `api_request`, `api_error`, `tool_result`, `tool_decision`, `mcp_server_connection`, `plugin_loaded`, `auth`.

Every event carries `session.id`, `user.id`, `user.email`, `organization.id`, and — the useful one — a **`prompt.id` that correlates every event produced by a single user prompt.**

**Traces are beta:**

```bash
export CLAUDE_CODE_ENHANCED_TELEMETRY_BETA=1
export OTEL_TRACES_EXPORTER=otlp
```

The span hierarchy is `claude_code.interaction` → `claude_code.llm_request`, `claude_code.tool`, `claude_code.hook`.

Two things worth connecting:

- **`OTEL_LOG_TOOL_DETAILS=1`** records skill names verbatim instead of redacted, which is what makes the `skill_activated` event usable for finding unused skills — the technique from Module 6.
- Administrators deploy all of this through the **`env` block in managed settings**, so developers don't configure it individually.

### Analytics dashboards

Separate from OTel export, Anthropic hosts analytics for tracking adoption and engineering velocity across a team, at `claude.ai/analytics/...`. The Code Review dashboard from Module 12 — PRs reviewed, weekly cost, resolved comments, per-repo breakdown — is one example.

Use OTel when you want the data in your own stack. Use the dashboards when you want the answer without building anything.
