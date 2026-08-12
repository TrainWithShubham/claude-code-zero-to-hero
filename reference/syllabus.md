# Claude Code: Zero To Hero
### For Developers, DevOps, Cloud, SRE, and Platform Engineers — Anyone Who Wants to Actually Use Claude Code

**Format:** Self-paced, recorded | **Language:** English | **Channel:** trainwithshubham.ai
**Built from:** Anthropic's official Claude Code documentation and changelog as of August 2026

---

## How This Course Is Structured

16 modules across 4 phases. Everyone takes Modules 1–14. Module 13 is a DevOps/Cloud deep-dive every learner should still watch (even the "general dev" track — it's what makes this course different from a generic AI coding tutorial). Module 15 branches into two capstone tracks so a frontend/backend developer and an SRE both leave with a portfolio project that matches their actual job.

| Phase | Modules | What it covers |
|---|---|---|
| Zero → Comfortable | 1–3 | AI/agent fundamentals, prompt engineering for agents, installing Claude Code everywhere, your first session |
| Comfortable → Productive | 4–9 | The agentic loop and built-in tools, permissions & config, planning real projects, Skills/Hooks/Plugins, MCP |
| Productive → Advanced | 10–13 | Multi-agent orchestration, every surface (Desktop/Web/Mobile/Slack), CI/CD & security, enterprise/cloud deployment |
| Advanced → Hero | 14–16 | Agent SDK, capstone (choose your track), staying current |

---

## Module 1 — Welcome to the Era of Agentic Coding
1. What is Generative AI and a Large Language Model (quick refresher)
2. Claude's Model Lineup in 2026: Haiku 4.5, Sonnet 5, Opus 5, and the Mythos Tier
3. Claude.ai vs Claude API vs Claude Code vs Claude Cowork vs Agent SDK — Choosing the Right Surface
4. What Makes an AI System "Agentic": The Agentic Loop Explained
5. Where Claude Code Fits — For Developers, DevOps, SRE, Cloud and Platform Engineers
6. Course Roadmap: How to Use This Course and the Two Capstone Tracks

## Module 2 — Prompt Engineering for Agentic Work
1. Anatomy of a Good Prompt for an Agent (not a chatbot)
2. System Prompts, CLAUDE.md, and Output Styles — Where Instructions Actually Live
3. Zero-Shot vs Few-Shot vs Chain-of-Thought for Coding Tasks
4. Common Mistakes When Prompting an Autonomous Agent
5. Hands-On: Writing Your First Effective Prompts in Claude Code

## Module 3 — Installing Claude Code Everywhere
1. Installing on macOS, Windows, and Linux (native binaries)
2. Full Platform Tour: CLI vs VS Code vs JetBrains vs Desktop vs Web vs Mobile
3. Authentication and Account Setup — Individual, Team, Enterprise
4. Your First Claude Code Session
5. Running Claude Code on Other Providers — Bedrock, Google Cloud, Foundry, and Gateways
6. Interface Deep Dive: Interactive Mode, Keyboard Shortcuts, Themes

## Module 4 — The Agentic Loop and Built-In Tools
1. How Claude Code Works: Reasoning, Tool Use, Iteration
2. Read, Write, Edit — File Operation Tools
3. Grep and Glob — Searching Large Codebases
4. The Bash Tool and the Sandboxed Bash Tool
5. Understanding the Context Window and Prompt Caching
6. Tools Reference Walkthrough

## Module 5 — Permissions, Memory, and Configuration
1. Choosing a Permission Mode: Manual, Accept Edits, Auto Mode
2. Auto Mode Deep Dive — The New Default and How the Classifier Decides
3. CLAUDE.md Explained: User, Project, and Directory Levels
4. Auto Memory — How Claude Remembers Your Project Over Time
5. Exploring the .claude Directory and settings.json
6. Debugging Your Config: /context, /doctor, /hooks, /mcp

## Module 6 — Planning and Executing a Real Project
1. Green Field vs Brown Field Projects
2. Plan Mode vs Direct Execution vs Auto Mode — When to Use Which
3. Session Management: Resume, Branch, Rewind, Checkpointing
4. Working Across a Monorepo or Large Codebase
5. Common Workflows: Bug Fixes, Refactors, Test Generation
6. Hands-On: Planning and Building a Starter Project

## Module 7 — Extending Claude Code: Skills, Commands, and Hooks
1. Introduction to Agent Skills (SKILL.md)
2. Creating Custom Slash Commands
3. Automating Actions with Hooks (formatting, notifications, validation)
4. Output Styles — Adapting Claude Code Beyond Software Engineering
5. Hands-On: Building Your Own Skill

## Module 8 — The Plugin Ecosystem
1. What Are Plugins and Why They Matter
2. Discovering and Installing Plugins from Marketplaces
3. Creating and Distributing Your Own Plugin Marketplace
4. Claude Security Plugin — Scanning Your Codebase for Vulnerabilities
5. security-guidance Plugin — Catching Issues as Claude Writes Code
6. Hands-On: Installing and Configuring a Plugin Stack

## Module 9 — MCP (Model Context Protocol) in 2026
1. What Is MCP and Why It Became the Industry Standard
2. MCP Architecture: The 2026-07-28 Spec, Stateless Core, OAuth/OIDC
3. Connecting Your First MCP Server (claude mcp login)
4. Real-World MCP Example: AWS and Observability Tools
5. Channels — Pushing Alerts and Webhooks Into a Running Session
6. Hands-On: Wiring an MCP Server Into Your Own Stack

## Module 10 — Subagents and Orchestration at Scale
1. Creating Custom Subagents for Task-Specific Work
2. Background Subagents and Agent View — Managing Many Sessions at Once
3. Agent Teams — Coordinating Multiple Claude Code Instances
4. Dynamic Workflows — Orchestrating Dozens to Hundreds of Subagents
5. Worktrees — Running Parallel Sessions Without Collisions
6. Cross-Session Messaging — Let Sessions Talk to Each Other
7. Hands-On: Building a Multi-Agent Codebase Audit

## Module 11 — Claude Code Across Every Surface
1. Your IDE — VS Code and JetBrains
2. Claude Code Desktop: Parallel Sessions, Computer Use, In-App Browser
3. Claude Code on the Web and Routines (Scheduled Cloud Agents)
4. Claude Code on Mobile — Monitor and Steer from Your Phone
5. Remote Control — Continuing a Session From Any Device
6. Claude Tag — Bringing Claude Into Slack
7. Claude in Chrome — Testing and Debugging Web Apps

## Module 12 — CI/CD, Code Review, and Security
1. Headless Mode — Claude Code Without a Terminal (`claude -p`, `--bare`, JSON output)
2. Claude Code GitHub Actions — @claude Mentions and Auto-Fix PRs
3. Claude Code in GitLab CI/CD
4. Automated Code Review and /ultrareview — Multi-Agent Bug Hunting
5. Security Fundamentals: Sandbox Environments and Threat Models
6. Managing Costs: Token Usage, Model Selection, Effort Levels
7. Hands-On: Wiring Claude Code Into a CI/CD Pipeline

## Module 13 — Claude Code for Infrastructure, Cloud, and Enterprise
*(Watch this even if you're doing the general-dev capstone — this is the module that makes "Zero to Hero" different from every other Claude Code tutorial.)*
1. Claude Code for Infrastructure as Code: Terraform, Kubernetes, Docker
2. Deploying on Amazon Bedrock, Google Cloud's Agent Platform, and Microsoft Foundry
3. Self-Hosted Environments — Running Cloud Sessions on Your Own Infrastructure
4. Claude Apps Gateway — Centralized Credentials, Spend Limits, SSO
5. Admin Setup — Managed Settings, MCP Allowlists, Policy Enforcement
6. Observability — OpenTelemetry Monitoring and the Analytics Dashboard

## Module 14 — Building Products with the Agent SDK
1. Agent SDK Overview: Claude Code as a Library
2. The Agent Loop — Message Lifecycle and Architecture
3. Giving Claude Custom Tools and Structured Outputs
4. Subagents, Skills, and Plugins Inside the SDK
5. Hosting the Agent SDK in Production (Docker/Kubernetes)
6. Tie-In: Pairing the Agent SDK with Durable Execution (Temporal)

## Module 15 — Capstone: Choose Your Track
1. Capstone Kickoff — Planning with Plan Mode + Auto Mode
2. **Track A (General Dev):** Ship a Full-Stack SaaS Feature End-to-End with Claude Code
3. **Track B (DevOps/SRE):** Build a Self-Healing CI/CD or Incident-Response Agent
4. Build Sprint (track-specific, guided)
5. Capstone Review with /code-review and /ultrareview
6. Presenting and Documenting Your Project with Artifacts

## Module 16 — Staying Sharp: What's Next
1. Reading the Weekly Changelog — How to Stay Current With a Fast-Moving Tool
2. Claude's Expanding Surfaces — Where This Is Headed
3. Building Your Own Learning Loop and Community
4. Course Recap, Certification, and Next Steps

---

## Notes for Production
- Every module references named, current features (auto mode, agent teams, dynamic workflows, MCP 2026-07-28, Claude Security plugin, self-hosted environments, etc.) pulled from Anthropic's official docs/changelog as of Aug 12, 2026 — not third-party recap blogs, which had inconsistent claims.
- Claude Code ships weekly, so a "last verified" note in each module's description (like your existing course does) is worth keeping.
- Module 13 and 14 are the modules that differentiate this from generic "vibe coding" tutorials — worth extra production polish given your DevOps/Cloud/SRE audience overlap with Temporal/AIOps India.
- Two-track capstone (Module 15) means you'll want two guided project repos/briefs, similar to how you built the DevBoard Live hackathon brief for Udaan Batch 11.
