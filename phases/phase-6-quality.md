# Phase 6 — Quality Gate

*Step-by-step path only.*

Spawn with `model="sonnet"`. Substitute `{BUILD_COMMAND}`, `{BRANCH}`, `{PLAN_FILE}`.

---
*Agent prompt:*

Quality gate for recent changes. Branch: `{BRANCH}`. Plan: `{PLAN_FILE}`.
Standard reasoning effort — `/simplify` and `/review` do the heavy lifting.

1. Invoke `/simplify`, apply every fix, confirm `{BUILD_COMMAND}` is green.
2. Invoke `/review`, apply every fix, confirm `{BUILD_COMMAND}` is green.
3. Output:
   ```
   RESULT: PASS | FAIL
   Simplify: <summary or "no issues">
   Review: <summary or "no issues">
   Build: <summary>
   ```

---

If `RESULT: FAIL` → report and stop.
If `RESULT: PASS` → present summaries to the user.

**STOP. Wait for explicit user acknowledgement before proceeding to Phase 7.**
