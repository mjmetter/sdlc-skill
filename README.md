# SDLC Orchestrator Skill

Runs the full development workflow end-to-end — from a problem statement (or optional JIRA
ticket) to a merged pull request — using model-optimised agents at each phase.

> Extracted from internal work at Bloomreach and published with permission; history squashed for confidentiality.

## Invoke

```
/sdlc PROJ-123
/sdlc fix the cache invalidation bug in the metrics pipeline
/sdlc
```

Accepts a JIRA ticket key, a free-text problem description, or nothing (prompts
interactively).

## What you get

| Phase | What happens |
|-------|-------------|
| **0 Bootstrap** | Reads CLAUDE.md, detects language / build command / test presence, writes `conductor/tech-stack.md` |
| **1 JIRA** *(optional)* | If JIRA is configured: links to an existing ticket, creates a new one, or looks up a key you provide. Skipped entirely when JIRA is not configured. |
| **2 Brainstorm** | Explores the codebase, proposes 2–3 approaches with trade-offs, agrees direction with you |
| **3 Planning** | Opus subagent writes a detailed plan to `.claude/plans/` covering approach, tests, and PR description |
| **4 Implementation** | Sonnet subagent implements the plan (writes failing tests first when a test suite exists), runs the build — up to 3 fix-and-retry cycles |
| **5 Security Review** | Conditional: spawns a security agent if the diff touches HTTP calls, secrets, shell construction, or untrusted data parsing |
| **6 Quality Gate** | Runs `/simplify` then `/review`, applies fixes, confirms build is still green |
| **7 Commit & PR** | You approve the diff, then the skill commits, opens the PR via `gh`, and monitors CI + review threads |

## Two run modes

After the plan is presented you choose how to proceed:

- **Autonomous** — a single subagent runs Phases 4–6 unattended and reports back. You
  approve the diff once before anything is committed.
- **Step-by-step** — each phase runs with a checkpoint, giving you full visibility and
  control at every stage.

## Platform support

**GitHub.** Phase 7 (push, PR creation, CI monitoring, and review-thread resolution) uses
the GitHub CLI (`gh`). The repo must have a GitHub remote and you must be authenticated
(`gh auth login`).

## Prerequisites

| Requirement | Details |
|-------------|---------|
| `CLAUDE.md` in the project root | Strongly recommended. Run `/init` to generate one if missing. |
| GitHub CLI (`gh`), authenticated | Required for push, PR creation, and CI monitoring in Phase 7. Authenticate with `gh auth login`. |
| JIRA integration *(optional)* | Requires an Atlassian MCP server configured in your Claude Code setup, plus `JIRA_CLOUD_ID` (Atlassian API URL containing the cloud UUID), `JIRA_PROJECT` (e.g. `PROJ`), and `JIRA_PARENT` (epic key) present in CLAUDE.md. Without all three, the JIRA phases are skipped and the workflow runs without any issue-tracker coupling. |

## Skills and commands used

Phase 7 performs its own commit and `gh` PR operations inline — no external commit/PR
skill is required. The following generic Claude Code skills/commands are used when
available and skipped gracefully if not:

- `/security-review` — security analysis of the diff (Phase 5)
- `/simplify` and `/review` — code quality passes (Phase 6)
- `/init` — generates CLAUDE.md if the project lacks one

## Install

Copy this directory into your Claude Code skills folder:

```bash
cp -R sdlc-skill ~/.claude/skills/sdlc
```

Then invoke it with `/sdlc` in any project.

## License

MIT — see [LICENSE](LICENSE).
