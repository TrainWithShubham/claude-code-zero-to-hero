# CLAUDE.md — Claude Code: Zero To Hero (Course Production Repo)

## What this repo is
Source content for **"Claude Code: Zero To Hero"** — a TrainWithShubham course (English channel, trainwithshubham.ai) teaching Claude Code to Developers, DevOps, Cloud, and SRE engineers. Self-paced, recorded format, 16 modules across 4 phases, with a two-track capstone (general dev vs. DevOps/SRE).

This repo holds the **written source material**: the sixteen module files, the syllabus, a condensed quick reference, and (later) generated PDF study guides. It does not hold video files or editing project files.

## Structure
```
README.md                   # Front door — module index with links to all 16
modules/                    # THE CONTENT. Sixteen module files, one per module.
                            # Source of truth for all course material.
                            # Each ends with a <!-- nav --> prev/next footer.
reference/
  topics.md                 # A–Z concept index → module. Add a row for any new concept
  commands.md               # Every slash command, CLI flag, env var → module
  quick-reference.md        # All 16 modules condensed. Derived from modules/ —
                            # when you change a module, update the matching entry here
  troubleshooting.md        # Symptom → cause → fix. Deliberately NOT a module:
                            # it's lookup material, read when something is broken
  power-moves.md            # Compound techniques that span several modules, plus
                            # anti-patterns. Also not a module — it presumes all 16
  syllabus.md               # Chapter-level outline of all 16 modules
labs/                       # Spec for the practice repo the exercises assume
```

**Gitignored, local only.** `production/` exists on disk but is deliberately kept out of the published repo (see `.gitignore`):

- `production/instructor/` — filming cues, runtimes, diagram ideas
- `production/distribution-plan.md` — working document for the plugin-marketplace and freshness plans, with open business decisions in it

Keep writing to them, but never link to them from learner-facing files — the links would 404 for anyone who clones the repo.

## When you change a module, update these too
A change in `modules/` usually needs a matching change elsewhere. In rough order of how often:

1. `reference/quick-reference.md` — the condensed version of the same fact
2. `reference/commands.md` — if a command, flag, or env var changed
3. `reference/topics.md` — if you introduced a new concept worth indexing
4. `README.md` — only if a module's one-line description changed
5. `reference/syllabus.md` — only if chapters were added, removed, or renamed
6. `reference/troubleshooting.md` / `reference/power-moves.md` — only if the change touches a documented failure mode or a technique they cite

Contradictions between these files are the main failure mode of this repo.

**Before pointing an index row at a module, confirm the module actually covers it.** `reference/commands.md` once listed `/goal` against Module 2 when Module 2 never mentioned it. An index that lies is worse than a missing row.

## Module file conventions
Every file in `modules/` follows the same shape:

1. **An HTML comment on line 1** carrying metadata:
   `<!-- module: 4 | phase: 2 | format: deep-dive | last_verified: 2026-08-12 -->`
   HTML comments are stripped before markdown renders, so learners never see it, and tooling can grep it. Update `last_verified` whenever you re-check a module's facts.
2. **The `# Module N — Title` heading**, then straight into content. No phase labels, no "verified on" banners, no "this may be outdated" hedging in the body — learners should only read what's useful to them.
3. **Two density levels**, decided per module:
   - **Deep-dive** (Modules 4, 5, 7, 9, 10, 14): per-chapter *concept → try it yourself → common pitfalls → key takeaways*. These are the hands-on modules where a learner is at a keyboard.
   - **Reference** (all others): denser notes, 3–8 bullets per topic, tables where they help. Conceptual and enumerative modules don't need "try it yourself" blocks.

## Content rules — read before editing any module content
1. **Audience is learners first.** Every note should make sense to someone who missed a sentence in the video and is reading this to catch up — not just a memory-jog for the instructor. Full sentences over cryptic fragments; explain the "why," not just the command.
2. **Claude Code ships weekly.** Before editing or adding any content that references specific commands, flags, version numbers, limits, prices, or UI behavior, verify it against the current official docs — start at `https://code.claude.com/docs/llms.txt` for the doc index, and `https://code.claude.com/docs/en/whats-new` for recent changes. Don't rely on training data alone for anything version-specific. Prefer official Anthropic docs over third-party recap blogs, which have been inconsistent.
3. **Don't ship unverifiable claims.** If a fact can't be checked against official documentation — a download statistic, an undocumented integration, a marketing number — cut it or flag it explicitly rather than stating it. Stats age badly and can't be re-checked.
4. **Keep the references in sync.** See *When you change a module, update these too* above.
5. **English, not Hinglish.** This is the trainwithshubham.ai (global/English) course, distinct from the Hindi-language main channel content.
6. **No AI-sounding language.** Practitioner voice, concrete, no filler phrases like "in today's fast-paced world" or generic transition sentences.

## Course structure reference (see reference/syllabus.md for full detail)
- **Phase 1 (Modules 1–3):** Zero → Comfortable — fundamentals, prompting for agents, install, first session
- **Phase 2 (Modules 4–9):** Comfortable → Productive — agentic loop, permissions, planning, skills/hooks, plugins, MCP
- **Phase 3 (Modules 10–13):** Productive → Advanced — multi-agent orchestration, every surface, CI/CD & security, infra/cloud/enterprise
- **Phase 4 (Modules 14–16):** Advanced → Hero — Agent SDK, two-track capstone, staying current

Everyone takes Modules 1–14; Module 15 splits into Track A (general dev) and Track B (DevOps/SRE).

## What's next for this repo
Two planned directions, both described in `production/distribution-plan.md`:

1. **Ship the repo as a plugin marketplace** so learners can install the course into their own Claude Code. Requires a `.claude-plugin/marketplace.json` and per-module skill files. **Open decision:** whether skill files become the source of truth and `modules/` is generated, or the reverse. Until that's decided, `modules/` is authoritative.
2. **Branded PDF study guides** generated from `modules/`, one per module or one combined guide, matching the ReportLab/Playwright pipeline used for other TrainWithShubham materials. When that starts:
   - Treat `modules/` as the content source of truth; the PDF step is presentation-only. Don't restyle or rewrite the markdown as part of it.
   - Ask before assuming brand colors/fonts. This course may use its own English-channel branding rather than the Hindi-channel purple/orange/gold + Poppins system — confirm rather than assume.

## Things not to do
- Don't add instructor-only production notes (filming checklists, B-roll cues, runtimes) to anything in `modules/`. That content lives in `production/instructor/`.
- Don't put phase labels, verification banners, or staleness warnings in module bodies — metadata goes in the HTML comment on line 1.
- Don't invent Claude Code commands, flags, or version numbers that haven't been checked against current docs.
- Don't merge the two capstone tracks (Module 15) into one — they're intentionally separate for the general-dev vs. DevOps/SRE audience split.
- Don't let `reference/quick-reference.md` and `modules/` drift. If you can only update one, update the module and note that the quick reference is behind.
