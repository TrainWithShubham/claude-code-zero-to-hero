<!-- module: 2 | phase: 1 | last_verified: 2026-08-12 -->

# Module 2 — Prompt Engineering for Agentic Work

## The constraint everything else follows from

> **Claude's context window fills up fast, and performance degrades as it fills.**

The context window holds your entire conversation — every message, every file Claude reads, every command's output. A single debugging session can burn tens of thousands of tokens. As it fills, Claude starts forgetting earlier instructions and making more mistakes.

So prompt engineering here isn't about magic words. It's about managing a budget. Nearly every technique below is an answer to one question: how do I get the right information in, and keep the wrong information out?

---

## 1. Anatomy of a good prompt for an agent

State the outcome and the constraints, and let the loop work out the "how." But be precise — **agent prompts usually need more detail than chatbot prompts, not less.** Automatic context-gathering means Claude *can* find things; it doesn't tell Claude which things matter or what "done" looks like.

Four moves that make a prompt work:

**Scope the task** — name the file, the scenario, the preference.
- ❌ `add tests for foo.py`
- ✅ `write a test for foo.py covering the edge case where the user is logged out. avoid mocks.`

**Point to sources** — tell Claude where the answer lives.
- ❌ `why does ExecutionFactory have such a weird api?`
- ✅ `look through ExecutionFactory's git history and summarize how its api came to be`

**Reference existing patterns** — the highest-leverage move on a real codebase.
- ❌ `add a calendar widget`
- ✅ `look at how existing widgets are implemented on the home page. HotDogWidget.php is a good example. follow that pattern, and build from scratch without libraries other than the ones already used.`

**Describe the symptom, not the fix.**
- ❌ `fix the login bug`
- ✅ `users report login fails after session timeout. check the auth flow in src/auth/, especially token refresh. write a failing test that reproduces it, then fix it.`

### Always give it a way to verify

This matters more than any other single thing in your prompt:

> Claude stops when the work looks done. Without a check it can run, "looks done" is the only signal available — and you become the verification loop.

The check is anything returning a signal Claude can read: a test suite, a build exit code, a linter, a script that diffs output against a fixture, a browser screenshot compared to a design. In practice, adding `run the tests after implementing` to the end of a prompt is often worth more than three paragraphs of specification.

Ask for evidence rather than assertion — the test output, the command and what it returned. Reading evidence is faster than re-running the check yourself.

### Getting context in

- **`@file`** references — Claude reads the file before responding
- **Paste or drag images** directly into the prompt
- **Give URLs** for docs and API references; allowlist frequent domains with `/permissions`
- **Pipe data in** — `cat error.log | claude`

### Let Claude interview you

For anything larger than a single change, this beats writing a spec yourself:

```
I want to build [brief description]. Interview me in detail using the AskUserQuestion tool.
Ask about technical implementation, UI/UX, edge cases, concerns, and tradeoffs.
Don't ask obvious questions, dig into the hard parts I might not have considered.
Keep interviewing until we've covered everything, then write a complete spec to SPEC.md.
```

Then start a **fresh session** to execute the spec — clean context, written plan. Good specs name the files and interfaces involved, state what's out of scope, and end with an end-to-end verification step.

---

## 2. Where instructions actually live

| Mechanism | How it works | Use it when |
|---|---|---|
| **System prompt** | Claude Code's built-in instructions | Not yours to edit directly |
| **Output styles** | *Modifies* the system prompt | You want a different role, tone, or format **every turn** |
| **CLAUDE.md** | Adds a **user message after** the system prompt | Claude should always know your project conventions |
| **`--append-system-prompt`** | Appends to the system prompt | A one-off addition for a single invocation |
| **Skills** | Loads task-specific instructions on demand | You have a reusable workflow |
| **Subagents** | Its own system prompt, model, and tools | You want a separately scoped helper |

**CLAUDE.md is not part of the system prompt.** It arrives as a user message after it. That's exactly why it's context rather than enforcement — and why a hook is the right tool when something must actually be blocked (Module 7).

### Writing a CLAUDE.md that works

Start with `/init` to generate one from your project structure, then prune. For every line, ask:

> *"Would removing this cause Claude to make mistakes?"* If not, cut it.

| ✅ Include | ❌ Exclude |
|---|---|
| Bash commands Claude can't guess | Anything Claude can figure out by reading code |
| Style rules that differ from defaults | Standard conventions Claude already knows |
| Testing instructions and preferred test runners | Detailed API docs — link to them instead |
| Repo etiquette (branch naming, PR conventions) | Information that changes frequently |
| Architectural decisions specific to your project | File-by-file descriptions of the codebase |
| Environment quirks (required env vars) | Self-evident advice like "write clean code" |

**A bloated CLAUDE.md makes Claude ignore your actual instructions.** If Claude keeps doing something you have a rule against, the file is probably too long and the rule is getting lost. If Claude asks a question the file already answers, the phrasing is ambiguous. Treat it like code: review it when things go wrong, prune it regularly, and confirm it loaded with `/context`.

It can live at `~/.claude/CLAUDE.md` (all sessions), `./CLAUDE.md` (checked into git, shared with the team), `./CLAUDE.local.md` (personal, gitignored), or in parent and child directories for monorepos. Full detail in Module 5.

### Output styles

Output styles change *how* Claude responds, not what it knows. Built-ins:

- **Default** — the standard software-engineering system prompt
- **Proactive** — executes immediately, makes reasonable assumptions, prefers action over planning
- **Explanatory** — adds educational "Insights" while working
- **Learning** — leaves `TODO(human)` markers in your code for you to implement

Select one with `/config` → **Output style**, or set `"outputStyle": "Explanatory"` in a settings file.

Custom styles are markdown files in `~/.claude/output-styles/` or `.claude/output-styles/`:

```markdown
---
name: Diagrams first
description: Lead every explanation with a diagram
keep-coding-instructions: true
---

When explaining code, architecture, or data flow, start with a Mermaid diagram
showing the structure, then explain in prose.
```

**Two things that catch people out:**

- Without `keep-coding-instructions: true`, a custom style **strips Claude Code's built-in software-engineering instructions** — how to scope changes, write comments, verify work. That's correct for a writing assistant; it's a footgun if you only wanted diagrams.
- Output style is read once at session start. A change takes effect after `/clear` or in a new session.

---

## 3. Zero-shot vs few-shot vs chain-of-thought

**Zero-shot — just ask.** The default, and it works, because the loop explores on its own. Most Claude Code prompts are zero-shot.

**Few-shot — show the output format.** Still useful, but narrowly: commit messages, PR descriptions, structured reports — anything with a shape you care about. Examples are the strongest signal in a prompt; Claude matches their length, tone, and structure. That cuts both ways, so a stale example freezes old behavior into new work.

**Chain-of-thought — largely obsolete, and sometimes harmful.** "Think step by step" was a real technique for older models. Current models do adaptive thinking natively, so the incantation is redundant at best. Worse, step-by-step scaffolding written for older models actively reduces output quality on current ones. The modern equivalent is configuration, not prose — **effort levels** (Module 12), not "think carefully."

**The principle:** state the goal and the constraints; don't script the steps. Over-specification is now a failure mode, not a best practice.

### Making the finish line explicit with `/goal`

Stating the outcome in your prompt tells Claude what you want. `/goal` makes it a condition Claude keeps working toward **across turns**, rather than something it can decide it's finished with:

```
/goal the full test suite passes and there are no type errors
```

With no argument it shows the current or most recently achieved goal. `/goal clear` removes an active one early (`stop`, `off`, `reset`, `none`, and `cancel` all work too).

This is the constraint from the top of the module made operational. A goal is only as good as its checkability — "the suite passes" is a goal, "the code is clean" is a wish. Give it something it can actually run.

---

## 4. Common mistakes when prompting an autonomous agent

| Pattern | What it looks like | Fix |
|---|---|---|
| **The kitchen sink session** | One task, then something unrelated, then back to the first — context is full of noise | `/clear` between unrelated tasks |
| **Correcting over and over** | Correct, still wrong, correct again — context is polluted with failed approaches | After **two** failed corrections, `/clear` and rewrite the prompt with what you learned |
| **The over-specified CLAUDE.md** | Too long, so half of it gets ignored | Prune ruthlessly; convert must-happen rules into hooks |
| **The trust-then-verify gap** | Plausible-looking code that doesn't handle edge cases | Always provide verification. If you can't verify it, don't ship it |
| **The infinite exploration** | "Investigate X" with no scope; Claude reads hundreds of files | Scope narrowly, or delegate to a subagent so it doesn't consume your main context |

The two-corrections rule is the one worth internalizing, because instinct pulls the other way. A clean session with a better prompt almost always beats a long session with accumulated corrections.

**Steering tools:**
- `Esc` — stop Claude mid-action; context is preserved so you can redirect
- `Esc Esc` or `/rewind` — restore previous conversation and code state
- `"Undo that"` — have Claude revert its changes
- `/clear` — reset context between unrelated tasks
- `/btw` — ask a side question whose answer never enters conversation history

---

## 5. Hands-on: writing your first effective prompts

**Try:** run the same task twice in the same repo and compare.

```
Add a health-check endpoint to this API.
```

```
Add a health-check endpoint to this API. It should return 200 and a JSON status.
Follow the pattern in the existing routes. Add a test and run the test suite.
```

Notice which one holds up when the repo doesn't match your assumptions.

**Try:** write a third version that dictates the code line by line. Watch it produce something worse, or fight the actual shape of the codebase. That's over-specification in action.

**Try:** point Claude at a deliberately failing test and say only `Fix the failing test.` Watch it read, edit, re-run, and iterate. That's the agentic loop from Module 1, live.

**Try:** run `/init`, then delete half the generated CLAUDE.md using the "would removing this cause mistakes?" test. Confirm what loaded with `/context`.

---
<!-- nav -->

[← Welcome to the Era of Agentic Coding](module-01-agentic-era.md) · [All modules](../README.md#the-course) · [Installing Claude Code Everywhere →](module-03-installing-everywhere.md)
