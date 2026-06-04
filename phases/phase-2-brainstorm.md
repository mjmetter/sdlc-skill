# Phase 2 — Brainstorm

In the main session — no subagent. Orient yourself to the problem and align on approach
before any planning begins.

### 1. Explore

Read the most relevant code to form a grounded view:
- Re-read CLAUDE.md for architecture context if needed.
- Explore files and modules most likely involved based on `$ARGUMENTS`.
- If `$JIRA_KEY` is set, read the full comment thread — not just the description. Extract any referenced tickets, prior fix attempts, expert hypotheses, or statements like "this is actually in layer X". These are often the most valuable context and save significant misdirection.

### 2. Present analysis

```
PROBLEM: <one sentence restatement>
AREA:    <files/modules most likely involved>

FIX LAYER: <the specific component/layer where the bug originates AND where the fix must land>
           (e.g. "delivery side, Lucene query filter" not "CMS side session handling")
           If uncertain, state that explicitly — do not proceed to APPROACHES until this is resolved.

APPROACHES:
A) <name> — <one sentence trade-off>
B) <name> — <one sentence trade-off>
C) <name> — <optional third if genuinely distinct>
```

> **Hard check:** Every approach must target the `FIX LAYER` identified above. If an approach
> touches a different layer, drop it or flag the mismatch explicitly before the user decides.

### 3. Discuss

Ask:
> Which approach? Any constraints (performance, scope, backwards-compatibility, time)?

Allow back-and-forth until the user signals readiness ("go", "proceed", selects an option).

### 4. Capture direction

Summarise as `DIRECTION` — 2–4 bullets: chosen option, key constraints, anything to avoid.

```
DIRECTION:
- <bullet 1>
- <bullet 2>
...
```

This is passed verbatim to the planning agent.
