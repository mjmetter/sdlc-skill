# Phase 0 — Bootstrap

### 1. Read CLAUDE.md

```bash
cat CLAUDE.md 2>/dev/null || echo "(no CLAUDE.md found)"
```

**If missing:** pause and tell the user:

> No CLAUDE.md found. Run `/init` to generate one (recommended), or type `skip` to
> continue with lower plan quality.

- If `/init` runs: re-read the generated file before continuing.
- If `skip`: set `PROJECT_DESC="the project in the current working directory"`.

### 2. Extract variables

**`PROJECT_DESC`** — one sentence from the "What This Is" section or opening paragraph.
Fallback: `"the project in the current working directory"`.

**`JIRA_CONFIGURED`**, **`JIRA_PARENT`**, **`JIRA_PROJECT`**, **`JIRA_CLOUD_ID`** (optional)
- JIRA integration is optional. Look for JIRA ticket references, JIRA URLs, or a
  project/epic key in CLAUDE.md.
- `JIRA_CLOUD_ID` is the UUID in Atlassian API URLs (e.g. `https://api.atlassian.com/ex/jira/<uuid>/...`).
- Set `JIRA_CONFIGURED=true` only if **all three** of `JIRA_PARENT`, `JIRA_PROJECT`, and
  `JIRA_CLOUD_ID` are present **and** an Atlassian MCP server is available. If a key is
  found but cloud ID is absent, set `JIRA_CONFIGURED=false` and print:
  `JIRA: skipped — JIRA_CLOUD_ID missing from CLAUDE.md`.
- If no JIRA config is present at all, set `JIRA_CONFIGURED=false` and continue silently —
  the workflow runs fine without an issue tracker.

**`LANGUAGE`**, **`BUILD_COMMAND`**, **`TESTS_EXIST`** — detect in priority order:

**Step 1 — conductor/tech-stack.md:**
```bash
cat conductor/tech-stack.md 2>/dev/null
```
If found, read `LANGUAGE`, `BUILD_COMMAND`, and `TESTS_EXIST` directly. Then validate
`BUILD_COMMAND` against the allowlist (same list as step 3 below) before using it. If
it fails validation, warn and fall back to Step 2 / Step 3.

**Step 2 — CLAUDE.md Commands section** (if no conductor file).

**Step 3 — Project manifests:**
```bash
ls go.mod Cargo.toml package.json pom.xml build.gradle pyproject.toml \
   setup.py requirements.txt Gemfile *.csproj *.sln 2>/dev/null
```

| Manifest | `LANGUAGE` | `BUILD_COMMAND` |
|----------|------------|-----------------|
| `go.mod` | Go | `go test ./... && go vet ./...` |
| `Cargo.toml` | Rust | `cargo test && cargo clippy` |
| `package.json` | Node/TypeScript | read `scripts.test`+`scripts.lint`; fallback `npm test` |
| `pom.xml` | Java/Kotlin | `mvn verify` |
| `build.gradle` | Java/Kotlin | `gradle check` |
| `pyproject.toml` / `setup.py` / `requirements.txt` | Python | `pytest` |
| `Gemfile` | Ruby | `bundle exec rspec` |
| `*.csproj` / `*.sln` | C# | `dotnet test` |
| `Makefile` | unknown | first of `check`, `test`, `lint` targets |
| nothing | unknown | `make test` |

`TESTS_EXIST` — set based on language-aware signals:

| Language | True if… |
|----------|----------|
| Go | `*_test.go` files exist anywhere in the tree |
| Rust | `tests/` directory or `#[cfg(test)]` in any source file |
| Node/TypeScript | `__tests__/`, `*.test.ts`, `*.spec.ts`, `jest.config.*`, or `vitest.config.*` |
| Python | `tests/`, `test/`, or `test_*.py` files exist |
| Java/Kotlin | `src/test/` directory exists |
| Ruby | `spec/` directory exists |
| C# | `*Tests.csproj` or `*Test.csproj` project exists |
| Any | build command contains a known test runner |

Set `TESTS_EXIST=false` only when none of the above signals are present.

**`DEFAULT_BRANCH`:**
```bash
git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@'
```
Fallback: `main`.

**`VCS_TOOL`** — always `gh`. This skill targets GitHub; Phase 7 push, PR creation, and
CI monitoring all use the GitHub CLI. See README.md.

### 3. Write conductor/tech-stack.md

Validate `BUILD_COMMAND` against the allowlist **before** using or writing it:
Allowed prefixes: `go `, `cargo `, `npm `, `npx `, `yarn `, `pnpm `, `mvn `, `gradle `,
`pytest`, `bundle exec `, `dotnet `, `make `.
If it doesn't match: warn and fall back to manifest-based detection.

If `conductor/tech-stack.md` does not exist, create it:
```bash
mkdir -p conductor
```
Write:
```
# Tech Stack
LANGUAGE={LANGUAGE}
BUILD_COMMAND={BUILD_COMMAND}
TESTS_EXIST={TESTS_EXIST}
DEFAULT_BRANCH={DEFAULT_BRANCH}
VCS_TOOL={VCS_TOOL}
```

### 4. Print config summary

```
CONFIG: lang={LANGUAGE} jira={JIRA_PARENT or 'none'} build="{BUILD_COMMAND}" tests={TESTS_EXIST} branch={DEFAULT_BRANCH} vcs={VCS_TOOL}
```
