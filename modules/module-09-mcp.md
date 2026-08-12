<!-- module: 9 | phase: 2 | format: deep-dive | last_verified: 2026-08-12 -->

# Module 9 — MCP (Model Context Protocol) in 2026

Everything so far has extended Claude Code with things you write. MCP extends it with things other people run — your issue tracker, your monitoring stack, your database, your cloud.

---

## Chapter 1 — What MCP is and why it became the standard

**What you'll learn:** what problem MCP solves, and the moment you should reach for it.

### The concept

MCP is an open standard for connecting agents to external tools and data sources. Claude Code speaks it, and so do hundreds of services.

The trigger to remember:

> **Connect a server when you find yourself copying data into chat from another tool.**

If you're pasting a Jira ticket, a Sentry stack trace, or a query result into the prompt, that's a server you should have connected. Once it's connected, Claude reads and acts on that system directly instead of working from what you pasted.

### The architecture in three words

- **Hosts** — the LLM application that initiates connections (Claude Code)
- **Clients** — connectors inside the host
- **Servers** — the services providing context and capabilities

Servers offer three things: **Resources** (context and data), **Prompts** (templated messages and workflows), and **Tools** (functions Claude can execute). Clients can offer **Elicitation** — letting a server ask the user for more information mid-task.

### What it unlocks

- **Implement from an issue tracker** — "Add the feature described in ENG-4521 and open a PR"
- **Analyze monitoring data** — "Check Sentry for errors in the checkout flow this week"
- **Query databases** — "Find the 10 most recent signups in Postgres"
- **React to external events** — an MCP server can also push events *into* your session (Chapter 5)

### Key takeaways

- MCP is the standard interface between Claude Code and everything outside your codebase
- Hosts, clients, servers; servers expose resources, prompts, and tools
- The signal you need one: you're copy-pasting from another tool

---

## Chapter 2 — The 2026-07-28 spec

**What you'll learn:** what changed in the current revision and why it matters in practice.

### Stateless core

The current specification revision is **2026-07-28**. Its base protocol is:

- JSON-RPC 2.0 message format
- **Stateless, self-contained requests**
- **Per-request capability negotiation**

The practical consequence: a server no longer has to hold a session open across requests, so it can run on serverless or edge infrastructure like any other stateless HTTP service. That's why the ecosystem of hosted remote servers grew the way it did.

### Three opt-in extensions

Beyond the core protocol, MCP defines extensions that are always opt-in and negotiated during initialization:

| Extension | What it adds |
|---|---|
| **Tasks** | Asynchronous execution of long-running operations, with polling, mid-flight input, and durable handles |
| **MCP Apps** | Interactive UI elements — charts, forms, video players — rendered inline in conversations |
| **Skills over MCP** | Structured agent-workflow instructions, discovered and consumed through MCP |

That last one connects directly to Module 7: skills have a distribution path over MCP, not just through plugins.

### Authentication in Claude Code

Remote servers commonly use OAuth 2.0. Claude Code handles the flow for you: run `/mcp` to authenticate interactively, and a server that returns a `WWW-Authenticate` header pointing at its authorization server gets automatic discovery. When a stored refresh token is rejected, Claude Code surfaces a notice pointing at `/mcp`, whose menu offers **Re-authenticate**.

### A security principle from the spec itself

> Descriptions of tool behavior, such as annotations, **should be considered untrusted** unless obtained from a trusted server.

The spec treats tool descriptions as an attack surface. Keep that in mind when adding a server you didn't write — it comes back in Module 12.

### Key takeaways

- Current revision is 2026-07-28: stateless, self-contained requests, per-request capability negotiation
- Extensions: Tasks, MCP Apps, Skills over MCP
- Tool descriptions from untrusted servers are untrusted input

---

## Chapter 3 — Connecting your first MCP server

**What you'll learn:** the transports, the scopes, and the two things that trip everyone up.

### Adding a server

```bash
# HTTP — most remote servers
claude mcp add --transport http notion https://mcp.notion.com/mcp

# With a header for token auth
claude mcp add --transport http secure-api https://api.example.com/mcp \
  --header "Authorization: Bearer ${TOKEN}"

# SSE — some services still expose only this
claude mcp add --transport sse asana https://mcp.asana.com/sse

# stdio — local processes
claude mcp add --env AIRTABLE_API_KEY=YOUR_KEY --transport stdio airtable \
  -- npx -y airtable-mcp-server
```

> **The `--` separator.** For stdio servers, `--` divides Claude's own options from the command that runs the server. Everything after it is passed through untouched. Getting this wrong is the most common first-time mistake.

WebSocket servers hold a persistent bidirectional connection, which suits servers that push events unprompted. They're configured with `claude mcp add-json` and `type: "ws"` — the `--transport` flag doesn't accept `ws`, and WebSocket supports header auth only, not OAuth.

In JSON config, `streamable-http` is an accepted alias for `http`, so configurations copied from a server's own documentation work unmodified.

### Three scopes

| Scope | Stored in | Available to |
|---|---|---|
| **local** (default) | `~/.claude.json`, under that project's path | You, in this project only |
| **project** | `.mcp.json` at the project root | Your team, via version control |
| **user** | `~/.claude.json` | You, across all projects |

```bash
claude mcp add --transport http stripe --scope project https://mcp.stripe.com
```

> ⚠️ **MCP "local scope" is not the same thing as general local settings.** MCP local-scoped servers live in `~/.claude.json` in your home directory. General local settings live in `.claude/settings.local.json` in the project. Same word, different file, easy to conflate.

### Authenticating

```bash
claude mcp login <name>     # OAuth from the shell
claude mcp logout <name>    # clear stored credentials
```

`/mcp` does the same interactively. The shell form matters because **in `claude -p` and Agent SDK runs there is no `/mcp` panel**, so the OAuth flow can't run there — complete the sign-in from an interactive session first.

### Checking your work

```bash
claude mcp list
```

Each server shows a health status: `✔ Connected`, `! Needs authentication`, `✘ Failed to connect`, or `⏸ Pending approval (run claude to approve)` for project-scoped servers waiting on you.

Two operational notes:

- **A cloned repository can't approve its own servers.** `enableAllProjectMcpServers` committed to `.claude/settings.json` is ignored in an untrusted folder — run `claude` in it and accept the workspace trust dialog first.
- **Reserved server names**: `workspace`, `claude-in-chrome`, `computer-use`, `Claude Preview`, `Claude Browser`. `claude mcp add` rejects them.

### Try it yourself

Add a server you already use, then confirm it:

```bash
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp
claude mcp list
```

If it shows `! Needs authentication`, run `claude mcp login sentry`.

### Common pitfalls

- **Forgetting `--` on a stdio server.** Claude Code tries to parse the server's arguments as its own.
- **Adding a team server at local scope.** Use `--scope project` so it lands in `.mcp.json` and gets checked in.
- **Expecting OAuth to work in `-p` mode.** Authenticate interactively first.

### Key takeaways

- Four transports: http, sse, stdio, ws — with `--` separating flags from the stdio command
- Three scopes: local (default, private), project (`.mcp.json`, shared), user (all your projects)
- `claude mcp list` gives you per-server health at a glance

---

## Chapter 4 — Tool search: how to add many servers without wrecking context

**What you'll learn:** why adding ten MCP servers doesn't cost what you'd expect.

### The concept

**Tool search is enabled by default.** Instead of loading every tool schema at session start, Claude Code loads only tool *names* and each server's instructions. Full schemas are deferred, and Claude searches for the ones a task needs. Only the tools it actually uses enter context.

The result: **Claude Code imposes no fixed per-server tool cap.** The practical limit is your context window budget, and deferral keeps that budget mostly free.

If you'd rather load upfront, `ENABLE_TOOL_SEARCH=auto` loads schemas when they fit within 10% of the context window and defers only the overflow.

Not supported on Microsoft Foundry deployments hosted on Azure, which reject it server-side; Claude Code detects the rejection and loads tools upfront for that deployment.

### If you're building a server

With tool search on, the **server instructions** field does the job a skill's `description` does — it's how Claude decides whether to go looking for your tools at all. Write it to say what category of tasks your tools handle and when Claude should search for them.

### Key takeaways

- On by default; only names and server instructions load at startup
- No per-server tool cap — context budget is the real limit
- Server instructions are the discovery signal, so write them like a skill description

---

## Chapter 5 — Channels: pushing events into a running session

**What you'll learn:** the inverse of a normal MCP server, and how to run one.

### The concept

A normal MCP server waits to be queried. **A channel is an MCP server that pushes events into your running session**, so Claude can react to things that happen while you're away from the terminal. Channels can be two-way: Claude reads the event and replies through the same channel.

Events only arrive while the session is open, so an always-on setup means running Claude in a background process or a persistent terminal.

### Constraints, before you plan around it

Channels are a **research preview**:

- Requires Anthropic authentication through claude.ai or a Console API key
- **Not available on Amazon Bedrock, Google Cloud's Agent Platform, or Microsoft Foundry**
- **Team and Enterprise organizations must explicitly enable them** (`channelsEnabled` in managed settings, or the Owner toggle in the claude.ai admin console)
- Each channel plugin requires [Bun](https://bun.sh)
- `--channels` doesn't appear in `claude --help` during the preview — the flag works anyway

Supported channels: **Telegram**, **Discord**, **iMessage**, plus **fakechat**, a localhost demo with nothing to authenticate.

### The quickstart worth doing first

```shell
/plugin install fakechat@claude-plugins-official
```

```bash
claude --channels plugin:fakechat@claude-plugins-official
```

Open `http://localhost:8787`, type a message, and watch it arrive in your terminal as an inbound line like `← fakechat · web: what's in my working directory?`. Claude works, calls the channel's `reply` tool, and the answer appears back in the browser.

You can pass several plugins to `--channels`, space-separated.

### Security model

Every approved channel keeps a **sender allowlist** — only IDs you've added can push messages, and everyone else is silently dropped.

- **Telegram and Discord** bootstrap it by pairing: message the bot, it replies with a code, you approve it with `/telegram:access pair <code>`, then lock it down with `/telegram:access policy allowlist`
- **iMessage** lets you text yourself with no setup; add others with `/imessage:access allow +15551234567`

> **Being in `.mcp.json` isn't enough to push messages.** A server also has to be named in `--channels`. That's a deliberate second gate.

Note that the allowlist also gates permission relay if the channel supports it — anyone who can reply through the channel can approve or deny tool use in your session. Only allowlist people you trust with that authority.

### The two shapes worth building

- **Chat bridge** — ask Claude something from your phone, and the work runs on your machine against your real files
- **Webhook receiver** — a CI failure, deploy event, or error-tracker alert lands in a session where Claude already has your files open and remembers what you were debugging

The second is the DevOps one, and it's what Track B of the capstone builds on.

### Try it yourself

Run the fakechat quickstart end to end. Then decide which of your real event sources — CI, alerting, deploys — would be worth wiring in.

### Common pitfalls

- **Expecting a channel to work on Bedrock or Vertex.** It won't.
- **Forgetting `--channels`.** The plugin can be installed and configured and still deliver nothing.
- **Leaving a session unattended without thinking about permission prompts.** If Claude hits one, the session pauses until you respond, unless the channel relays prompts.

### Key takeaways

- A channel is an MCP server that pushes in, not one you query
- Research preview: Anthropic auth only, no third-party providers, Bun required, org opt-in on Team/Enterprise
- Sender allowlist plus the `--channels` flag are two independent gates

---

## Chapter 6 — Hands-on: wiring MCP into your own stack

**Try:** connect a server for something you check daily — GitHub, Sentry, Linear — and ask Claude a question that requires live data:

```
What are the three most recent unresolved errors in Sentry, and which commits likely introduced them?
```

Notice that Claude queries the server *and* reads your code to answer, which neither source could do alone.

**Try:** add a server at project scope so it lands in `.mcp.json`, commit it, and confirm a teammate gets the `⏸ Pending approval` prompt rather than a silent connection.

**Try:** compare `/plugin install sentry@claude-plugins-official` against wiring the same server by hand with `claude mcp add`. The plugin route is usually less work.

**Try:** run the fakechat channel and push a message in from the browser.
