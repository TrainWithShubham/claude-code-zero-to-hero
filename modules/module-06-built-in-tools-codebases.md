<!-- module: 6 | phase: 2 | format: deep-dive | last_verified: 2026-09-15 -->
# Module 6 — Built-in Tools and Working with Codebases

## Chapter 1: Built-in Tools, Bash and File Operations

**Concept**

Everything Claude Code does to your project happens through a fixed set of built-in tools. Understanding what each one is for makes it much easier to predict what Claude will do next, and to write prompts that steer it toward the right one. The core tools you'll see used constantly on any coding task:

- **Read** — reads the contents of a file. Claude uses this before editing anything, and often before answering a question about your code, so it's working from the actual current content rather than a guess.
- **Write** — creates a new file or completely overwrites an existing one. Used for new files; risky for existing ones, since it replaces the whole thing.
- **Edit** — makes a targeted change to a specific file using exact string replacement, rather than rewriting the whole file. This is the tool Claude reaches for most often when fixing a bug or adjusting existing code, because it changes only what needs to change.
- **Bash** — executes shell commands: running tests, installing dependencies, checking git status, starting a dev server. This is also how Claude does things that don't have a dedicated tool.
- **Grep** — searches for a pattern across file contents, the equivalent of you running `grep -r` across the codebase.
- **Glob** — finds files by name pattern, like `**/*.test.ts`, without looking inside them.

Grep and Glob matter because Claude doesn't have your whole codebase loaded into its context by default — it explores the same way you would, by searching for the right file and the right lines before reading and editing them. This is also why giving Claude specific file paths or function names in your prompt speeds things up: it skips a search step it would otherwise have to run itself.

Which of these tools Claude can use without asking you first depends on your permission mode (Module 5, Chapter 2). In Manual mode, Read/Grep/Glob run freely because they're read-only, but Write, Edit, and most Bash commands prompt for approval. In `acceptEdits` mode, Write and Edit run without prompting, plus a specific list of filesystem Bash commands (`mkdir`, `touch`, `mv`, `cp`, `sed`, and a few others) — everything else in Bash still asks.

There's also a shortcut for when you want to run a shell command yourself rather than asking Claude to run it: prefix your input with `!` (for example, `!git status`) and Claude Code executes it directly, with no model call and no interpretation in between. The output still gets added to the conversation, so Claude can see and act on it, but you didn't spend a turn asking for it. It's the fastest way to check something — test output, a git diff, a running process — without pulling Claude's reasoning into a step that doesn't need it.

**Try it yourself**

1. In a real (not empty) project, ask Claude: "find every place `fetchUser` is called." Watch which tool it reaches for — this should be Grep, not a manual read of every file.
2. Ask Claude to "list all the test files in this project" and see it use Glob rather than Read.
3. Ask Claude to make a one-line fix to an existing file (e.g., renaming a variable) and check the diff it shows you — this should go through Edit, changing only the relevant lines, not Write rewriting the whole file.
4. Ask Claude to run your project's test suite. In Manual mode, notice the permission prompt before the Bash command runs; approve it and read the output Claude reports back.

**Common pitfalls**

- Asking Claude to "rewrite this file" when you actually want a small change — that phrasing nudges it toward Write instead of Edit, which is more likely to lose formatting or unrelated code you wanted kept.
- Being surprised that Claude "can't see" a file it hasn't read yet. Claude Code doesn't pre-load your whole repository into context; if you reference a file by name, that speeds up the search, but Claude will still explicitly Read it before acting on it.
- Approving a broad Bash permission (like an unscoped wildcard) just to stop the prompts, then losing track of what Claude has actually run. Narrow, specific allow rules (Module 5, Chapter 1) give you the same reduction in prompts with far less risk.

**Key takeaways**

- The core file and search tools are Read, Write, Edit, Bash, Grep, and Glob — each with a distinct job, and Claude picks between them based on what your prompt asks for.
- Edit is for targeted changes to existing files; Write is for new files or full overwrites. Prefer prompts that point toward Edit when you only want a small change.
- Grep and Glob are how Claude finds things in a codebase it hasn't fully read — give it specific paths or names in your prompt to skip that search step.

## Chapter 2: Reviewing Code and Fixing Bugs with Claude Code

**Concept**

Reviewing and debugging is one of the most practical everyday uses of Claude Code, and it follows a natural extension of the same tool loop: search to find the relevant code, read it, reason about it, and — if you approve — edit it.

For a **code review**, describe the scope you want covered rather than just saying "review my code." "Review my changes and suggest improvements" is a fine starting prompt, but you'll get sharper results narrowing it: "review the changes on this branch for correctness bugs, not style" or "check the new API handlers for missing input validation." Claude will use Grep and Glob to locate the relevant files, Read them, and report findings — often without touching anything, since a review doesn't require an edit.

For **bug fixing**, the loop looks like this:

1. Describe the bug in concrete terms: "there's a bug where users can submit empty forms — fix it," not just "fix the bug."
2. Claude locates the relevant code (Grep/Glob), reads the surrounding context (Read) so it understands why the bug happens, not just where.
3. It proposes and makes a fix (Edit).
4. If your project has tests, it runs them (Bash) to check the fix didn't break anything else, and to see whether an existing test already covers the behavior.

The quality of the fix depends heavily on how much context you give up front. "Fix the bug" forces Claude to guess which bug you mean and how to reproduce it. A description that includes the symptom, when it happens, and ideally how to trigger it gets you a targeted fix on the first try far more often.

Claude Code also makes ordinary git operations conversational, which matters for review workflows: "what files have I changed?", "commit my changes with a descriptive message," and "show me the last 5 commits" all work as plain prompts, so you can move from finding a bug to fixing it to committing the fix without switching out of the conversation.

**Try it yourself**

1. Pick an existing project with at least a few files (your portfolio build from Module 5 works, or any small repo you have locally). Ask Claude: "review this codebase and point out anything that looks like a bug or an obvious risk."
2. Take one finding from that review and ask Claude to fix it, giving it the specific symptom rather than just pointing at the finding by number.
3. After the fix, ask "what files have I changed?" and then "commit my changes with a descriptive message" — watch how Claude drafts the commit message from the actual diff rather than a generic placeholder.
4. Deliberately introduce a small bug yourself (an off-by-one, a wrong comparison operator) and ask Claude to find and fix it without telling it where the bug is — this is a good way to gauge how well it searches versus how well it reasons once it finds the right code.

**Common pitfalls**

- Asking for a review with no scope ("review my code") on a large project — Claude will either cover too much shallowly or ask you to narrow it. Scope the review yourself: a branch, a directory, or a specific concern (security, performance, correctness).
- Accepting a proposed fix without reading the diff, especially in `acceptEdits` or `auto` mode. A fast fix is not the same as a correct one — treat Claude's fix the way you'd treat a colleague's pull request, not as ground truth.
- Not mentioning that tests exist. If your project has a test suite, say so and ask Claude to run it after a fix — otherwise it may not know one exists or where the command to run it lives, especially if that's not documented anywhere in CLAUDE.md yet.

**Key takeaways**

- A good review or bug-fix prompt states the scope and the symptom concretely — "fix the bug" invites guessing, a described symptom invites a targeted search and fix.
- Claude's fix loop is search, read, edit, then verify with tests if they exist — the same built-in tools from Chapter 1, chained together.
- Git operations work as plain-language prompts inside the same session, so review, fix, and commit can happen without leaving the conversation — but always read the diff before trusting the result.

<!-- nav -->
[← Module 5: Claude Code Setup, Configuration and Memory](module-05-setup-configuration-memory.md) | [Module 7: Planning a Complete Project With Claude Code →](module-07-planning-a-project.md)
