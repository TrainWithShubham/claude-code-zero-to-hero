<!-- module: 4 | phase: 2 | format: deep-dive | last_verified: 2026-08-12 -->

# Module 4 — The Agentic Loop and Built-In Tools

Everything in this module is written for you to keep. Read it before you watch if you like reading first, or after if you'd rather follow along live and review later. Either way, by the end you should be able to explain *why* Claude Code behaves the way it does, not just repeat what buttons to press.

---

## Set up your practice environment

You'll get much more out of this module if you follow along on your own machine.

**What you need:**
- Claude Code installed and logged in (Module 3)
- A small practice repo with:
  - 3–4 small source files
  - One test that currently **fails**
  - One small feature that's **missing** — for example, a route with no input validation

If you don't have a repo handy, clone the course's `zth-demo-api` repo (link in the course resources). It's a tiny REST API built so these demos behave the same way on your machine as they do on screen.

Keep the same repo open for all six chapters — they build on each other.

---

## Chapter 1 — How Claude Code works: reasoning, tool use, iteration

**What you'll learn:** why Claude Code behaves like an agent instead of a chatbot, and the three-phase loop underneath everything it does.

### The concept

A chatbot answers your question and waits. Claude Code doesn't wait — given a task, it works through three phases that blend together:

1. **Gather context** — reading files, searching the codebase, understanding what's there
2. **Take action** — editing files, running commands, making the change
3. **Verify results** — running tests, checking output, confirming the change worked

None of this is a fixed script. Every tool call returns new information, and that information shapes what Claude does *next*. If a test fails after an edit, Claude sees the failure and tries again — you didn't have to ask twice. That's what "agentic loop" means, and the phrase recurs throughout the course, so it's worth having it click here.

You can interrupt the loop at any point and redirect it. It isn't a black box you're stuck watching — it's closer to pairing with someone who takes initiative but still listens when you say "wait, not like that."

### Try it yourself

```
Fix the failing test in this repo.
```

**What should happen:**
- Claude reads the test file, then the source file it's testing
- It explains what's wrong in a sentence or two before touching anything
- It edits the source file
- It re-runs the test to confirm the fix worked

If it stops after editing without re-running:

```
Now run the test again to confirm it passes.
```

That's a normal way to steer the loop — you're not doing it wrong if you have to nudge.

### Common pitfalls

- **Writing one giant, over-detailed prompt describing the entire fix.** You don't need to. The loop discovers what it needs as it goes. Start smaller than feels natural.
- **Assuming a pause means it's stuck.** A pause is very often a permission prompt waiting on you, not a crash. More on permissions in Module 5.

### Key takeaways

- The loop is gather context → take action → verify results, repeating as needed
- Tool results feed the next decision — nothing is pre-scripted
- You can and should interrupt and redirect

---

## Chapter 2 — Read, Write, and Edit: file operation tools

**What you'll learn:** three different tools for touching files, and why Claude picks one over another.

### The concept

- **Read** — opens a file so Claude can see its contents, with line numbers. The most direct way to look at something.
- **Edit** — makes a targeted change. It matches exact existing text before changing it, which is why Claude usually reads a file first. The edit is surgical, not a rewrite.
- **Write** — overwrites an entire file. Good for creating new files; risky on files with content you care about, since it replaces everything.

**Read, Edit, and Write don't prompt for permission on files inside your working directory** — part of why the loop feels fast. They do still prompt for paths outside your project.

### The read-before-edit rule

Claude generally has to read a file in the current conversation before editing it. The exact behavior depends on your model and version:

- **Claude Opus 4.6, Haiku 4.5, and older models always require the read.**
- **Newer models can edit an unread file** when reading it wouldn't need a permission prompt and the Read tool is available. This relaxation requires Claude Code v2.1.208 or later.
- **Viewing a file with Bash also counts** — `cat`, `head`, `tail`, `sed -n 'X,Yp'`, `grep`, `egrep`, or `fgrep`, on a single file, with no pipes or redirects. Piped output doesn't count.

So if a teammate on Haiku sees Claude insist on reading a file yours edits directly, nothing is broken — that's the model difference.

Permission rules also compose across these tools: an `Edit` allow rule grants read access to the same path, and a `Read` deny rule blocks Edit and Write there too, including creating a new file. More in Module 5.

### Try it yourself

```
Read app.py and tell me what the /users endpoint does.
```
```
Add input validation to the POST /users route so it rejects requests with no email field.
```

**What should happen:**
- First prompt: a clean summary, no permission prompt
- Second prompt: Claude reads the relevant file if it hasn't already, then makes a precise change. Look at the diff — it's not rewriting the whole file, just the lines that changed.

Then see Write in contrast to Edit:

```
Create a new validators.py file with a reusable email-validation function, and use it in the /users route.
```

### Common pitfalls

- **Wondering why Edit prompted but Read didn't.** Read, Grep, and Glob don't change anything, so they're lower-risk by default. Anything that writes to disk gets more scrutiny.
- **Letting Claude use Write where Edit was the better fit.** A full rewrite costs more and can lose unrelated changes you made by hand. If you notice this on a file you've hand-edited, say so in your prompt.

### Key takeaways

- Read = look, Edit = targeted change, Write = full overwrite
- Edit's precision is *why* it usually reads first
- File tools inside your project are low-friction by design

---

## Chapter 3 — Grep and Glob: searching large codebases

**What you'll learn:** how Claude finds things without you telling it exactly where to look.

### The concept

- **Glob** answers "which *files* match this pattern?" — it works on names and paths, not contents.
- **Grep** answers "which files *contain* this text?" — it searches content.

Neither prompts for permission inside your working directory, which is exactly why Claude can explore quickly without constantly interrupting you. On a real project — not a four-file demo — this is the difference between guessing which file to open and finding the right one first try.

One quirk: Claude sometimes reaches for Bash `grep` or `find` instead of the dedicated tools, even though the dedicated tools are usually better (structured results, no permission prompt). If you notice it, that's known behavior, not a broken setup.

### Try it yourself

```
Find every file in this repo that references the /users route.
```
```
Find all test files in this project.
```

**What should happen:**
- First → a Grep search, results with file name and line context
- Second → a Glob search on a filename pattern like `**/test_*.py` or `**/*.test.js`

To see the tool names directly:

```
Did you use Grep or Bash for that last search?
```

### Common pitfalls

- **Reaching for your own terminal habits.** If your instinct is "I'd just grep this myself," fine — but notice Claude gets the same result *and* skips the permission prompt using its dedicated tool.

### Key takeaways

- Glob = filename patterns, Grep = content search
- Both are fast and low-friction inside your project
- Claude occasionally defaults to Bash for search — a quirk, not a bug

---

## Chapter 4 — The Bash tool and the sandboxed Bash tool

**What you'll learn:** how Claude runs shell commands, and how sandboxing changes what that means for your safety.

### The concept

Bash handles everything the dedicated tools don't: installing packages, running tests, git operations, build tools. There's no dedicated tool for `npm install`, and there doesn't need to be.

Bash is permission-gated by default, with one nuance: it runs a **built-in set of read-only commands without prompting**. That's why some commands sail through and others stop.

Here's the part that matters for how you think about autonomy: **without sandboxing, a Bash command Claude runs has the same access your own terminal session does.** That's the plain truth, not a scare tactic.

**Sandbox mode** changes that with real OS-level isolation:

- **macOS** uses the built-in Seatbelt framework — nothing to install
- **Linux and WSL2** need two packages: `bubblewrap` (filesystem isolation) and `socat`
- **No native Windows support** — run Claude Code inside WSL2

```bash
sudo apt-get install bubblewrap socat     # Debian / Ubuntu
sudo dnf install bubblewrap socat         # Fedora / RHEL
```

An optional seccomp filter adds Unix-domain-socket blocking: `npm install -g @anthropic-ai/sandbox-runtime`.

> **On Ubuntu 24.04 and later**, the default AppArmor policy prevents bubblewrap from creating the user namespaces it needs. You'll need to allow it before the sandbox works. The `/sandbox` **Dependencies** tab tells you exactly what's missing — and the check runs at startup, so restart Claude Code after installing anything.

### Filesystem isolation: writes and reads are not symmetric

This is the part most people get wrong.

**Writes are tightly bounded.** Commands get read and write access to your working directory, its subdirectories, and the session temp directory. They cannot modify anything outside that without explicit permission — including `~/.bashrc` and system binaries in `/bin/`.

**Reads are not.** The default is **read access to the entire computer**, minus a few denied directories. That default still allows reading credential files such as `~/.aws/credentials` and `~/.ssh/`.

If you work with cloud credentials, kubeconfigs, or SSH keys — and you do — treat "sandboxed" as meaning *write*-protected, not read-protected, until you configure otherwise. Block credential reads with the `sandbox.credentials` setting or by adding paths to `denyRead`.

How credential protection behaves differs by platform:

- **Linux and WSL2** — sandboxed commands read a *sentinel copy* of the file with the secret replaced by a placeholder, and the sandbox proxy substitutes the real value on egress. Tools that authenticate with the file still work.
- **macOS** — the file can't be read at all. Tools that authenticate with it won't work inside the sandbox.

### Network isolation

Configured separately from the filesystem. **No domains are pre-allowed.** The first time a command needs a new domain you get a prompt; approving allows that host for the rest of the session. Pre-allow domains with the `allowedDomains` setting to skip the prompt entirely.

The practical payoff of all this: once sandboxing is on, commands can run *without* asking permission every time, because the operating system — not you — enforces the boundary. That's a meaningfully different trust model than clicking "approve" repeatedly.

### Reading large command output

Claude Code reads back up to **30,000 characters** of a command's output by default, configurable up to a hard ceiling of **150,000** via `BASH_MAX_OUTPUT_LENGTH`.

Output larger than roughly 30,000 characters isn't lost — it arrives as **a path to a file in the session directory plus a short preview**, and Claude reads or searches that file when it needs the rest. Raising `BASH_MAX_OUTPUT_LENGTH` widens the read-back window but doesn't change that inline ceiling.

### Try it yourself

```
Run the full test suite and show me the results.
```

Without sandboxing, expect a permission prompt unless you've allow-listed the command. Now turn it on:

```
/sandbox
```

The panel has three tabs — **Mode** (how sandboxed commands are approved), **Overrides** (whether commands that fail under the sandbox may fall back to running unsandboxed), and **Dependencies** on Linux when something's missing. Run the same test command again and notice whether it interrupts you.

### Common pitfalls

- **Confusing sandboxing with permission modes.** Sandboxing only affects the Bash tool. It doesn't replace the permission system for Read, Write, and Edit — the two work side by side.
- **Assuming sandboxing is full container isolation.** It's OS-level process confinement, not a virtual machine. For genuinely untrusted code you want dev containers, Docker, or VMs — Module 13.
- **Assuming a sandbox protects your secrets by default.** It doesn't. Configure credential protection explicitly.

### Key takeaways

- Bash covers everything the dedicated tools don't
- Sandboxing isolates filesystem and network separately, using OS-level primitives
- Writes are confined to your working directory; **reads default to the whole machine**
- Sandboxed commands can skip permission prompts because the OS enforces the boundary

---

## Chapter 5 — Understanding the context window and prompt caching

**What you'll learn:** why long sessions slow down or get expensive, and the habits that keep them fast and cheap.

### The concept

Claude doesn't "remember" between messages the way a person would. Every message is a fresh request, and Claude Code re-sends the relevant context each time. Caching is what makes that affordable: if the start of a new request matches something processed recently, that part is reused instead of reprocessed.

Two things break the cache, worth remembering as a pair:

1. **Switching models mid-session** — triggers a slower, uncached turn
2. **Editing CLAUDE.md or memory files mid-session** — invalidates the cached project-context layer

`/compact` replaces conversation history with a summary once things get long. It isn't free, but because the system prompt, tools, and CLAUDE.md prefix stay cached, a compaction costs far less than the raw size of your conversation suggests.

**The habit worth building:** pick your model and effort level at the start of a session, avoid CLAUDE.md edits mid-task, and save `/compact` for natural breaks between tasks. That single habit is the biggest lever on both speed and cost.

`/context` is the diagnostic — an interactive breakdown of exactly what's filling your context window right now, so you're not guessing.

### Try it yourself

```
/context
```

Do a few real turns of work, then:

```
/compact
```

**What to notice:** `/context` shows the breakdown before you compact — system prompt, tools, CLAUDE.md, conversation. After `/compact` the history is shorter, but nothing is deleted; it's summarized, and the summary carries forward.

### Common pitfalls

- **Switching models "just to try something" mid-session.** Occasionally fine, but it costs a slower, uncached turn. Make it deliberate.
- **Editing CLAUDE.md out of habit during a session.** Treat CLAUDE.md edits as a between-session activity if you want a cache-friendly session.

### Key takeaways

- Every message is a new request; caching makes repeated context affordable
- Model switches and CLAUDE.md edits both break the cache
- `/context` diagnoses, `/compact` treats — in that order

---

## Chapter 6 — Tools reference recap

**What you'll learn:** a consolidated view of this module, and where the tools you haven't met yet fit in.

### Recap

This module covered **Read, Write, Edit, Grep, Glob**, and **Bash** (sandboxed and not). Two more categories get their own modules:

- **Subagents** (the Agent tool) — Module 10
- **MCP tools** (external services connected to Claude) — Module 9

One rule to carry forward: file-access tools (Read, Grep, Glob) generally don't prompt inside your working directory. Anything that writes, or reaches outside your project, gets more scrutiny by design.

### Try it yourself

```
Which tools did you use to fix that test earlier, and in what order?
```

Claude can usually reconstruct its own tool-call sequence from the session — a good way to see the whole module's toolkit applied to one real task, end to end.

### Key takeaways

- Six tools this module: Read, Write, Edit, Grep, Glob, Bash
- Two more families — subagents and MCP — are coming later
- Everything here serves one thing: the gather → act → verify loop from Chapter 1

---
<!-- nav -->

[← Installing Claude Code Everywhere](module-03-installing-everywhere.md) · [All modules](../README.md#the-course) · [Permissions, Memory, and Configuration →](module-05-permissions-memory-config.md)
