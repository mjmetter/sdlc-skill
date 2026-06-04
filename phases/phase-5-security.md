# Phase 5 — Security Review (conditional)

*Step-by-step path only.*

### Evaluate in the main session

```bash
git diff {DEFAULT_BRANCH}...HEAD --name-only
```

Read each changed file. Check for:

- **HTTP/API calls** — `requests`, `httpx`, `fetch`, `axios`, `net/http`, `reqwest`,
  `HttpClient`, `OkHttp`, `Retrofit`, `Net::HTTP`, `httparty`, etc.
- **Credential handling** — env vars, `.env` files, auth headers, API keys, tokens
- **Shell construction** — `subprocess`, `exec`, `os.system`, Bash calls with external input
- **Untrusted data parsing** — CSV, JSON, XML, file uploads from network or users

**If none match:** note "Security review skipped" and proceed to Phase 6.

> **Note:** this heuristic matches library names in the diff. Code using internal helpers
> or framework abstractions may not match. For sensitive projects, always run
> `/security-review` manually.

### Spawn security agent (only if triggered)

`model="sonnet"`. Substitute `{BRANCH}`, `{DEFAULT_BRANCH}`, `{BUILD_COMMAND}`.

---
*Agent prompt:*

Security review of recent changes. Branch: `{BRANCH}`. Use maximum reasoning effort.

1. `git diff {DEFAULT_BRANCH}...HEAD` — full diff.
2. Invoke `/security-review` — apply every fix it recommends.
3. Run `{BUILD_COMMAND}` — must be green after fixes.
4. Output:
   ```
   RESULT: PASS | FAIL
   Issues: <bullet list or "none found">
   Build: <summary>
   ```

---

If `RESULT: FAIL` → report and stop. If `RESULT: PASS` → proceed to Phase 6.
