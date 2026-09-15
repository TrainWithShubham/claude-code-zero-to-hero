<!-- module: 15 | phase: 4 | format: reference | last_verified: 2026-09-15 -->
# Module 15 — Beyond Claude Code: Careers, Tools and What's Next

This is the last module in the course. There's no separate capstone here — the URL shortener you built in Module 12, deployed and checked in Module 14, already did that job. What's left is the part that doesn't fit neatly into a technical module: what to do with what you've learned, and how to think about your own role now that a meaningful share of the implementation work can be delegated to an agent.

## Career Applications and the Human Role in AI

- **Turn what you built into evidence, not just a memory.** The URL shortener project is a working, deployed artifact with a Git history, a CI pipeline, and a live URL — that's a stronger portfolio piece than a description of it. Push the repository publicly if the code allows it, write a short README explaining the architecture and the decisions behind it, and be ready to walk through both in an interview: what Claude Code did, what you reviewed and changed, and why.

- **Learn to talk about the workflow, not just the output.** A hiring manager evaluating you on agentic tooling cares less about "I used Claude Code to build X" and more about how you directed it — how you scoped tasks, when you used plan mode before letting it edit, how you split work across subagents, how you caught and corrected a wrong turn. That judgment is the actual skill; the code is the byproduct.

- **The human role doesn't disappear, it moves up a level.** Across this course, the pattern has been consistent: Claude Code executes, but you decide what "correct" looks like, review the diff before it merges, choose the architecture before implementation starts, and set the guardrails — permissions, hooks, CLAUDE.md rules — that keep an agent inside the lines you've drawn. Prompting well is a skill, but architecture decisions, security judgment calls, and the plain question of whether a feature should exist at all remain yours to make. Treat every one of those as a place where your review is the actual deliverable, not a formality before shipping.

- **Apply this beyond web projects.** Nothing about persona prompting, parallel subagents, context management, or CI/CD is specific to a URL shortener. The same patterns apply to internal tooling, infrastructure automation, data pipelines, or a DevOps/SRE workflow — defining a role for an agent, giving it a scoped task, and reviewing its output before it touches production is the same discipline whether the output is a web endpoint or a Terraform plan.

- **Build a habit of staying current, because this tool changes weekly.** Claude Code ships new features, flags, and default behaviors on a weekly cadence — the kind of pace where something you learned two months ago may already have a better way of doing it. Check `https://code.claude.com/docs/en/whats-new` periodically; it's a running weekly digest of what shipped and why it matters, with links to the full documentation for anything you want to go deeper on. When you're unsure whether a command, flag, or behavior described anywhere — including in this course — still holds, that page and the full doc index at `https://code.claude.com/docs/llms.txt` are the way to check rather than guess.

- **Don't treat any single course, including this one, as a finished picture.** The modules here cover the fundamentals and the workflows that were current when they were written. The discipline that keeps that useful — verifying specifics against current docs before relying on them, treating your own judgment as the layer that doesn't get automated — is the actual takeaway to carry forward, more than any individual command.

Thanks for working through all fifteen modules. The tool will keep changing; the way you've learned to work with it — defining scope, reviewing output, keeping the judgment calls in your own hands — is what carries over.

<!-- nav -->
[← Module 14: CI/CD and Reliability](module-14-cicd-and-reliability.md)
