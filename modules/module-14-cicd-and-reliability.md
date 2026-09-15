<!-- module: 14 | phase: 4 | format: deep-dive | last_verified: 2026-09-15 -->
# Module 14 — CI/CD and Reliability

The URL shortener runs on your machine. This module takes it from "works locally" to "deployed and checked automatically" — connecting the repository to GitHub so pull requests get reviewed on the way in, standing up a production database and deploying to Vercel, and then stepping back to the concepts that make any of that worth doing: CI/CD as a pipeline, and the SRE vocabulary — SLI, SLO, SLA — that defines what "reliable" actually means for a service.

## Chapter 1: GitHub Repository and Automated Checks

### Concept

Claude Code has a dedicated GitHub integration, shipped as a GitHub Action (`anthropics/claude-code-action`) that runs Claude Code inside your repository's workflows. Once it's set up, mentioning `@claude` in a pull request or issue comment has Claude analyze the code, make changes, and push commits — directly from GitHub, without anyone opening a local session. You can also configure it to run automatically on any GitHub event with a fixed prompt, which is what turns it into an automated check rather than an on-demand assistant.

The fastest way to set this up is `/install-github-app`, run from inside a Claude Code session in the repository you want to connect. It requires the GitHub CLI (`gh`) installed and authenticated, and admin access to the repository. It installs the Claude GitHub App, sets up an authentication secret (either an `ANTHROPIC_API_KEY` or a `CLAUDE_CODE_OAUTH_TOKEN` generated from your Claude subscription), and opens a pull request with the workflow file already wired to that secret. Merging that PR is what turns `@claude` on for the repository.

For automated checks specifically — the kind that run on every pull request without anyone asking — the workflow can run in **automation mode**: give it a `prompt` input instead of waiting for a mention, and it fires on whatever GitHub event triggers the workflow (a new PR, a push, a schedule). A review-focused version of this posts inline comments directly on the pull request, flagging issues the way a human reviewer would, while a scheduled version can generate a daily summary of activity with no PR involved at all. This is distinct from Claude Code's separate **Code Review** feature, which reviews every pull request automatically without you writing or maintaining a workflow file at all — the GitHub Action gives you full control over the trigger and prompt, Code Review trades that control for zero setup.

Two things matter for keeping the automation safe and cheap: the action checks that whoever triggered it has write access to the repository (so outside comments on a public repo can't run arbitrary Claude sessions against your code), and each run consumes both GitHub Actions minutes and API tokens, so setting `--max-turns` and workflow-level timeouts keeps a misbehaving prompt from running away.

### Try it yourself

In a repository you control, run `/install-github-app` and walk through the setup. Pick the review workflow if it's offered. Once the PR it opens is merged, open a small pull request against the repo and watch what happens automatically — either an `@claude` mention response you trigger yourself, or an automatic review comment, depending on which workflow you chose.

### Common pitfalls

- **Committing an API key directly into a workflow file instead of a GitHub secret.** Always reference it as `${{ secrets.ANTHROPIC_API_KEY }}` or the equivalent OAuth token secret — never paste the key itself into a `.yml` file.
- **Expecting CI to trigger on commits Claude's GitHub Action makes.** GitHub doesn't fire workflows from commits authenticated with the default `GITHUB_TOKEN` — if your CI isn't running after Claude pushes, check whether the action is using that token instead of authenticating as the Claude GitHub App.
- **Granting the shared Claude GitHub App broader permissions than you're comfortable with.** It's a single permission set shared across several Claude GitHub features; if your organization wants a narrower footprint, a custom GitHub App scoped to just Contents, Issues, and Pull requests covers the GitHub Action alone.

### Key takeaways

- `/install-github-app` is the fastest path to connecting a repository — it needs `gh` authenticated and admin access, and it handles the GitHub App install, the secret, and the workflow PR for you.
- Automation mode (a fixed `prompt` input) is what turns the integration into an unattended check on every PR or on a schedule, separate from the on-demand `@claude` mention flow.
- Claude Code's separate Code Review feature reviews every PR with no workflow file to maintain — pick it over a custom GitHub Action workflow when you don't need control over the trigger or prompt.

## Chapter 2: Production Database and Vercel Deployment

### Concept

Getting the URL shortener into production means two separate pieces: a real, persistent database instead of the local one you developed against, and a deployment target that serves the app to the outside world. Vercel is a common choice for the second half of that when your project is a web app — it builds from your Git repository and gives every push a deployment, with production deployments tied to your default branch.

Claude Code itself doesn't have a documented Vercel-specific integration beyond what it can already do as a general coding agent: reading your project's configuration, running the Vercel CLI on your behalf if it's installed, editing environment-variable references, and writing or fixing the code that a deployment build fails on. Treat "deploy to Vercel" the same way you'd treat any other command-line task you hand to Claude Code — describe the outcome, let it run the standard Vercel CLI commands (`vercel link`, `vercel env`, `vercel deploy`, `vercel --prod`) with your approval, and review what it changes in your project's Vercel configuration files before merging.

The database side is where most of the real work happens for a project like the URL shortener. A production database is a different animal from a local one: it needs its own connection string held as an environment variable (never committed to the repository or hardcoded into a config file), a migration step so your schema exists there before the app tries to use it, and enough sizing headroom that a first burst of traffic doesn't take it down. Whichever managed database provider you choose, the same shape of work applies — provision the instance, capture its connection string as a secret in both your local `.env` (gitignored) and your deployment platform's environment variable settings, run migrations against it, and only then point the deployed app at it.

### Try it yourself

Take the URL shortener from Module 12 and get it in front of a real URL. Provision a managed database instance (any provider you're comfortable with), set its connection string as an environment variable in your deployment platform rather than in code, deploy the app, and confirm a shortened link created through the deployed app persists across a redeploy — proof that it's actually writing to the production database and not falling back to a local one.

### Common pitfalls

- **Hardcoding a database connection string or API key into a config file that gets committed.** Every secret belongs in environment variables, set through your deployment platform's dashboard or CLI, never checked into Git — Claude Code should never be the one adding a raw credential to a tracked file.
- **Deploying before running migrations against the production database.** A deploy that succeeds but points at an empty schema fails the moment a user tries to write to it — migrations are a separate, ordered step before traffic hits the new deployment.
- **Treating a preview deployment as production-ready without checking which database it's pointed at.** A preview build using a shared staging database can quietly corrupt data that a demo or a reviewer relies on — keep production and preview environment variables distinct.

### Key takeaways

- Claude Code has no documented Vercel-specific feature beyond acting as a general coding agent that can run the Vercel CLI and edit configuration on your behalf — deployment itself follows standard Vercel practice.
- A production database needs its own connection string as an environment variable, a migration step before the app depends on it, and separation from whatever database your local development or preview environment uses.
- Never commit secrets — database URLs, API keys, deployment tokens — into the repository; they belong in your platform's environment variable settings.

## Chapter 3: CI/CD and Reliability Basics (SRE, SLI, SLO, SLA)

### Concept

**CI/CD** is shorthand for two connected practices. **Continuous Integration** means every change gets merged frequently and verified automatically — a pipeline runs on each push or pull request, typically in stages: build the project, run automated tests, run static checks or linters, and (as covered in this module) run an automated review. Failing any stage blocks the merge, which is the point — catching a broken build or a failing test before it reaches anyone else's branch, rather than after. **Continuous Deployment** (or **Continuous Delivery**, depending on how much human approval sits in the loop) extends that pipeline past merge, automatically building and shipping the change to a staging or production environment once it passes.

**Site Reliability Engineering (SRE)** is the discipline of running production software with reliability treated as a measurable, budgeted property rather than a vague aspiration. Three terms carry that discipline:

| Term | What it is | Example |
|---|---|---|
| **SLI** (Service Level Indicator) | A measured metric describing some aspect of the service's behavior | The percentage of HTTP requests to the URL shortener that return successfully within 300ms |
| **SLO** (Service Level Objective) | An internal target for an SLI, set by the team running the service | 99.9% of requests succeed within 300ms over a rolling 30-day window |
| **SLA** (Service Level Agreement) | An external, often contractual, commitment to customers — usually looser than the SLO, with consequences for missing it | 99.5% uptime per month, with service credits owed if it's missed |

The relationship between the three is deliberate: the SLA is a promise to someone outside the team, so it's set with margin below the SLO you actually target internally, which itself is set with margin below the ceiling your SLIs could theoretically hit. That gap is what SRE calls an **error budget** — the amount of unreliability you're allowed to spend, on deploys, on experiments, on accepted risk, before you're at risk of breaking the promise you made externally. A team that's spent its error budget for the period slows down on risky changes; a team with budget to spare has room to ship faster.

Tying this back to the pipeline: CI/CD is one of the main levers for protecting an SLO. Automated tests and reviews on every PR catch regressions before they ship; automated deployment reduces the manual, error-prone steps that tend to cause outages; and a fast, reliable rollback path (redeploying a previous build) matters as much as the forward path when something does slip through.

### Concept — applying it to the URL shortener

Concretely, for a small project like the URL shortener: an SLI might be "percentage of redirect requests that return within 200ms," measured from your hosting platform's request logs. An SLO built on that might be "99% of redirects complete within 200ms, measured weekly" — a number you pick based on what your users actually need, not an arbitrary round figure. You likely wouldn't publish a formal SLA for a personal or course project, but the exercise of defining one clarifies what "good enough" means before you're debugging a real incident and guessing.

### Common pitfalls

- **Setting an SLO before you have any SLI data.** Without a baseline measurement, an SLO target is a guess — measure first, then set a target you can defend.
- **Confusing SLA with SLO.** The SLA is the outward-facing promise with consequences attached; the SLO is the internal target you actually work to, and it should be stricter than the SLA so you have room to notice and fix problems before you breach the external commitment.
- **Treating 100% reliability as the goal.** Chasing zero downtime trades away development speed for marginal reliability gains nobody asked for — the error budget concept exists specifically to make that tradeoff explicit and deliberate instead of accidental.

### Key takeaways

- CI/CD is a pipeline: build, test, check, and (for CD) deploy automatically, with each stage able to block the next.
- SLI is what you measure, SLO is your internal target for it, and SLA is the external promise — each one looser than the last, with the gap forming your error budget.
- Automated pipelines are a reliability tool, not just a convenience: catching regressions before they ship, and having a fast rollback path, are how CI/CD protects the SLO you've set.

<!-- nav -->
[← Module 13: Prompting Strategies and Multi-Agent Architecture](module-13-prompting-strategies-multiagent.md) | [Module 15: Beyond Claude Code: Careers, Tools and What's Next →](module-15-beyond-claude-code.md)
