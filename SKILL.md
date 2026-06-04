---
name: sdlc
description: Orchestrates the full development workflow end-to-end — (optional JIRA ticket) → brainstorm → plan (Opus) → implement → security review → quality gate → commit → PR. Use when starting any development task. Accepts a JIRA ticket key (e.g. PROJ-123), a problem description, or runs interactively when invoked with no arguments. Reads project context from CLAUDE.md at runtime.
argument-hint: <PROJ-123 or problem description>
---

# SDLC Orchestrator

Runs the full development workflow for the current project. Execute phases in order.
Never skip a phase. Never proceed to the next phase without the required approval.

## Phases

Read and execute each phase file in sequence:

| Phase | File | What happens |
|-------|------|--------------|
| 0 — Bootstrap | [phases/phase-0-bootstrap.md](phases/phase-0-bootstrap.md) | Read CLAUDE.md, detect stack, write `conductor/tech-stack.md` |
| 1 — JIRA (optional) | [phases/phase-1-jira.md](phases/phase-1-jira.md) | Link to existing ticket, look up a key, or create a new one — skipped if JIRA is not configured |
| 2 — Brainstorm | [phases/phase-2-brainstorm.md](phases/phase-2-brainstorm.md) | Explore problem area, propose approaches, capture `DIRECTION` |
| 3 — Planning | [phases/phase-3-planning.md](phases/phase-3-planning.md) | Opus subagent writes plan with coverage table; choose run mode |
| 4 — Implementation | [phases/phase-4-implementation.md](phases/phase-4-implementation.md) | Sonnet subagent implements the plan (step-by-step path only) |
| 5 — Security Review | [phases/phase-5-security.md](phases/phase-5-security.md) | Conditional security agent if sensitive code detected |
| 6 — Quality Gate | [phases/phase-6-quality.md](phases/phase-6-quality.md) | `/simplify` + `/review`, fixes applied, build confirmed green |
| 7 — Commit & PR | [phases/phase-7-commit-pr.md](phases/phase-7-commit-pr.md) | Diff approval, commit, PR creation, CI monitoring |

## Context variables

Phase 0 detects and sets these; all later phases reference them:

| Variable | Set in | Source |
|----------|--------|--------|
| `PROJECT_DESC` | Phase 0 | CLAUDE.md opening paragraph |
| `LANGUAGE`, `BUILD_COMMAND`, `TESTS_EXIST` | Phase 0 | conductor/tech-stack.md → CLAUDE.md → manifest detection |
| `DEFAULT_BRANCH` | Phase 0 | `git symbolic-ref refs/remotes/origin/HEAD` |
| `VCS_TOOL` | Phase 0 | Always `gh` — GitHub (see README.md) |
| `JIRA_CONFIGURED`, `JIRA_KEY`, `JIRA_CLOUD_ID`, `JIRA_PROJECT`, `JIRA_PARENT` | Phase 1 | CLAUDE.md + user input (optional) |
| `DIRECTION` | Phase 2 | Agreed approach bullets from brainstorm |
| `PLAN_FILE`, `BRANCH` | Phase 3 | Planning agent output |
| `PR_TITLE`, `PR_BODY` | Phase 7 | Commit subject + plan sections |

## Run modes

After Phase 3 presents the plan the user chooses:

- **Autonomous** — single Sonnet subagent runs Phases 4–6 unattended; main session
  handles diff approval, commit, and PR (jumps straight to Phase 7 on pass)
- **Step-by-step** — each phase runs with an explicit checkpoint
