<!-- module: 5 | phase: 2 | format: deep-dive | last_verified: 2026-09-15 -->
# Module 5 — Claude Code Setup, Configuration and Memory

## Chapter 1: Settings Files and CLAUDE.md

**Concept**

Claude Code reads its configuration from a stack of `settings.json` files, each scoped differently, and applies them in a fixed precedence order — from highest to lowest:

1. **Managed settings** — `managed-settings.json`, MDM, or the claude.ai console. Controlled by your organization, applies to every session on the machine, and individual users can't override it.
2. **Command line** — values passed via `claude --settings`, for that one invocation.
3. **Project local** — `.claude/settings.local.json`. Your personal overrides for one project; keep this out of version control.
4. **Shared project** — `.claude/settings.json`. Checked into the repo, shared with your whole team.
5. **User** — `~/.claude/settings.json`. Applies to every project you personally work on.

A key set at a higher level wins when the same key appears in more than one file. This is why a project's `.claude/settings.json` can force a stricter permission mode than your own `~/.claude/settings.json` wants, and why organization-managed settings always win over both.

Permissions are one of the most common things people configure here. `settings.json` supports `permissions.allow`, `permissions.deny`, and `permissions.ask` rules that pre-approve, block, or force a prompt for specific tools or commands (for example, allowing `Bash(npm test)` without a prompt, or denying writes to a sensitive directory outright). It also sets `permissions.defaultMode`, which controls which permission mode a session starts in — more on modes in the next chapter.

CLAUDE.md is a separate but related mechanism: it's not settings, it's instructions. Where `settings.json` enforces things at the client level regardless of what Claude decides, CLAUDE.md shapes Claude's behavior as context — it's not a hard enforcement layer. A project CLAUDE.md (`./CLAUDE.md` or `./.claude/CLAUDE.md`) is checked into source control and shared with your team; a `CLAUDE.local.md` sits alongside it for your own private, gitignored notes.

**Try it yourself**

1. In a test project, create `.claude/settings.json` and add a permission rule, for example an `ask` rule on a command you want to be careful with.
2. Run `/context` inside a Claude Code session in that project — check the list under **Memory files** and confirm the settings and CLAUDE.md files you expect actually loaded.
3. Add a conflicting value for the same key in `~/.claude/settings.json` and confirm (by reading the docs' precedence order, or by testing behavior) that the project-level file wins.
4. Run `/init` to have Claude Code generate a starting CLAUDE.md for the project automatically, based on what it finds in your codebase.

**Common pitfalls**

- Committing `.claude/settings.local.json` to git by accident — it's meant to be personal and machine-specific; add it to `.gitignore` immediately.
- Expecting a CLAUDE.md instruction to be enforced like a permission rule. If you need something to always happen regardless of what Claude decides (block a specific command outright, for instance), that belongs in `permissions.deny` in `settings.json`, not in prose in CLAUDE.md.
- Not checking `/context` after changing settings, then wondering why nothing changed — the change may not be in the file Claude Code actually loads for that session and directory.

**Key takeaways**

- Settings precedence, highest to lowest: managed settings, command line, project local, shared project, user. Higher always wins on a conflicting key.
- `settings.json` controls permissions, environment variables, and other enforced behavior; CLAUDE.md shapes behavior as context Claude tries to follow, not a hard rule.
- Use `/context` to verify which settings and memory files actually loaded into your current session — don't assume.

## Chapter 2: Claude Code Modes and the First Portfolio Build

**Concept**

Permission modes decide how much Claude does before it stops and asks you. Claude Code ships six modes:

| Mode | What runs without asking | Best for |
|---|---|---|
| `default` (labeled **Manual** in the CLI, VS Code, and desktop app) | Reads only | Reviewing every action yourself, sensitive work |
| `acceptEdits` | Reads, file edits, and common filesystem commands (`mkdir`, `touch`, `mv`, `cp`, etc.) | Iterating on code you're actively reviewing |
| `plan` | Reads, plus classifier-approved commands when auto mode is available | Exploring a codebase before changing anything |
| `auto` | Everything, with background safety checks from a separate classifier model | Long tasks, reducing prompt fatigue |
| `dontAsk` | Reads and pre-approved tools only; anything that would otherwise prompt is denied instead | Locked-down CI and scripts |
| `bypassPermissions` | Everything, no checks | Isolated containers and VMs only |

On Pro, Max, and Team plans, `auto` is the built-in starting mode for terminal and VS Code sessions — a classifier model reviews risky actions instead of you, blocking things like production deploys, force pushes, or sending sensitive data externally by default. On Enterprise plans and Console API keys, sessions start in Manual (`default`) instead.

Switch modes at any time in the CLI by pressing `Shift+Tab`, which cycles `default → acceptEdits → plan → default` (with `auto`, `bypassPermissions`, or `dontAsk` slotting in if enabled for your session). In VS Code, click the mode indicator at the bottom of the prompt box. Plan mode is particularly useful for a first build: Claude researches your codebase and proposes a plan without touching any files, and you approve, request changes, or reject before anything is written.

**Try it yourself: your first portfolio build**

Pick a small, real deliverable — a single-page portfolio site is a good fit for a first build.

1. Create a new empty project directory and start Claude Code inside it: `claude`.
2. Switch to plan mode with `Shift+Tab` (or start directly with `claude --permission-mode plan`).
3. Describe what you want: "Build a single-page portfolio site with a header, an about section, a projects section, and a contact form. Use plain HTML, CSS, and JavaScript, no framework."
4. Read the plan Claude proposes. If something's off, tell it what to change before approving — this is the point of plan mode.
5. Approve the plan (choose **Yes, and use auto mode** or **Yes, manually approve edits**, depending on how much oversight you want for this first run) and let Claude build the files.
6. Open the result in a browser, ask for a couple of follow-up tweaks in the same session, and watch how Claude re-reads its own files rather than regenerating everything from scratch.

**Common pitfalls**

- Starting a big, unfamiliar task in `auto` mode with no plan first — for a first build, plan mode gives you a checkpoint before any files exist, which is worth the extra step while you're still calibrating trust.
- Assuming `acceptEdits` mode also auto-approves arbitrary shell commands — it only covers file edits and a specific list of filesystem commands (`mkdir`, `touch`, `rm`, `rmdir`, `mv`, `cp`, `sed`). Anything else, like installing a package or running a build script, still prompts.
- Forgetting that `Shift+Tab` cycles differently depending on which optional modes are enabled for your session — if you don't see the mode you expect, check what's available rather than assuming the cycle is broken.

**Key takeaways**

- Six permission modes exist: `default` (Manual), `acceptEdits`, `plan`, `auto`, `dontAsk`, and `bypassPermissions` — each trades convenience for oversight differently.
- `Shift+Tab` switches modes in the CLI; the mode indicator does it in VS Code and the desktop app.
- Plan mode is the safest way to start an unfamiliar build: Claude proposes a plan, you approve it, and only then does it start editing.

## Chapter 3: Guiding the Project with CLAUDE.md and Memory

**Concept**

CLAUDE.md is how you give Claude persistent, project-specific instructions without repeating yourself every session. It's loaded fresh at the start of every conversation, alongside a second, complementary mechanism called auto memory.

The difference between the two:

- **CLAUDE.md** — you write it. Coding standards, build commands, architectural decisions, naming conventions, "always do X" rules. Lives at `./CLAUDE.md` or `./.claude/CLAUDE.md` for the team, `~/.claude/CLAUDE.md` for your own personal preferences across every project, and `CLAUDE.local.md` for personal, gitignored, project-specific notes.
- **Auto memory** — Claude writes it, automatically, based on your corrections and things it learns during a session that it can't derive from the code itself (your working preferences, project context, where to find things outside the repo). It's stored per-project under `~/.claude/projects/<project>/memory/`, indexed by a `MEMORY.md` file, and it's on by default.

Both load into every session, but CLAUDE.md is deliberately concise — the docs recommend keeping it under 200 lines, because longer files consume more context and reduce how reliably Claude follows them. If instructions only matter for part of the codebase, scope them with a path-specific rule in `.claude/rules/` instead of bloating the main file. Run `/init` to have Claude generate a starting CLAUDE.md by analyzing your codebase, and `/memory` at any time to browse, edit, or toggle auto memory.

Writing a good CLAUDE.md is a specificity exercise: "Use 2-space indentation" works better than "format code properly," and "Run `npm test` before committing" works better than "test your changes." Vague instructions get inconsistently followed; concrete, verifiable ones don't.

**Try it yourself**

1. In your portfolio project from Chapter 2, run `/init` and read the CLAUDE.md it generates. Check whether it captured your actual build command and folder layout correctly.
2. Add one concrete rule of your own — for example, a note about which CSS approach you want used for any new page — and start a new session to confirm it gets picked up.
3. Ask Claude to do something slightly wrong on purpose, correct it in chat, and then run `/memory` afterward to see whether it saved that correction as an auto memory entry.
4. Deliberately write two conflicting instructions in different CLAUDE.md files (one in `~/.claude/CLAUDE.md`, one in the project's `./CLAUDE.md`) and observe which one Claude tends to follow — this is a good way to internalize why keeping instructions consistent matters.

**Common pitfalls**

- Letting CLAUDE.md grow into a multi-hundred-line document that tries to cover every edge case — past a certain size it stops being read reliably, and the fix is path-scoped rules or trimming, not a bigger file.
- Treating CLAUDE.md as an enforcement mechanism ("this will always block Claude from doing X") — it's context, not a hard rule. Anything that must be enforced regardless of Claude's judgment belongs in a `permissions.deny` rule instead.
- Forgetting that auto memory is machine-local — it won't follow you to a different laptop, and it isn't shared with teammates the way a committed CLAUDE.md is.

**Key takeaways**

- CLAUDE.md is instructions you write; auto memory is notes Claude writes about your corrections and preferences — both load at the start of every session.
- Keep CLAUDE.md under roughly 200 lines, specific and verifiable; use `.claude/rules/` for instructions that should only load for particular files.
- `/init` generates a starting CLAUDE.md from your codebase; `/memory` lets you inspect, edit, or disable auto memory at any time.

<!-- nav -->
[← Module 4: Getting Started with Claude Code](module-04-getting-started.md) | [Module 6: Built-in Tools and Working with Codebases →](module-06-built-in-tools-codebases.md)
