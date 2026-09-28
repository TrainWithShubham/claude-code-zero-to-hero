<!-- module: 4 | phase: 2 | format: deep-dive | last_verified: 2026-09-28 -->
# Module 4 — Getting Started with Claude Code

## Chapter 1: What Claude Code Is and Where to Use It

**Concept**

Claude Code is a terminal-native agentic coding tool. Instead of pasting code into a chat window and copying the answer back out, you run a `claude` command inside your project directory, describe what you want, and Claude reads your files, edits them, runs shell commands, and iterates — all inside the same session. It ships as a CLI first, and that CLI is the surface everything else is built on top of.

You are not limited to the terminal, though. The same engine is available in several places:

- **Terminal (CLI):** the primary interface, installed with a native installer, Homebrew, or WinGet.
- **IDE extensions:** VS Code and JetBrains IDEs run Claude Code inside an integrated panel, so it shows diffs and mode state alongside your editor instead of in a separate window.
- **Desktop app:** a standalone application with its own Code tab and a Cowork tab for longer-running sessions.
- **Web:** Claude Code on the web (claude.ai/code) runs sessions in the cloud, useful when you want to kick off a task without a local machine, or from a phone.
- **CI/CD:** GitHub Actions and GitLab integrations let Claude Code run as part of a pipeline, non-interactively.

The common thread across all of these is the agentic loop: Claude reads context, decides on an action (read a file, edit it, run a command), executes it through a tool, observes the result, and repeats until the task is done or it needs your input. Module 6 covers those built-in tools in depth; this module is about getting the tool installed and running at all.

**Try it yourself**

1. Open a terminal and check whether Claude Code is already installed: `claude --version`. If you see a version number followed by `(Claude Code)`, skip to Chapter 2.
2. Install it. On macOS, Linux, or WSL:
   ```
   curl -fsSL https://claude.ai/install.sh | bash
   ```
   On Windows PowerShell:
   ```
   irm https://claude.ai/install.ps1 | iex
   ```
   Native installs update themselves in the background, so you generally don't need to think about upgrades again.
3. Run `claude --version` again to confirm the install worked.
4. If you use VS Code or a JetBrains IDE, open the Claude Code extension from the marketplace so you can compare the in-editor experience with the terminal later in this module.

**Common pitfalls**

- Installing through Homebrew and expecting it to auto-update like the native installer does — Homebrew casks don't auto-update; you have to run `brew upgrade claude-code` yourself.
- Forgetting that on native Windows, Claude Code needs Git for Windows installed to use the Bash tool; without it, Claude Code falls back to PowerShell as the shell tool, which changes some of the commands it will choose to run.
- Assuming the terminal is the only "real" way to use Claude Code and never trying the IDE extension — for day-to-day editing, seeing diffs inline in your editor is often faster than reading them in a terminal pane.

**Key takeaways**

- Claude Code is one engine exposed across the terminal, IDE extensions, a desktop app, the web, and CI/CD — pick the surface that fits the task, not just the one you installed first.
- Install with the native installer (macOS/Linux/WSL: `curl -fsSL https://claude.ai/install.sh | bash`; Windows PowerShell: `irm https://claude.ai/install.ps1 | iex`), Homebrew, or WinGet, and verify with `claude --version`.
- The agentic loop — read, act, observe, repeat — is the same everywhere Claude Code runs; the interface just changes how you see it happen.

## Chapter 2: Logging In and Choosing Your Model

**Concept**

Claude Code needs an account before it will do anything. The first time you run `claude` in a project directory, it prompts you to authenticate in your browser. You can log in with:

- A **Claude subscription** — Pro, Max, Team, or Enterprise. This is the recommended path for most learners and is what most of this course assumes.
- A **Claude Console** account — API access billed by usage, with pre-paid credits. On first login, Claude Code automatically creates a "Claude Code" workspace in the Console for cost tracking.
- An **enterprise cloud provider** — Amazon Bedrock, Google Cloud's Agent Platform, or Microsoft Foundry, if your organization routes Claude Code through one of these.

If you've already set the `ANTHROPIC_API_KEY` environment variable, Claude Code skips the login prompt and asks you to approve the key instead. Once you're logged in, credentials are stored locally, so you won't be asked again until you explicitly switch accounts with `/login` inside a running session.

Once logged in, Claude Code picks a model for you based on your account type — Opus for Max and Enterprise plans, Sonnet for Pro — but you're not stuck with the default. Model families you'll see referenced are Opus (strongest reasoning, higher cost), Sonnet (the default for daily coding work), and Haiku (fastest, cheapest, best for simple or high-volume tasks). You switch models with the `/model` command inside a session, or by passing `--model` at startup, for example `claude --model opus`. Selecting a model from the `/model` picker with Enter saves it as your default for future sessions; pressing `s` switches it for the current session only.

**Try it yourself**

1. Run `claude` in an empty test directory and complete the login flow in your browser.
2. Inside the session, type `/model` and look at the picker. Note which model is marked as your account's default.
3. Try `/model sonnet` and `/model haiku` to see how the picker responds, then pick the one you want to keep working with.
4. Ask Claude a one-line question ("what files are in this directory?") to confirm the session actually responds — this proves login and model selection both worked.

**Common pitfalls**

- Setting `ANTHROPIC_API_KEY` in your shell profile out of habit, then being confused when `/login` doesn't behave the way the docs describe — an API key in the environment changes the auth flow, so remove it if you want the subscription login path.
- Treating the model picker's Enter and `s` options as interchangeable — Enter changes your saved default for every future session, `s` only affects the one you're in. Picking the wrong one means your next session silently starts on a different model than you expected.
- Assuming a higher-tier model is always the right choice for every task. Haiku is often the better pick for small, repetitive edits, and using Opus everywhere just burns through usage faster for no real quality gain on simple work.

**Key takeaways**

- Claude Code supports login via Claude subscriptions (Pro/Max/Team/Enterprise), Claude Console (API credits), and enterprise cloud providers (Bedrock, Google Cloud's Agent Platform, Microsoft Foundry).
- `/login` re-authenticates or switches accounts inside a running session; an `ANTHROPIC_API_KEY` environment variable bypasses the browser login entirely.
- Switch models with `/model` (interactive picker) or `--model` at startup; Sonnet is the practical daily driver, Opus is for harder reasoning, Haiku is for fast and simple work.

## Chapter 3: Running Models Locally and Free with Ollama

**Concept**

Ollama — a separate, third-party tool for running open-weight models on your own machine — ships its own integration for launching Claude Code against a locally-hosted model, documented at `docs.ollama.com/integrations/claude-code`. This is not an Anthropic-built feature: Claude Code itself still only knows how to speak the Anthropic API, but Ollama runs a local server that speaks that same API shape, so Claude Code can be pointed at `localhost` instead of Anthropic's servers without knowing the difference.

The quick path is a single command, `ollama launch claude`, which handles setup for you. The manual version makes the mechanism clear: you install Claude Code as normal, then set three environment variables before starting a session —

```
export ANTHROPIC_AUTH_TOKEN=ollama
export ANTHROPIC_API_KEY=""
export ANTHROPIC_BASE_URL=http://localhost:11434
```

`ANTHROPIC_BASE_URL` is the important one — it redirects every request Claude Code would normally send to Anthropic toward your local Ollama server instead. From there you run `claude --model <model-name>` (for example `qwen3.5`) and Claude Code behaves as it always does: chat, file edits, command execution, tool calling — against whichever open-weight model Ollama is serving.

This is genuinely free once the model is downloaded, since nothing leaves your machine and there's no per-token billing. It is not a drop-in replacement for the hosted experience, though: Ollama's own docs note that web search and some of the finer tool controls need extra setup, and cloud-hosted models routed through Ollama have reduced feature support compared to talking to Claude directly. Pick a model with enough context for your repository — the guidance is 64k+ context for anything beyond a small project — since local models vary widely in how much they can hold in one session.

For this course, a Claude subscription is the recommended default — it's the path every module assumes and where results are most consistent. Treat Ollama as a fallback for practicing the mechanics without a subscription, not an equivalent setup: it needs real local compute (a decent GPU or a lot of RAM to run a model with enough context to be useful) and responses are noticeably slower than a hosted session, since you're waiting on your own machine's inference speed instead of Anthropic's infrastructure.

**Try it yourself**

1. Install Ollama separately (it's not bundled with Claude Code) and pull a model with enough context for a small project.
2. Try the quick path first: `ollama launch claude`.
3. Then do it manually to see what's actually happening — set the three environment variables above in your shell, then run `claude --model qwen3.5` (or whichever model you pulled) in a test project directory.
4. Ask Claude to read a file and make a small edit. Compare the experience — latency, edit quality — against a normal hosted session from Chapter 2.

**Common pitfalls**

- Forgetting to unset `ANTHROPIC_BASE_URL` and `ANTHROPIC_AUTH_TOKEN` afterward — they stay in your shell environment and silently redirect your *next* `claude` session to Ollama too, even if you meant to go back to your subscription.
- Expecting full feature parity with a hosted Claude session. Web search and some tool controls need extra setup on the Ollama side, and this path is documented by Ollama, not Anthropic — check both docs if something behaves unexpectedly.
- Picking a model with too little context for the repository you're working in — Claude Code will still run, but it will lose track of earlier parts of the session faster than you'd expect from a hosted model.

**Key takeaways**

- Ollama, not Anthropic, documents and maintains this integration — it works because Ollama's local server mimics the Anthropic API shape that Claude Code already speaks.
- The fast path is `ollama launch claude`; the manual path is three environment variables (`ANTHROPIC_AUTH_TOKEN`, `ANTHROPIC_API_KEY=""`, `ANTHROPIC_BASE_URL=http://localhost:11434`) followed by `claude --model <name>`.
- It's genuinely free to run once a model is downloaded, but expect reduced feature support compared to a hosted Claude subscription — verify current capabilities at `docs.ollama.com/integrations/claude-code` since this is a third-party integration that can change independently of Claude Code itself.

<!-- nav -->
[← Module 3: Understanding Agentic AI Systems](module-03-agentic-ai-systems.md) | [Module 5: Claude Code Setup, Configuration and Memory →](module-05-setup-configuration-memory.md)
