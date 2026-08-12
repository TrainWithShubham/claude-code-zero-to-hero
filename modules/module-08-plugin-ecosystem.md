<!-- module: 8 | phase: 2 | last_verified: 2026-08-12 -->

# Module 8 — The Plugin Ecosystem

## 1. What plugins are and why they matter

- A plugin bundles **skills, subagents, hooks, MCP servers, LSP servers, commands, output styles, and themes** into one installable unit. It's the packaging layer above everything in Module 7.
- The framing that makes it click: Module 7 was building extensions one at a time. A plugin is how you ship them together — to your team, or to the world.
- Plugins are not free. Each one adds to your context window every turn, and the install screen tells you how much before you commit.
- Plugins execute arbitrary code with your user privileges. Only install from sources you trust.

---

## 2. Discovering and installing plugins

**The official marketplace is added for you.** `claude-plugins-official` registers automatically the first time you start Claude Code interactively. If your network blocked it, add it manually:

```shell
/plugin marketplace add anthropics/claude-plugins-official
```

Then:

```shell
/plugin                                          # interactive panel
/plugin install github@claude-plugins-official
/plugin list                                     # add --enabled or --disabled
```

`/plugin` opens a four-tab panel, cycled with `Tab`: **Discover**, **Installed**, **Marketplaces**, **Errors**.

Before installing, the details pane shows:

- **Context cost** — tokens the plugin adds to your context every turn
- **Last updated**
- **Will install** — the exact commands, agents, skills, hooks, and MCP/LSP servers it adds

### Installation scopes

| Scope | Effect |
|---|---|
| **User** | You, across all projects |
| **Project** | Everyone on this repository — writes to `.claude/settings.json` |
| **Local** | You, in this repository only |
| **Managed** | Installed by administrators; can't be modified |

After installing, read the summary. If it reports `Run /reload-plugins to activate.`, run it. When a reload would invalidate the prompt cache, the command warns and skips until you rerun with `--force`.

### The three marketplaces to know

| Marketplace | How to get it | What's in it |
|---|---|---|
| `claude-plugins-official` | Added automatically | Curated by Anthropic |
| `claude-community` | `/plugin marketplace add anthropics/claude-plugins-community` | Third-party, passed automated validation, each plugin pinned to a commit SHA |
| `claude-code-plugins` | `/plugin marketplace add anthropics/claude-code` | Demo plugins showing what's possible |

### Worth installing early: code intelligence

```shell
/plugin install typescript-lsp@claude-plugins-official
```

Code intelligence plugins connect Claude to a language server, giving it **automatic diagnostics after every edit** — type errors and missing imports reported back without running a compiler — and real symbol navigation instead of grep. Plugins exist for C/C++, C#, Go, Java, Kotlin, Lua, PHP, Python, Rust, Swift, and TypeScript.

You must install the language server binary yourself; the plugin doesn't do it for you. If the **Errors** tab shows `Executable not found in $PATH`, that's why.

### Managing what you've installed

```shell
/plugin disable plugin-name@marketplace-name
/plugin enable  plugin-name@marketplace-name
/plugin uninstall plugin-name@marketplace-name
```

The **Installed** tab groups by scope and surfaces problems first. It also flags plugins you installed but haven't used in two weeks across at least 10 sessions under a **Not used recently** header — a good prompt to prune things still costing you startup and context.

---

## 3. Creating and distributing your own marketplace

A marketplace is two JSON files in a git repository.

**`.claude-plugin/marketplace.json`** at the repository root:

```json
{
  "name": "my-plugins",
  "owner": { "name": "Your Name" },
  "plugins": [
    {
      "name": "quality-review-plugin",
      "source": "./plugins/quality-review-plugin",
      "description": "Adds a quality-review skill for quick code reviews",
      "version": "1.0.0"
    }
  ]
}
```

**`plugins/<name>/.claude-plugin/plugin.json`** for each plugin:

```json
{
  "name": "quality-review-plugin",
  "description": "Adds a quality-review skill for quick code reviews",
  "version": "1.0.0",
  "author": { "name": "Your Name" }
}
```

Users then run:

```shell
/plugin marketplace add your-org/your-repo
/plugin install quality-review-plugin@my-plugins
```

Marketplaces can be added from a GitHub `owner/repo`, any git URL (including GitLab, Bitbucket, and self-hosted — append `#v1.0.0` for a specific ref), a local path, or a URL to a hosted `marketplace.json`.

### Distributing to a team

Add `extraKnownMarketplaces` to the project's checked-in settings, and everyone who trusts the repo gets prompted to install:

```json
{
  "extraKnownMarketplaces": {
    "my-team-tools": {
      "source": { "source": "github", "repo": "your-org/claude-plugins" }
    }
  }
}
```

### Two things that will catch you out

- **Users only receive updates when you bump `version` in `plugin.json`.** Without a bump, nothing changes on their machines.
- **Auto-update defaults differ by source.** Official Anthropic marketplaces auto-update. **Third-party and local marketplaces do not** — users must enable it per marketplace, or refresh with `/plugin marketplace update <name>`. Admins can set `"autoUpdate": true` on an `extraKnownMarketplaces` entry.

⚠️ **Removing a marketplace uninstalls every plugin installed from it.**

---

## 4. Claude Security plugin — scanning your codebase

```shell
/plugin install claude-security@claude-plugins-official
```

A multi-agent vulnerability scan that runs **locally in your session**. A team of Claude agents maps your architecture, builds a threat model, hunts for vulnerabilities, and independently reviews every finding before writing the report.

**Prerequisites — check these first, they block people:**

- Claude Code **v2.1.154 or later on a paid plan** (the scan uses dynamic workflows; on Pro, enable them in `/config`)
- **Python 3.9.6+** on your `PATH` as `python3` — standard library only, nothing gets installed
- **Git**, for change scans and patches. A full scan works in any directory

### Running it

The plugin adds one command, `/claude-security`, with three jobs: **Scan codebase**, **Scan changes**, **Suggest patches**. You can also ask directly — `/claude-security scan my branch`, or "scan commit abc1234".

It works best in **auto mode**, so the scan's agents aren't stopped by a permission prompt at every step.

On a large repository, scan one area at a time rather than the whole tree. The report's coverage section states what was and wasn't examined.

### Reading the results

Every scan writes a timestamped `CLAUDE-SECURITY-<timestamp>/` directory:

- **`CLAUDE-SECURITY-RESULTS.md`** — each finding's ID, impact, exploit scenario, severity, confidence, recommendation
- **`CLAUDE-SECURITY-RESULTS.jsonl`** — the same, machine-readable
- **`CLAUDE-SECURITY-REVISION-<commit>.json`** — which commit was scanned, at what effort, and how thoroughly it was verified

The directory carries its own `.gitignore`, so a stray `git add` never sweeps a report into a commit.

### Patches are never applied automatically

Each patch is drafted in a **scratch copy** of your repository, then reviewed by an agent independent of the one that wrote it — running your tests when the code has them. A patch is only written when that reviewer can vouch that it addresses the finding, introduces no new vulnerability, and leaves behavior otherwise unchanged. Otherwise you get a note explaining why instead.

```bash
git apply CLAUDE-SECURITY-<timestamp>/patches/F1.patch
```

Apply each patch in its own pull request.

### Two honest caveats

- **Scans are nondeterministic.** Two scans of the same code can surface different findings. Run them regularly and use the revision stamps to tie each report to the code it covered.
- **On Fable 5 you may see "Fable 5's safeguards flagged this message."** Its cybersecurity classifiers fire and the work is automatically downgraded to Opus. This is expected and the scan still completes.

---

## 5. security-guidance plugin — catching issues as Claude writes

```shell
/plugin install security-guidance@claude-plugins-official
```

Runs automatically once installed. Nothing to invoke, no command to remember. **Available on all plans.**

### Three layers, at three depths

| Layer | When | Cost |
|---|---|---|
| **Pattern check** | On each file edit | **No model call, no cost** |
| **Diff review** | End of each turn, in the background | One model call per turn that changes files |
| **Agentic review** | On each `git commit` / `git push` Claude makes | Several model turns |

- The **pattern check** matches known risky calls: `eval(`, `new Function`, `os.system`, `child_process.exec`, `pickle`, `dangerouslySetInnerHTML`, `.innerHTML =`, and edits under `.github/workflows/`. **It runs after the edit lands** and appends a warning to Claude's context for the next step.
- The **diff review** catches what a string match can't — authorization bypass, insecure direct object references, injection, SSRF, weak cryptography. It runs as a **separate Claude call with fresh context**, so the model that wrote the code isn't grading itself.
- The **agentic review** reads surrounding code — callers, sanitizers, related files — before deciding a finding is real, which keeps false positives low.

**None of the layers block writes or commits.** Findings reach Claude as instructions and it addresses them in conversation. Treat it as one layer of defense in depth.

Caps: 30 changed files per turn, at most 3 consecutive re-prompts, 20 commit reviews per rolling hour. Commits **you** make from your own shell — including the `!` escape — are not reviewed.

### Adding your own rules

Both extension points are additive; you can add checks but not remove built-in ones.

**`.claude/claude-security-guidance.md`** — your threat model in plain language, loaded by the model-backed reviews (8 KB cap across all locations):

```markdown
# Security guidance for this repo

- Do not log `customer_id` or `account_number` at INFO level or above.
- All routes under `/admin` must call `require_role("admin")` before any database read.
- Use `crypto.timingSafeEqual` for token comparison instead of `===`.
```

**`.claude/security-patterns.yaml`** — regex or substring rules for the per-edit check (50 rules max):

```yaml
patterns:
  - rule_name: internal_api_key
    substrings: ["sk_live_", "AKIA"]
    reminder: "Hardcoded API key prefix. Load credentials from the secret manager."
```

Disable layers individually with `ENABLE_PATTERN_RULES=0`, `ENABLE_STOP_REVIEW=0`, `ENABLE_COMMIT_REVIEW=0`, or the whole plugin with `SECURITY_GUIDANCE_DISABLE=1`.

### It is a worked example of Module 7

The plugin is built **entirely on hooks**:

| Hook event | Purpose |
|---|---|
| `SessionStart` | Bootstrap the plugin's Python environment |
| `UserPromptSubmit` | Capture the working-tree baseline the end-of-turn review diffs against |
| `PostToolUse` on `Edit`, `Write`, `NotebookEdit` | Per-edit pattern match |
| `Stop` | End-of-turn diff review, in the background |
| `PostToolUse` on `Bash`, filtered to `git commit`/`git push` | Commit and push review |

Everything Module 7 taught, shipped as a product. Its source is public — a working example of running a separate model call from a hook and feeding the result back into the session.

---

## 6. The defense-in-depth stack

| Stage | Tool | What it covers |
|---|---|---|
| In session | security-guidance plugin | Vulnerabilities in code Claude writes, fixed in the same session |
| On demand, single pass | `/security-review` | One security pass on the current branch |
| On demand, deep scan | Claude Security plugin | Multi-agent scan of a repo or diff, with reviewed findings and patches |
| On pull request | Code Review (Team and Enterprise) | Multi-agent correctness and security review with full codebase context |
| Managed | Claude Security product (Enterprise) | Hosted scanning of connected repositories |
| In CI | Your existing static analysis and dependency scanners | Language rules, supply chain, policy enforcement |

**None of these replace the others.** Each later stage catches what earlier ones miss; the value of the earlier ones is reducing what reaches them.

---

## 7. Hands-on: installing and configuring a plugin stack

**Try:** install `security-guidance`, then deliberately write a hardcoded API key. Watch the pattern check flag it and Claude fix it before the turn ends.

**Try:** add a `.claude/security-patterns.yaml` with a rule for something specific to your codebase, and trigger it.

**Try:** install `claude-security`, run `/claude-security` → **Scan changes** on a branch, read the report, then **Suggest patches** and `git apply` one.

**Try:** install a code intelligence plugin for your language, make an edit that introduces a type error, and watch Claude notice and fix it in the same turn without running a compiler.
