<!-- module: 1 | phase: 1 | format: reference | last_verified: 2026-09-15 -->

# Module 1 — Understanding Claude and How It Works

## From AI to Generative and Agentic AI

Claude Code sits at the end of a fairly short but fast-moving evolution in artificial intelligence. Knowing where it came from makes it easier to understand why it behaves the way it does — why it can hold a conversation about your codebase, and why it can also go off and edit files on its own.

- **Rule-based AI** came first: systems that followed hand-written if-this-then-that logic. A rule-based system only does what a human explicitly programmed it to do — it can't generalize to a situation its author didn't anticipate.
- **Machine learning** replaced hand-written rules with patterns learned from data. Instead of programming every case, you show the system many examples and it learns the mapping between input and output itself. This is what powers things like spam filters and recommendation engines.
- **Generative AI** is a subset of machine learning built on large language models (LLMs). Instead of just classifying or predicting a label, these models generate new content — text, code, images — by predicting, one token at a time, what should come next given everything before it. Claude, GPT, and Gemini are all generative AI in this sense.
- **Agentic AI** is the newest layer on top of generative AI. A generative model that only replies in a chat window is still passive — it produces an answer and stops. An agentic system pairs that same LLM with the ability to *act*: it can call tools (run a command, edit a file, search the web, query a database), observe the result, and decide what to do next in a loop, without a human approving every single step.
- The distinction that matters for this course: a chatbot generates a response; an agent generates a response, executes on it, checks whether it worked, and adjusts. Claude Code is an agent in this sense — it doesn't just tell you what code to write, it writes it, runs it, and reads the output.
- This shift from "generate an answer" to "pursue a goal through a loop of actions" is why agentic AI needs new skills from the people using it: prompting well is necessary but no longer sufficient — you also need to think about permissions, context, tool access, and how to keep an autonomous loop pointed at the right goal. That's most of what the rest of this course covers.

## Claude's Models and Product Ecosystem

### The model family

Anthropic ships Claude as a family of models, each tuned for a different point on the speed-versus-intelligence curve. As of this writing, the current lineup is:

| Model | Best for | Relative speed | Context window |
|---|---|---|---|
| Claude Haiku 4.5 | Fast, lightweight tasks where near-frontier intelligence at low latency matters most | Fastest | 200K tokens |
| Claude Sonnet 5 | The default choice for most work — the best balance of speed and intelligence | Fast | 1M tokens |
| Claude Opus 5 | Complex agentic coding and enterprise-grade work | Moderate | 1M tokens |
| Claude Fable 5.1 | Demanding reasoning and long-horizon agentic tasks, or when Opus 5 at high effort still isn't enough | Slower | 1M tokens |

A few things worth understanding about this table rather than memorizing the exact numbers, since Anthropic updates models and pricing regularly:

- All current models take text and image input, produce text output, and support tool use — the capability that makes agentic behavior possible in the first place.
- "Adaptive thinking" is a feature of current models: the model decides for itself how much internal reasoning to spend on a problem, steered by an effort setting, rather than you manually turning "extended thinking" on or off as with older model generations.
- Model names carry a generation number (e.g., `4.5`, `5`, `5.1`). Each model has a pinned API identifier — treat exact model IDs as something to look up at build time rather than something to memorize, since Anthropic retires and replaces them on a published schedule.
- Picking a model is a real engineering decision, not just a preference: a fast, cheap model is often the right call for simple, repetitive edits, while a slower, more capable model earns its cost on multi-step reasoning or unfamiliar codebases. Claude Code lets you switch models mid-session for exactly this reason.

### The product ecosystem

Claude isn't one app — it's a family of surfaces that all connect to the same underlying models. Knowing what each one is for helps you pick the right entry point for a task:

- **claude.ai** — the consumer chat interface (web and mobile apps for iOS and Android). This is where you'd go for general-purpose conversation, research, writing, and analysis that doesn't need direct access to a codebase or your local machine.
- **Claude Developer Platform (the Claude API)** — the way developers integrate Claude programmatically into their own applications, sending messages to a model and getting completions back. This is the foundation everything else in the ecosystem is built on.
- **Claude Code** — the focus of this course: an agentic coding tool that reads your codebase, edits files, runs commands, and integrates with your development tools. It's available on several surfaces that all share the same underlying engine and the same project configuration (CLAUDE.md, settings, MCP servers):
  - **Terminal** — the full CLI, installed via a native installer, Homebrew, WinGet, or Linux package managers.
  - **IDE extensions** — for VS Code (and VS Code–based editors like Cursor) and JetBrains IDEs (IntelliJ IDEA, PyCharm, WebStorm, and others), giving you inline diffs and in-editor context sharing.
  - **Desktop app** — a standalone application for reviewing diffs visually, running multiple sessions side by side, and scheduling recurring tasks, available for macOS and Windows.
  - **Web** — running sessions in the browser at claude.ai/code, with no local setup, including from the Claude mobile app.
- **Claude Agent SDK** — for developers who want to build their own custom agents powered by the same tools and orchestration model that Claude Code uses internally, with full control over tool access and permissions. This is covered in depth in Module 14.
- Beyond these core surfaces, Claude Code also integrates with CI/CD pipelines (GitHub Actions, GitLab CI/CD), team chat (Slack), and browser-based debugging (Chrome) — these are covered later in the course as they become relevant.

Picking the right surface is itself a skill: a quick one-off question belongs in claude.ai, a task against your local repo belongs in the terminal or an IDE extension, and something you want to kick off and check on later fits the web or desktop surfaces better. The rest of this course is almost entirely about Claude Code, but it's worth knowing where it sits relative to the rest of the family.

<!-- nav -->
[Module 4: Getting Started with Claude Code →](module-04-getting-started.md)
