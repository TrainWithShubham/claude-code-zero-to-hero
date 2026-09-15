<!-- module: 8 | phase: 2 | format: deep-dive | last_verified: 2026-09-15 -->
# Module 8 — Executing and Building the Project

Module 7 ended with an approved plan for the portfolio project. This module is where that plan turns into working code. The shift here is from careful scoping to careful execution — trusting Claude Code to make more changes per turn while still keeping track of what it's doing and why.

## Chapter 1: Starting the Build and Tracking Progress

### Concept

Plan mode is deliberately restrictive — it won't touch a file. Once you approve a plan and exit plan mode, you choose how much oversight to keep for the build itself. Manual mode (the CLI's default review-everything mode) asks before every file edit and every command, which is safe but slow for a build with dozens of small edits. `acceptEdits` mode auto-approves file edits and common filesystem commands like `mkdir`, `mv`, and `touch`, while still asking before anything riskier — a good middle ground once you trust the plan. `auto` mode goes further, letting a background classifier review actions instead of you prompting for each one, which suits long, mostly-mechanical builds where you don't want to sit at the keyboard approving every step. You switch between these with `Shift+Tab` during a session, or start a session in a specific mode with `--permission-mode`.

For a project with multiple features — About section, projects grid, contact form — a single long build benefits from a visible task list. Ask Claude Code to track its progress in a plain markdown file (a `TASKS.md` or similar) as it works, checking items off as it completes them. This isn't a Claude Code feature with a specific flag; it's a working pattern — you're asking the agent to externalize its own progress into a file you can both read, so a build that spans many turns or gets interrupted doesn't lose its place. Treat that file as a checkpoint: if a session ends or you come back the next day, reading it tells you and Claude Code exactly where the build stood.

### Try it yourself

With your approved plan from Module 7, exit plan mode and switch to `acceptEdits` mode with `Shift+Tab`. Ask Claude Code to create a `TASKS.md` breaking the plan into a checklist — one item per feature or setup step — and to update it as each item is finished. Let it build the project skeleton and the About section, then stop and read `TASKS.md` to confirm it reflects what actually happened, not just what was planned.

### Common pitfalls

- **Jumping straight to `auto` mode on a project you haven't reviewed edits from before.** Auto mode's classifier catches a lot, but it's not a substitute for you having seen how this particular agent session behaves on this particular codebase. Start a new project in `acceptEdits` mode first so you're watching the pattern of changes before handing off more control.
- **Letting the task file drift from reality.** A task list only helps if it's updated honestly — if Claude Code marks something done that only partially works, the file becomes misleading instead of useful. Spot-check a few "done" items against the actual code.
- **Forgetting which mode you're in.** The status bar shows the active mode, but it's easy to keep typing without checking it. Running a destructive command while assuming you're still in Manual mode is a common surprise — glance at the indicator before a build session that touches a lot of files.

### Key takeaways

- Manual mode reviews everything, `acceptEdits` auto-approves file edits and common filesystem commands, and `auto` mode hands routine review to a background classifier — choose based on how much you trust the plan and how much attention you want to give the build.
- A markdown task file is a working pattern, not a built-in feature: ask Claude Code to maintain one so a multi-turn build has a visible, resumable checkpoint.
- Check the task file against the actual code occasionally — a stale or overly generous checklist defeats the point of tracking progress.

## Chapter 2: Running, Testing and Debugging the Portfolio

### Concept

Once the portfolio has enough built to run, the loop changes shape: build a piece, run it, see what's actually happening, fix what's wrong. Claude Code can run your dev server or build command directly through its Bash tool, so you don't have to switch to a separate terminal to check progress — ask it to start the server and report back what loads, or to run a build and paste back any errors.

Testing a project like this doesn't require heavy infrastructure to be worth doing. Ask Claude Code to write a handful of focused tests for the pieces most likely to break silently — the projects grid reading from your JSON file, the contact form's validation — and to run them as part of finishing each task in `TASKS.md`. When a test fails, resist the urge to just ask "fix it." Ask Claude Code to explain what the failure means first: what the test expected, what actually happened, and why. That explanation is what turns a fixed bug into something you understand, rather than a diff you accepted without reading.

Debugging with an agent works best when you give it the actual error, not a paraphrase of it. Paste the real stack trace or console output rather than describing the symptom from memory — Claude Code can search your codebase for the failing line far faster than it can guess at a vague description.

### Try it yourself

With the projects grid and contact form built, ask Claude Code to run the project locally and confirm both features work. Then deliberately introduce a bug — rename a field in your JSON data file without updating the code that reads it — and ask Claude Code to run the project again, read the resulting error, and explain what broke before fixing it. Confirm its explanation matches your own read of the error before letting it apply the fix.

### Common pitfalls

- **Asking for a fix before asking for an explanation.** Skipping straight to "fix it" gets you working code and no understanding of why it broke — which matters when a similar bug shows up somewhere else in the project later.
- **Describing an error instead of pasting it.** "It's not loading the projects" gives Claude Code far less to work with than the actual console error. Always paste the real output.
- **Writing tests only after everything is built.** Testing a feature right after building it, while the code is fresh in context, catches problems while they're cheap to fix — waiting until the whole portfolio is done makes failures harder to trace back to a cause.

### Key takeaways

- Claude Code can run your dev server, build commands, and tests directly — use that instead of switching to a separate terminal to check progress.
- Ask for an explanation of a failure before asking for a fix, and always give Claude Code the real error output rather than a description of the symptom.
- Test each feature close to when it's built, not all at once at the end.

<!-- nav -->
[← Module 7: Planning a Complete Project With Claude Code](module-07-planning-a-project.md) | [Module 9: Slash Commands, Skills and Sub-Agents →](module-09-slash-commands-skills-subagents.md)
