<!-- module: 10 | phase: 3 | format: deep-dive | last_verified: 2026-09-15 -->
# Module 10 — MCP Foundations

By the end of this module you can explain how MCP connects Claude Code to external tools, and find, install, and use MCP servers in real workflows.

## MCP Clients, Servers and External Tools

**Concept**

The Model Context Protocol (MCP) is an open standard for connecting an AI agent to external systems — databases, SaaS products, internal APIs, documentation sites — without writing custom integration code for each one. Claude Code acts as an **MCP client**. An **MCP server** is a small program that exposes a set of capabilities (usually tools, sometimes resources or prompts) over a defined protocol. When you connect Claude Code to an MCP server, the tools that server exposes become available to Claude in the same way built-in tools like Read or Bash are — Claude can call them, see their results, and reason about what to do next.

The value isn't the protocol itself, it's what it removes: without MCP, giving an agent access to, say, your ticketing system means writing a bespoke plugin or wrapper script for that one integration. With MCP, anyone who has written an MCP server for that ticketing system can plug it into any MCP-compatible client — Claude Code included — with a single command. The protocol standardizes the handshake (what tools exist, what arguments they take, how results come back) so the ecosystem of servers keeps growing without each client needing custom code per integration.

MCP servers can run in different ways:

- **stdio** — a local process Claude Code starts and talks to over standard input/output. Good for local tools and servers that need direct filesystem or subprocess access.
- **HTTP** — a remote server Claude Code talks to over HTTP, often the pattern for hosted SaaS integrations that need OAuth.
- **SSE** — Server-Sent Events, another remote transport used by some servers.

Configured servers can live at three scopes: **local** (just your machine, just this project), **project** (checked into `.mcp.json` in the repo root so your whole team gets the same servers), or **user** (available across every project you work in). Project scope is what makes MCP a team capability rather than a personal one — commit `.mcp.json` and everyone who opens the repo in Claude Code gets the same tools.

**Try it yourself**

1. Run `/mcp` in an active Claude Code session to see what servers, if any, are currently connected — this works even with zero servers configured and shows you the panel you'll use throughout this module.
2. Run `claude mcp list` from a terminal to see the same information outside a session.
3. Open (or create) `.mcp.json` in a project you don't mind experimenting in, and look at the shape described later in this module — you don't need a real server yet, just get comfortable with where the file lives.

**Common pitfalls**

- Treating MCP servers as automatically trustworthy. A server can expose a tool that reads your filesystem, calls out to the internet, or has full access to whatever credentials it's configured with. Only connect servers you or your organization trust, the same way you'd vet a new dependency.
- Assuming local (default) scope is shared with the team. It isn't — if you want a server everyone on the project uses, add it with `--scope project` so it lands in `.mcp.json`.
- Forgetting that a stdio server is a process Claude Code spawns. If the underlying command (say, an `npx` package) isn't installed or reachable, the server will fail to start and its tools simply won't appear — check `/mcp` for connection status when a tool you expect is missing.

**Key takeaways**

- MCP standardizes how an agent talks to external tools — Claude Code is the client, the integration is the server.
- Servers connect over stdio (local process), HTTP, or SSE (remote), and are stored at local, project, or user scope.
- `.mcp.json` at project scope is how a team shares the same set of external tools.
- Trust a server before connecting it — its tools run with real access to whatever it's wired up to.

## Finding and Installing an MCP Server

**Concept**

You add an MCP server to Claude Code with the `claude mcp add` command, run from your terminal (not inside a session). Its shape depends on the transport:

```bash
# Remote server over HTTP
claude mcp add --transport http <name> <url>

# Remote server over HTTP, with an auth header
claude mcp add --transport http <name> <url> --header "Authorization: Bearer <token>"

# Local server that Claude Code starts as a process (stdio)
claude mcp add <name> -- <command> [args...]

# From a raw JSON config, useful for transports like SSE or WebSocket
claude mcp add-json <name> '<json>'
```

Add `--scope project` to share the server with your team via `.mcp.json`, or `--scope user` to make it available in every project you open. Omit `--scope` and it defaults to local — just this machine, just this project.

Once added, manage servers with:

```bash
claude mcp list              # see everything configured
claude mcp get <name>        # details on one server
claude mcp remove <name>     # take it out of your config
claude mcp login <name>      # OAuth sign-in for servers that need it
claude mcp logout <name>
```

Inside a session, `/mcp` shows connection status and lets you reauthenticate a server whose token expired, without leaving Claude Code.

Where do you find servers worth adding in the first place? A few sources:

- **Anthropic's connector directory** at claude.ai/directory lists reviewed connectors you can add.
- **Vendor documentation.** Companies that publish an MCP server for their own product (a database vendor, a project-management tool, a monitoring platform) usually document the exact `claude mcp add` invocation on their own site — copy that rather than guessing at flags.
- **Public GitHub repositories.** Community and vendor-maintained servers are often just open-source repos you point a stdio command at (for example, via `npx` or `uvx`). Because anyone can publish one, check who maintains it and how recently it's been updated before trusting it with real credentials.

**Try it yourself**

1. Pick a tool you already use day to day (a docs site, a ticket tracker, a database) and search for "<tool name> MCP server" to see whether an official or community server exists.
2. If you find one with a documented `claude mcp add` command, run it in a low-stakes test project first, then check `/mcp` to confirm it connected.
3. Try `claude mcp add --scope project` once, purely to see the `.mcp.json` file it writes — then decide whether you actually want that server shared with a team before committing it.

**Common pitfalls**

- Copying an add command from an old blog post instead of the server's own current documentation — MCP server flags and package names change, and a stale command fails silently or installs the wrong version.
- Adding a server at user scope by habit, then being surprised it's active (and consuming its tool-call budget) in unrelated projects. Scope it to where you actually need it.
- Not checking `claude mcp get <name>` after a failed connection — it will usually tell you exactly why (missing auth, wrong command, unreachable URL) instead of you guessing.

**Key takeaways**

- `claude mcp add` (with `--transport http`, or `add-json` for other transports) is how you connect a server; `--scope` controls who else sees it.
- `claude mcp list`, `get`, `remove`, `login`, and `logout` cover the day-to-day lifecycle; `/mcp` inside a session shows live status.
- Find servers through Anthropic's connector directory, vendor docs, or vetted community repositories — and read the maintainer's own install instructions rather than a stale recap.

## AWS Docs and Hacker News MCP Workflows

**Concept**

Two worked examples show how MCP changes a real workflow, one with an official server and one where you should stay more cautious.

**AWS Documentation.** AWS maintains an official, open-source set of MCP servers in the `awslabs/mcp` GitHub repository, including servers that expose AWS documentation search and retrieval as tools. Once connected, instead of you tabbing over to a browser to check the exact syntax of an IAM policy condition or an S3 lifecycle rule, Claude can search AWS's own documentation directly, pull the relevant page, and cite it while it writes your Terraform or your CLI command. This matters most for anything version-specific or region-specific in AWS's service surface, where guessing from training data is a common source of subtly wrong infrastructure code. Because this server is officially maintained by AWS, it's a reasonable one to trust with read access to public documentation — but always check `awslabs/mcp` for the current server name and install command before adding it, since AWS has iterated on this family of servers (for example, alongside a documentation-specific server, AWS also publishes a broader "AWS Knowledge" server that indexes docs, blog posts, and architecture guidance together).

**Hacker News.** Several community-maintained MCP servers expose the public Hacker News API as MCP tools — fetching top stories, searching discussions, pulling comment threads. A workflow here looks like asking Claude to pull the current top stories in a topic you care about, summarize the discussion threads, and flag anything relevant to a project you're working on, all without you opening a browser tab. Unlike the AWS documentation server, there is no single official Hacker News MCP server maintained by Y Combinator or Anthropic — what exists is a handful of independently maintained open-source projects of varying activity and quality. Treat this the way you'd treat any other third-party dependency: check the repository's maintenance activity, read what data it actually touches (the public HN API has no authentication, so the risk surface is smaller than most servers), and confirm its current install command from its own README rather than assuming a specific package name will still be correct by the time you read this.

**Try it yourself**

1. If you have AWS documentation needs in your current work, connect an AWS documentation MCP server following the current instructions in `awslabs/mcp`, then ask Claude a question you'd normally answer by searching AWS docs yourself — compare the answer against the cited page.
2. If you're curious about the Hacker News pattern, search for an actively maintained Hacker News MCP server, read its README fully before adding it, and try asking Claude to summarize a topic's current front-page discussion.
3. Either way, run `/mcp` afterward and look at what tools the server actually exposed — matching that against what you expected is a good habit for any new server.

**Common pitfalls**

- Assuming every "official-sounding" server name is actually maintained by the company it references. For Hacker News specifically, there is no canonical official server — verify independently rather than trusting a plausible-sounding package name.
- Pointing an AWS documentation server at write-capable AWS credentials by mistake. A documentation server should only need to read public docs; if an install path asks for AWS account credentials, stop and check what it's actually for.
- Relying on either server's output as ground truth without spot-checking citations, especially for fast-moving services or pricing pages — Claude Code ships weekly, and so do the docs a documentation MCP server retrieves.

**Key takeaways**

- AWS publishes an official, open-source MCP server family for its documentation in `awslabs/mcp` — a low-risk way to keep Claude's AWS answers grounded in current docs.
- Hacker News MCP servers exist and work well for the described workflow, but they're community-maintained, not official — verify the specific project before trusting it.
- The general pattern — connect a documentation or content source as an MCP server so Claude can search it directly instead of you doing it by hand — generalizes well beyond these two examples.

<!-- nav -->
[← Module 9: Slash Commands, Skills and Sub-Agents](module-09-slash-commands-skills-subagents.md) | [Module 11: Advanced Automation with Sub-Agents and Plugins →](module-11-automation-subagents-plugins.md)
