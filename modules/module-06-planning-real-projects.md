<!-- module: 6 | phase: 2 | last_verified: 2026-08-12 -->

# Module 6 — Planning and Executing a Real Project

## 1. Green field vs brown field

- **Green field** — Claude scaffolds from nothing. Fewer constraints, but no existing patterns to follow, so it invents conventions you may not want. Say what you want up front.
- **Brown field** — an existing codebase. Claude has to explore and infer conventions before it can safely change anything. This is where a good CLAUDE.md pays for itself, and where "reference existing patterns" from Module 2 does most of the work.
- Most real work is brown field. The habit that matters: point Claude at a good example in the codebase and tell it to follow that pattern.

---

## 2. Plan mode vs direct execution vs auto mode

The four-phase workflow:

1. **Explore** — enter plan mode with `Shift+Tab` (or `claude --permission-mode plan`). Claude reads files and answers questions without changing anything.
2. **Plan** — ask for an implementation plan. Press **`Ctrl+G`** to open it in your editor and hand-edit before approving. This is the single most useful plan-mode habit.
3. **Implement** — approve the plan, or `Shift+Tab` out, and let Claude build against it.
4. **Commit** — `commit with a descriptive message and open a PR`.

Approving a plan offers several exits: start in auto mode, accept edits, review manually, keep planning, or refine with Ultraplan in the browser. Accepting a plan also names the session from the plan content, if you haven't named it yourself.

**When to skip planning.** Plan mode adds overhead. For a typo, a log line, or a rename, just ask. The rule of thumb: **if you could describe the diff in one sentence, skip the plan.** Planning earns its cost when you're unsure of the approach, the change spans several files, or you don't know the code.

**For changes that span packages**, ask Claude to write the plan to a markdown file in the repo. A long session compacts as it goes, and a saved plan survives where conversation history may not.

---

## 3. Session management

### Resuming

| Command | What it does |
|---|---|
| `claude --continue` | Resumes the most recent session in this directory |
| `claude --resume` | Opens the session picker |
| `claude --resume <name>` | Resumes a named session directly |
| `claude --from-pr <number>` | Picker filtered to sessions linked to that pull request |
| `/resume` | Switch conversations from inside a session |

A resumed session restores conversation history, model, agent, permission mode, and any active goal. **`plan` and `bypassPermissions` are never restored.** Flags like `--mcp-config`, `--settings`, and `--add-dir` aren't restored either — pass them again.

### Naming

Name sessions and they become findable and resumable:

- `claude -n auth-refactor` at startup
- `/rename auth-refactor` during a session
- `Ctrl+R` on a highlighted session in the picker

Unnamed sessions get an AI-generated title, but **that isn't a resume handle** — only names you set can be resumed by name.

### Picker shortcuts

`Ctrl+W` widens to all worktrees of the repo · `Ctrl+A` to every project on the machine · `Ctrl+B` filters to the current git branch · `Space` previews · `/` searches. You can paste a PR or MR URL into search to find the session that created it.

### Branching

**The command is `/branch`, not `/fork`.**

```
/branch try-streaming-approach
```

Branching copies the conversation so far and **switches you into the copy**, leaving the original intact and still in the picker. Use it to try a different approach without losing the path you were on. From the CLI it's a flag: `claude --continue --fork-session`.

What carries over:

| State | After `/branch` |
|---|---|
| Conversation history | Copied up to the branch point |
| "Allow for this session" grants | Carried over — same process. With `--fork-session` it's a new process, so you re-approve |
| In-flight background subagents | Keep running; output appears in the **branch**, not the original |
| Remote Control connection | Follows you into the branch |

### Resume from a summary

On Pro or Max, resuming a session that's been idle for over an hour and is over 100,000 tokens opens a dialog:

- **Resume from summary** — runs `/compact` immediately; later requests carry the summary, so they cost less
- **Resume full session as-is** — keeps every detail; per-request cost scales with conversation size
- **Don't ask me again**

The prompt cache has expired by then, so the next request reprocesses the full history either way.

### Checkpointing and `/rewind`

Every prompt creates a checkpoint. Claude Code keeps file snapshots for the **100 most recent** checkpoints, saved with the conversation and deleted after 30 days.

Run `/rewind`, or press `Esc` twice on an empty prompt, and choose:

- **Restore code and conversation**
- **Restore conversation** (keep current code)
- **Restore code** (keep the conversation)
- **Summarize from here** — compress everything after this point
- **Summarize up to here** — compress everything before it

If you ran `/clear` earlier in the same process, the menu also offers an entry to resume the conversation from before the clear.

**The limitations decide whether this is a real safety net:**

- **Bash command changes are not tracked.** `rm`, `mv`, and `cp` run by Claude cannot be undone through rewind.
- **Subagent edits are not restored**, except a foreground forked skill. Background subagents and `/code-review --fix` need git.
- **External changes are not tracked** — your own manual edits, or edits from another session.
- **Symlinked and hard-linked paths are skipped**, with a `Restored the code, but skipped N files` warning. This hits dotfile managers and pnpm.
- **It is not a replacement for version control.** Checkpoints are session-level recovery; git is history.

---

## 4. Working across a monorepo or large codebase

### Where you start Claude decides everything else

| Start from | File access | CLAUDE.md loaded at launch |
|---|---|---|
| **Repository root** | Every file | Root only; subdirectory files load on demand |
| **A subdirectory** | That subtree only, until you grant more | That directory's, plus every ancestor's |

**The gotcha:** CLAUDE.md files are inherited from parent directories. **`.claude/settings.json` is not.** Project settings load only from the directory you start in, so a root `settings.json` does nothing when you start from `packages/api/`.

### The toolkit

| Want | Use |
|---|---|
| Only the conventions for code you touch | Per-directory `CLAUDE.md` |
| Skip packages you never work in | `claudeMdExcludes` |
| Block reads of generated and vendored code | `Read` deny rules in `permissions.deny` |
| Find symbols without scanning files | A code intelligence plugin — `/plugin install typescript-lsp@claude-plugins-official` |
| Check out only the directories a task needs | `worktree.sparsePaths` |
| Avoid duplicating `node_modules` per worktree | `worktree.symlinkDirectories` |
| Reach a sibling package or another repo | `permissions.additionalDirectories`, or `--add-dir` |
| Area-specific procedures | Per-directory skills in `packages/api/.claude/skills/` |

A subtle but important difference: `additionalDirectories` grants **file access only** — it never loads that directory's CLAUDE.md, rules, or skills. `--add-dir` does load skills, and loads CLAUDE.md only with `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1`.

### Per-directory vs path-scoped

Both target instructions at part of the tree:

| Approach | Lives | Loads when | Use when |
|---|---|---|---|
| Per-directory `CLAUDE.md` | Alongside the code | At launch if you start there; on demand when Claude reads a file there | Directory owners maintain their own conventions |
| Path-scoped rule in `.claude/rules/` | Central `.claude/` at the root | When Claude touches a file matching the `paths:` glob | You want conventions in one place, or one rule covers scattered paths |

---

## 5. Common workflows

The docs' **Common workflows** page has copy-paste recipes for bug fixes, refactors, and test generation. It's a better starting point than writing a prompt from scratch.

Pair those recipes with the Module 2 patterns:

- **Bug fixes** — describe the symptom, point at the likely location, ask for a failing test first, then the fix
- **Refactors** — point at the pattern to follow, name what's out of scope, ask for the test suite to be run after
- **Test generation** — name the file, the scenario, and any preferences ("avoid mocks")

For cross-package changes, hand Claude the whole change in one session rather than package by package — that keeps the decisions behind each edit consistent instead of re-derived.

---

## 6. Hands-on: planning and building a starter project

**Try:** `/plan` a small feature, press `Ctrl+G` to review and edit the plan, approve it into `acceptEdits`, and build it.

**Try:** mid-task, run `/branch experiment` and take a different approach. Then `/resume` the original by name and compare the two.

**Try:** make an edit, run `/rewind`, and choose **Restore code**. Then have Claude change a file using a Bash command instead and try to rewind that — watch it *not* be restored. That contrast is the clearest way to learn where the safety net ends.

**Try:** in a monorepo, start Claude once from the repo root and once from a package directory. Run `/context` in each and compare the **Memory files** list.
