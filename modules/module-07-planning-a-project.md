<!-- module: 7 | phase: 2 | format: deep-dive | last_verified: 2026-09-15 -->
# Module 7 — Planning a Complete Project With Claude Code

By this point you've used Claude Code's built-in tools on an existing codebase. This module is where you start a project from nothing — a portfolio site you'll keep building through the next two modules. The goal here isn't to write code yet. It's to get Claude Code to help you think clearly about what you're building before either of you touches a file.

## Chapter 1: Defining Requirements, Features and the Tech Stack

### Concept

Claude Code ships with a **plan mode** built for exactly this moment. When you start a session with plan mode active — pressing `Shift+Tab` to cycle to it, or launching with `claude --permission-mode plan` — Claude can read files, search the web, and reason about your project, but it cannot edit files or run changing commands. It has to lay out its thinking and show it to you before anything happens.

This matters for a from-scratch project because there's nothing to protect yet — no existing code plan mode needs to keep safe from accidental edits — but there's also nothing to react to. Left alone, an agent asked to "build me a portfolio site" will guess at scope: it'll pick a framework, invent features you didn't ask for, and skip ones you assumed were obvious. Plan mode forces the scoping conversation to happen in the open, as text you can read and correct, instead of inside decisions baked silently into code.

Feed Claude Code your raw idea in plain language — who the site is for, what you want a visitor to be able to do, any constraints you already know (a specific host, a language you want to practice, a deadline). Ask it to turn that into three things: a requirements list (what the project must do), a feature list (what a user experiences), and a proposed tech stack (language, framework, styling approach, hosting). Because it's in plan mode, it will present this as a plan rather than start building it.

### Try it yourself

Start a new session in the empty portfolio project directory and enter plan mode with `Shift+Tab`. Prompt something like: "I want to build a personal portfolio site — an About section, a projects grid pulling from a JSON file, and a contact form. I'm comfortable with JavaScript but new to frontend frameworks. Help me define the requirements, features, and a tech stack, and explain the tradeoffs of each stack option you suggest." Read the plan Claude Code returns in full before responding — don't skim past the tech stack reasoning to get to the feature list.

### Common pitfalls

- **Accepting the first stack suggestion without asking why.** A generated plan will name a framework, but the value is in the reasoning underneath it. If Claude Code recommends a static site generator over a full framework for a simple portfolio, ask it to spell out why — you want to be able to defend that choice later, not just repeat it.
- **Treating "requirements" and "features" as interchangeable.** Requirements are constraints the project must satisfy (loads fast, works on mobile, deploys as static files). Features are what a visitor does (browse projects, filter by tag, send a message). Conflating them produces a plan that's vague about both.
- **Skipping plan mode because the project feels too small to need it.** A five-page portfolio site is exactly the size where an ungoverned agent will overbuild — adding a database, an auth system, or a build pipeline you never asked for. Small projects benefit from scoping discipline just as much as large ones.

### Key takeaways

- Plan mode lets Claude Code read and reason without editing, which is what makes it safe for scoping a brand-new project out loud.
- Ask for requirements, features, and a tech stack as three distinct outputs, and ask for the reasoning behind stack choices, not just the choices themselves.
- The habit you're building here — scope before you build — is the same habit Module 8 assumes is already in place when it's time to execute.

## Chapter 2: Reviewing and Refining the Project Plan

### Concept

A first plan is a draft, not a contract. Claude Code's plan mode is built around a review step: once it presents a plan, you can push back, ask it to reconsider a piece, or ask it to fill in something it left vague — all still inside plan mode, still without touching a file. Only when you explicitly approve does the plan turn into permission to start editing.

This review step is where most of the real thinking happens. The first draft of a plan tends to be reasonable but generic — it covers the features you mentioned and misses the ones implied by them. If you asked for a contact form, does the plan account for where submissions go? If you asked for a projects grid from a JSON file, does it say who edits that file and how? Reading a plan critically means checking for the gaps between what you said and what the plan assumes.

Refining is a conversation, not a single revision. You can ask Claude Code to expand one section, cut a feature that's scope creep, or replace a stack choice — and it should regenerate just the affected part of the plan rather than starting over. Once you're satisfied, approving the plan (the CLI will prompt you for this, or you exit plan mode to let editing begin) is the moment the project actually starts.

One technique worth building into your review habit: run the plan through more than one lens before approving it, in sequence, rather than reading it once as yourself. Ask Claude Code to look at its own plan again as a senior engineer specifically hunting for concrete gaps — authentication, spam handling on that contact form, whether CI is accounted for, where secrets live — the kind of operational detail a first draft written to satisfy a feature list tends to skip. Then ask for a pass that checks the plan against current best practices for the model and for Claude Code itself, since a plan can be internally consistent and still miss a better-supported approach. Module 13 covers persona prompting as a general technique; this is that same idea applied narrowly, at the one moment — before any code exists — where catching a gap costs you nothing.

### Try it yourself

Take the plan from Chapter 1 and deliberately look for one gap: something you asked for that the plan didn't fully account for. Ask Claude Code to revise just that section — for example, "the plan doesn't say what happens to contact form submissions since this will be a static site with no backend — add that." Confirm the revised plan reads consistently with the rest before you approve it and let Claude Code exit plan mode to start building.

### Common pitfalls

- **Approving a plan you only skimmed.** Plan mode's whole value disappears if you rubber-stamp the output. Read every section, especially the tech stack and any part describing data storage or external services — those are the hardest to change once code exists.
- **Asking for a full regeneration for a small fix.** Requesting the whole plan again after one correction wastes turns and risks losing details you'd already approved elsewhere. Ask for a targeted revision instead.
- **Confusing "approved" with "final."** A plan can still change once you're building — Module 8 covers what to do when execution reveals something the plan missed. Approval means you're ready to start, not that nothing will move.

### Key takeaways

- Review a plan for gaps between what you asked for and what it assumes, especially around data storage, external services, and anything implied but not stated.
- Refine iteratively — ask for targeted changes to one section rather than regenerating the whole plan.
- Approving the plan is what lets Claude Code leave plan mode and start editing; treat that moment as deliberate, not automatic.

<!-- nav -->
[← Module 6: Built-in Tools and Working with Codebases](module-06-built-in-tools-codebases.md) | [Module 8: Executing and Building the Project →](module-08-executing-and-building.md)
