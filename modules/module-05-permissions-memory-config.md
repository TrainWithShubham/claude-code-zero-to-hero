<!-- module: 5 | phase: 2 | format: deep-dive | last_verified: 2026-08-12 -->

# Module 5 — Permissions, Memory, and Configuration

This is the module that decides how much you have to babysit Claude Code. Get it right and long sessions run themselves. Get it wrong and you're either clicking "approve" a hundred times or handing over more access than you meant to.

One idea runs through the whole module, and it's worth holding from the start:

> **Settings are enforced by the client regardless of what Claude decides. CLAUDE.md and memory shape Claude's behavior but are not enforcement.**

If something *must* happen or *must not* happen, it belongs in settings or a hook — not in a markdown file asking nicely.

---

## Chapter 1 — Choosing a permission mode

**What you'll learn:** the six modes, what each one lets through, and how to pick.

### The concept

When Claude wants to edit a file, run a command, or make a network request, it pauses and asks. Permission modes control how often that pause happens.

| Mode | Runs without asking | Best for |
|---|---|---|
| `default` | **Reads only** | Getting started, sensitive work |
| `acceptEdits` | Reads, file edits, and common filesystem commands (`mkdir`, `touch`, `mv`, `cp`) | Iterating on code you're reviewing |
| `plan` | Reads, plus classifier-approved commands when auto mode is available | Exploring a codebase before changing it |
| `auto` | Everything, with background safety checks | Long tasks, reducing prompt fatigue |
| `dontAsk` | Only pre-approved tools | Locked-down CI and scripts |
| `bypassPermissions` | Everything | Isolated containers and VMs only |

Two things people get wrong:

- **`default` is shown as "Manual mode"** in the interface. It doesn't ask about everything — reads run freely. Only writes and commands prompt.
- **`plan` is not purely read-only.** With auto mode available, plan mode also runs classifier-approved commands.

### Protected-path writes

Separate from the table above, writes to protected paths follow their own rules:

| Mode | Protected-path writes |
|---|---|
| `default`, `acceptEdits` | Prompted |
| `plan` | Prompted; routed to the classifier when auto mode is available |
| `auto` | Routed to the classifier |
| `dontAsk` | **Denied** |
| `bypassPermissions` | Allowed |

### Switching modes

`Shift+Tab` cycles `default` → `acceptEdits` → `plan`, with `auto` and `bypassPermissions` joining conditionally. Or start in a mode directly:

```bash
claude --permission-mode plan
```

The status bar always shows where you are:

| Badge | Mode |
|---|---|
| `⏸ manual mode on` (gray) | `default` |
| `⏵⏵ accept edits on` | `acceptEdits` |
| `⏸ plan mode on` | `plan` |
| `⏵⏵ auto mode on` | `auto` |
| `⏵⏵ don't ask on` | `dontAsk` |
| `⏵⏵ bypass permissions on` | `bypassPermissions` |

### Try it yourself

Press `Shift+Tab` a few times and watch the badge change. Then ask Claude to make a small edit in each of the first three modes and notice exactly where it stops to ask you.

### Common pitfalls

- **Reaching for `bypassPermissions` because the prompts are annoying.** That's what `auto` is for. Bypass is for isolated containers and VMs, not your laptop.
- **Assuming `plan` mode can't touch anything.** It can run classifier-approved commands when auto mode is available.

### Key takeaways

- Six modes, from reads-only to everything
- `default` is labeled Manual, and reads never prompt in any mode
- Protected-path writes follow separate rules — `dontAsk` denies them outright

---

## Chapter 2 — Auto mode deep dive

**What you'll learn:** what the classifier actually does, and what changes on August 14, 2026.

### The concept

Auto mode replaces permission prompts with background safety checks. A **separate classifier model** reviews commands before they run and blocks what looks risky: scope escalation, unknown infrastructure, and actions driven by hostile content. Routine reads and edits in your working directory run immediately.

It respects boundaries you state in conversation. Tell it "don't push to main" and it won't, even in auto mode.

### The default is changing

> Starting **August 14, 2026**, auto mode becomes the default permission mode for new sessions on **Pro, Max, and Team** plans.

Three conditions worth knowing:

- **A default you set yourself stays in place**, unless you accept the one-time switch prompt.
- **A default your organization manages is unchanged.**
- Admins can remove auto mode entirely with `disableAutoMode: "disable"` in managed settings. That strips `auto` from the `Shift+Tab` cycle and rejects `--permission-mode auto` at startup.

### On third-party providers

On Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry, and signed-in gateway sessions, auto mode **appears in the `Shift+Tab` cycle** by default — but sessions still **start** in your configured `defaultMode`, which is Manual unless you changed it. Appearing in the cycle is not the same as being the starting mode.

Only Claude Sonnet 5, Opus 4.7 or later, and Fable 5 are supported for auto mode on those providers.

### Try it yourself

```bash
claude --permission-mode auto -p "fix all lint errors"
```

In non-interactive `-p` runs, auto mode **aborts** if the classifier repeatedly blocks actions — there's no user to fall back to.

### Common pitfalls

- **Expecting auto mode to be on by default on Bedrock or Vertex.** It's in the cycle, not the starting mode.
- **Assuming auto mode means unsupervised.** The classifier blocks risky actions; it doesn't verify that the work is correct. That's still your test suite's job.

### Key takeaways

- A separate classifier vets commands, so routine work runs uninterrupted
- Default for new Pro/Max/Team sessions from August 14, 2026 — with carve-outs for self-set and org-managed defaults
- Third-party providers: in the cycle, but not the starting mode

---

## Chapter 3 — CLAUDE.md explained

**What you'll learn:** where CLAUDE.md files live, the order they load in, and how to keep them working.

### The load order

From broadest scope to most specific. Files are **concatenated, not overridden** — so a project instruction lands in context *after* a user instruction:

| Scope | Location |
|---|---|
| **Managed policy** | macOS `/Library/Application Support/ClaudeCode/CLAUDE.md` · Linux/WSL `/etc/claude-code/CLAUDE.md` · Windows `C:\Program Files\ClaudeCode\CLAUDE.md` |
| **User** | `~/.claude/CLAUDE.md` |
| **Project** | `./CLAUDE.md` or `./.claude/CLAUDE.md` |
| **Local** | `./CLAUDE.local.md` — add to `.gitignore` |

Claude walks up the directory tree from your working directory, loading every `CLAUDE.md` and `CLAUDE.local.md` it finds, ordered from the filesystem root down. Files in **subdirectories** aren't loaded at launch — they load on demand when Claude reads a file in that directory.

Managed policy CLAUDE.md **cannot be excluded** by individual settings.

### Writing one that works

- **Target under 200 lines.** Files load in full regardless of length, but longer files reduce adherence.
- **Be specific enough to verify.** "Use 2-space indentation" beats "format code properly."
- **Watch for contradictions.** If two files disagree, Claude may pick one arbitrarily.
- Run `/init` to generate a starting file from your codebase, then refine.

### Imports

```text
See @README for project overview and @package.json for npm commands.

# Additional Instructions
- git workflow @docs/git-instructions.md
```

- Relative paths resolve against the file containing the import, not your working directory. Max depth is four hops.
- **Imports don't save context** — imported files expand and load at launch.
- To mention a path without importing it, wrap it in backticks: `` `@README` ``.
- An import resolving **outside your working directory** triggers a one-time approval dialog. That's protection against files someone else commits to a shared project. Imports in `~/.claude/` load without the dialog.

### `.claude/rules/` — the answer to "my CLAUDE.md is too big"

```text
your-project/
├── .claude/
│   ├── CLAUDE.md
│   └── rules/
│       ├── code-style.md
│       ├── testing.md
│       └── security.md
```

Rules can be **scoped to file paths**, so they only enter context when relevant:

```markdown
---
paths:
  - "src/api/**/*.ts"
---
# API Development Rules
- All API endpoints must include input validation
- Use the standard error response format
```

- Rules **without** a `paths` field load at launch, at the same priority as `.claude/CLAUDE.md`.
- Rules **with** `paths` trigger when Claude reads a matching file.
- User-level rules in `~/.claude/rules/` load before project rules, so project rules win.
- Symlinks work, so one shared rule set can serve many repos.

In a monorepo, `claudeMdExcludes` skips other teams' CLAUDE.md files by glob.

### Two details that save confusion

**HTML comments are stripped** before CLAUDE.md is injected into context, so `<!-- maintainer notes -->` costs no tokens. Comments inside code blocks are preserved.

**`AGENTS.md` is not read.** Claude Code reads `CLAUDE.md`. If your repo already uses `AGENTS.md` for other agents, import it:

```markdown
@AGENTS.md

## Claude Code
Use plan mode for changes under `src/billing/`.
```

### Try it yourself

```
/init
```

Then run `/context` and check the **Memory files** list to confirm what actually loaded. Delete half the generated file using the test: *would removing this cause Claude to make mistakes?*

### Common pitfalls

- **Assuming user settings beat project settings.** Project loads after user, so project wins.
- **Splitting a huge CLAUDE.md into imports to save context.** It doesn't — imports load at launch. Use path-scoped rules instead.
- **Expecting CLAUDE.md to enforce anything.** It's context. Use a hook for guarantees.

### Key takeaways

- Load order: managed policy → user → project → local, concatenated
- Under 200 lines, specific, non-contradictory
- Path-scoped rules in `.claude/rules/` are how you scale instructions without bloating context

---

## Chapter 4 — Auto memory

**What you'll learn:** what Claude writes about your project on its own, where it goes, and how to audit it.

### The concept

Auto memory is on by default. Claude writes its own notes as it works — build commands, debugging insights, architecture notes, style preferences — and reads them back in later sessions. You write CLAUDE.md; Claude writes memory.

Each project gets a directory:

```text
~/.claude/projects/<project>/memory/
├── MEMORY.md          # concise index, loaded every session
├── debugging.md       # detailed notes
├── api-conventions.md
└── ...
```

The `<project>` path derives from the **git repository**, so every worktree and subdirectory of the same repo shares one memory directory.

### What actually loads

**The first 200 lines of `MEMORY.md`, or the first 25KB — whichever comes first.** Anything past that is dropped at load time.

Topic files like `debugging.md` are **not** loaded at startup. Claude reads them on demand with its normal file tools.

If `MEMORY.md` grows past a limit, the write still succeeds, but Claude Code returns an error telling Claude to rewrite the index — because everything past the limit silently disappears on the next load. YAML frontmatter and HTML comments are stripped before measuring, so they don't count.

**Auto memory is machine-local.** It is not shared across machines or cloud environments.

### Controlling it

- `/memory` — browse and edit every memory file, and toggle auto memory
- `autoMemoryEnabled: false` in settings — per project or per user
- `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1` — environment variable
- `autoMemoryDirectory` — store memory somewhere else

### Try it yourself

Work through a real task, then:

```
/memory
```

Open the auto memory folder and read what Claude saved. Everything is plain markdown you can edit or delete. Watch for **"Saved 2 memories"** or **"Recalled 2 memories"** in the interface during a session — that's this system working.

### Common pitfalls

- **Assuming memory syncs across machines.** It doesn't.
- **Letting `MEMORY.md` grow past 200 lines.** Content past the limit is dropped silently on load.
- **Confusing memory with CLAUDE.md.** You write CLAUDE.md deliberately; Claude writes memory opportunistically. Both are context, neither is enforcement.

### Key takeaways

- On by default, per git repository, machine-local
- `MEMORY.md` loads up to 200 lines / 25KB; topic files load on demand
- Audit with `/memory` — it's all plain markdown

---

## Chapter 5 — The `.claude` directory and settings.json

**What you'll learn:** where configuration lives and which layer wins.

### The concept

`.claude/` (project) and `~/.claude/` (user) hold CLAUDE.md, settings.json, hooks, skills, commands, subagents, rules, and output styles.

### Settings precedence, highest to lowest

1. **Managed** — server-managed settings, macOS managed preferences, Windows policy registry, or `managed-settings.json`. **Cannot be overridden**, with narrow exceptions.
2. **Command line arguments** — temporary session overrides
3. **Local** — `.claude/settings.local.json`, gitignored
4. **Project** — `.claude/settings.json`, checked into git
5. **User** — `~/.claude/settings.json`

Note that **managed settings outrank CLI flags**, and that **local outranks project** — both are easy to get backwards.

**Permission rules are the exception: they merge across scopes rather than override.** Security-sensitive settings also honor restrictive values from scopes that otherwise couldn't override.

### Settings vs CLAUDE.md

| Concern | Configure in |
|---|---|
| Block specific tools, commands, or paths | Managed settings — `permissions.deny` |
| Enforce sandbox isolation | Managed settings — `sandbox.enabled` |
| Environment variables and provider routing | Managed settings — `env` |
| Auth method and organization lock | Managed settings — `forceLoginMethod`, `forceLoginOrgUUID` |
| Code style and quality guidelines | CLAUDE.md |
| Data handling and compliance reminders | CLAUDE.md |
| Behavioral instructions | CLAUDE.md |

Settings are enforced by the client no matter what Claude decides. CLAUDE.md shapes behavior but is not a hard enforcement layer. When you catch yourself writing "NEVER do X" in CLAUDE.md, that's the signal it belongs in settings or a hook.

### Try it yourself

Open `.claude/settings.json` in your project and add a permission rule. Then check `/context` and `/doctor` to confirm it took effect.

### Common pitfalls

- **Putting a hard rule in CLAUDE.md and expecting enforcement.** It's advisory.
- **Assuming CLI flags win.** Managed settings outrank them.

### Key takeaways

- Precedence: managed → CLI args → local → project → user
- Permission rules merge across scopes instead of overriding
- Enforcement lives in settings and hooks; guidance lives in CLAUDE.md

---

## Chapter 6 — Debugging your config

**What you'll learn:** the four commands that answer "why isn't this working?"

### The toolkit

| Command | Answers |
|---|---|
| `/context` | What actually loaded — including a **Memory files** list |
| `/doctor` (alias `/checkup`) | Full setup checkup; can propose fixes |
| `/memory` | Every memory file, and the auto memory toggle |
| `/hooks` | What's registered |
| `/mcp` | What's connected |

`/context` is almost always the right first stop. If a CLAUDE.md file isn't in the **Memory files** list, Claude can't see it — no amount of rewording will help.

For deeper debugging, the **`InstructionsLoaded` hook** logs exactly which instruction files loaded, when, and why. That's the tool for diagnosing path-scoped rules that aren't triggering.

`/doctor` also proposes trims for a checked-in CLAUDE.md: it cuts content Claude can derive from the codebase (directory layouts, dependency lists) and keeps pitfalls, rationale, and conventions that differ from tool defaults.

### The compaction gotcha

This explains a confusing behavior worth knowing before it bites you:

- **Project-root CLAUDE.md survives `/compact`** — Claude re-reads it from disk and re-injects it.
- **Nested CLAUDE.md files and path-scoped rules do not.** They reload only the next time Claude reads a matching file.

So if an instruction seems to vanish after compaction, it was either given only in conversation, lives in a nested CLAUDE.md that hasn't reloaded, or is a path-scoped rule that hasn't matched a file since.

### Try it yourself

```
/context
```

Find the **Memory files** section and match it against what you expected to load. Then:

```
/doctor
```

Read what it flags. On most real projects it finds something.

### Common pitfalls

- **Rewording CLAUDE.md when the file never loaded.** Check `/context` first.
- **Blaming compaction for a conversation-only instruction.** If you said it in chat and want it to persist, write it down.

### Key takeaways

- `/context` first, always
- `/doctor` for a full checkup, `/memory` `/hooks` `/mcp` for specific layers
- Project-root CLAUDE.md survives compaction; nested files and path-scoped rules reload lazily

---
<!-- nav -->

[← The Agentic Loop and Built-In Tools](module-04-agentic-loop-tools.md) · [All modules](../README.md#the-course) · [Planning and Executing a Real Project →](module-06-planning-real-projects.md)
