# Labs — the practice repo

The hands-on exercises across the course assume a small repo you can safely break. Several modules reference it as `zth-demo-api`.

You can use your own project instead. If you'd rather have one that behaves the same way on your machine as it does on screen, build it to this spec.

## What it needs

From Module 4, which sets up the practice environment the later modules reuse:

- **3–4 small source files** — enough that Grep and Glob have something to find, small enough to read in full
- **One test that currently fails** — the whole gather → act → verify loop hangs off this
- **One small feature that's missing** — for example a route with no input validation
- **A test command that works from a clean clone** — `npm test`, `pytest`, whatever fits

A tiny REST API is the shape that works best: it gives you routes to search for, a validation gap to fill, and a test suite to run.

## Why the failing test matters

Module 1 makes the argument and Module 4 demonstrates it: **Claude Code works well exactly when you give it a way to check itself.** A repo with a green test suite and nothing broken can't demonstrate the loop. The failing test is the point.

## What each module needs from it

| Module | Uses the repo for |
|---|---|
| 4 | The whole module — every chapter builds on the same broken state |
| 5 | Writing a CLAUDE.md, testing permission modes |
| 6 | Plan mode, `/branch`, and watching `/rewind` fail to undo a Bash change |
| 7 | Building a `/security-review` skill and a `PreToolUse` hook |
| 9 | Somewhere to point an MCP server at |
| 15 | A starting point for Track A, if you don't have your own |

Keep the same repo across Modules 4–7. Module 4's chapters build on one fix, so reset with `git checkout .` between runs rather than starting fresh.

## Status

The starter repo isn't published yet. Until it is, use your own project — every exercise works on any repo that meets the spec above.
