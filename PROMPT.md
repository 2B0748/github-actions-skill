# GitHub Actions Workflow Skill

> Platform-agnostic AI skill — drop into any AI coding assistant.
> Compatible with: Cursor Rules · GitHub Copilot · Windsurf · Aider · Cline · CodeBuddy · Custom System Prompt

## 1. Trigger Conditions

**Activate this skill when ANY of the following apply:**

- User asks to "create", "generate", or "write" a GitHub Actions workflow
- User asks to "review", "optimize", or "fix" CI/CD config, or to "audit" workflow security
- User requests automation: automated testing, releases, labeling, or dependency updates
- User mentions specific Actions: `actions/checkout`, `actions/cache`, `actions/upload-artifact`
- User mentions workflow keywords: `on:`, `jobs:`, `runs-on`, `permissions`, `concurrency`
- User asks to "automate" or "put into CI" a manual process

**Do NOT trigger when:** the user discusses other CI platforms (Jenkins, GitLab CI, CircleCI), local scripts, or Docker builds unrelated to GitHub Actions.

---

## 2. Workflow Decision Tree

When generating a workflow, match the request to one of five templates:

```
User Request
│
├─ Mentions "PR" / "push" / "lint" / "test"
│  └─ → CI Workflow (Template A)
│
├─ Mentions "Tag" / "Release" / "publish" / "package"
│  └─ → Release Workflow (Template B)
│
├─ Mentions "cron" / "scheduled" / "periodic" / "dependency" / "Dependabot"
│  └─ → Scheduled Workflow (Template C)
│
├─ Mentions "security" / "scan" / "CodeQL" / "vulnerability"
│  └─ → Security Workflow (Template D)
│
└─ Mentions "Issue" / "triage" / "auto-label" / "auto-reply"
   └─ → Event-driven Automation Workflow (Template E)
```

---

## 3. Generation Rules

### 3.1 Workflow Generation Steps

**Execute in order — do not skip any steps:**

1. **Identify the runtime environment.** Determine the project language/framework. If the user has not specified it, inspect repo files (`package.json`, `go.mod`, `Cargo.toml`, `requirements.txt`, etc.).
2. **Determine the trigger events.** Select the `on:` block based on the decision tree above.
3. **Choose the runner.** Default to `ubuntu-latest` unless the user explicitly requests macOS or Windows.
4. **Generate the YAML.** Start from the matching template below, then fill in language-specific tools and commands.

### 3.2 Mandatory Configuration

**Every generated workflow MUST include all three of the following — no exceptions:**

| Config | Scope | Rule |
|--------|-------|------|
| `permissions` | Workflow or job level | Always declare explicitly; follow the principle of least privilege |
| `concurrency` | Workflow or job level | Auto-cancel stale runs for the same branch or PR |
| Action pinning | Every `uses:` step | Pin to a full SHA-1 commit hash; never use `@main` or `@master` |

### 3.3 Template A — CI Workflow

```yaml
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.ref }}
  cancel-in-progress: true

permissions:
  contents: read

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
      # Insert language-specific setup and lint command here

  test:
    needs: lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
      # Insert language-specific setup and test command here
      # MUST include cache configuration (see 3.4)
```

### 3.4 Cache Strategy by Language

| Language / Ecosystem | Setup Action | Cache Configuration |
|----------------------|-------------|---------------------|
| Node.js (npm) | `actions/setup-node@v4` | `cache: 'npm'` |
| Node.js (pnpm) | `actions/setup-node@v4` | `cache: 'pnpm'` |
| Python | `actions/setup-python@v5` | `cache: 'pip'` |
| Go | `actions/setup-go@v5` | `cache: true` |
| Rust | `actions-rust-lang/setup-rust-toolchain@v1` | Manual `actions/cache@v4` for `~/.cargo` and `target/` |
| Docker | N/A | Use `docker/build-push-action` with `cache-from` / `cache-to` |

### 3.5 Template B — Release Workflow

```yaml
name: Release

on:
  push:
    tags:
      - 'v*'

permissions:
  contents: write

jobs:
  release:
    runs-on: ubuntu-latest
    concurrency:
      group: release-${{ github.ref }}
      cancel-in-progress: false  # Releases must never be canceled

    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
      # Build -> create GitHub Release -> upload artifacts
```

### 3.6 Template D — Security / CodeQL

```yaml
name: CodeQL

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '31 7 * * 3'  # Every Wednesday

permissions:
  security-events: write
  contents: read

jobs:
  analyze:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        language: ['javascript', 'python']  # Adjust to your project
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
      - uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}
      - uses: github/codeql-action/analyze@v3
```

---

## 4. Before / After

### Example 1 — Least-Privilege Permissions

```yaml
# Before -- no permissions declared; inherits default write access (security risk)
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

# After -- explicit read-only permissions
permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
```

### Example 2 — Concurrency Control

```yaml
# Before -- no concurrency; each push queues a new run, wasting minutes
name: CI
on: [pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps: [ ... ]

# After -- new commits automatically cancel in-progress runs
name: CI
on: [pull_request]

concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.ref }}
  cancel-in-progress: true

jobs:
  test:
    runs-on: ubuntu-latest
    steps: [ ... ]
```

### Example 3 — Action Version Pinning

```yaml
# Before -- floating version tags; supply-chain attack risk
- uses: actions/checkout@v4
- uses: docker/setup-buildx-action@master

# After -- pinned to immutable commit SHA
- uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
- uses: docker/setup-buildx-action@c47758b77c9736f4b2ef4073d4d51994fabfe349 # v3.8.0
```

---

## 5. Few-Shot Examples

### Example 1 — Most Common: "Add CI to my Node.js project"

**User Input:**
> Write me a GitHub Actions workflow that runs eslint and jest on PRs

**What the skill produces:**
1. Detect `package.json` -> Node.js project
2. Select Template A (CI)
3. Generate a full workflow with: `permissions`, `concurrency`, `cache: 'npm'`, `npm ci` -> lint -> test pipeline

---

### Example 2 — Variation: "Weekly dependency check"

**User Input:**
> I want GitHub to check for npm dependency updates every Monday

**What the skill produces:**
1. Select Template C (Scheduled)
2. Generate `.github/workflows/dependency-update.yml`
3. Use `cron: '0 9 * * 1'` + `actions/checkout` + `npm outdated`, auto-create a PR if updates are found
4. Permissions scoped to `contents: write` and `pull-requests: write`

---

### Example 3 — Edge Case: "My CI takes 15 minutes"

**User Input:**
> My CI takes 15 minutes per run -- how do I speed it up?

**What the skill produces:**
1. Audit the existing workflow file
2. Check for missing cache config -> add the language-appropriate cache directive
3. Identify serial bottlenecks -> split lint and test into parallel jobs, remove unnecessary `needs` chains
4. Estimate time savings for each change

---

### Example 4 — Security Audit: "Is my workflow secure?"

**User Input:**
> Audit `.github/workflows/ci.yml` for security issues

**What the skill produces:**

| Check | Status | Detail |
|-------|--------|--------|
| `permissions` declared | No | Not set -- inherits write permission |
| Action versions pinned | No | `actions/checkout@v4` -- not SHA-pinned |
| `concurrency` configured | Yes | Present |
| Secret handling | Yes | Uses `${{ secrets.X }}`, no plaintext |
| `pull_request_target` risk | Yes | Not used (safe) |

---

### Example 5 — Multi-language: "Rust + Node.js monorepo CI"

**User Input:**
> My project has a Rust backend and a React frontend -- how do I write a single CI?

**What the skill produces:**
1. Detect `Cargo.toml` + `package.json` at the repo root
2. Split into two parallel jobs (or use a matrix strategy)
3. Rust job: manual `actions/cache@v4` for `~/.cargo` and `target/`
4. Node.js job: `setup-node` with built-in `cache: 'npm'`
5. Both jobs run in parallel, each scoped to its own `working-directory`

---

## 6. Checkpoints

| CP | Stage | What to Verify | How to Verify |
|----|-------|----------------|---------------|
| CP1 | After template selection | `on:` events match the user's request | Cross-check against the decision tree in Section 2 |
| CP2 | After YAML generation | All three mandatory configs are present | Search the file for `permissions:`, `concurrency:`, and `# v` (version comment) |
| CP3 | After YAML generation | YAML syntax is valid | Run `yamllint` or `python3 -c "import yaml; yaml.safe_load(open('...'))"` |
| CP4 | After cache setup | Cache paths match the language ecosystem | Verify against the cache table in Section 3.4 |
| CP5 | Before output | Every `uses:` is SHA-pinned | Search for `@main`, `@master`, or bare `@v\d` -- must return empty |

---

## 7. Pitfalls and Edge Cases

1. **`pull_request_target` is a security minefield.** It runs in the target repository's context with full secrets access. Never suggest it unless the user explicitly understands the risk. Always prefer `pull_request`.

2. **`cancel-in-progress` must be `false` for releases.** Release workflows should never be interrupted. If a release job is accidentally canceled, it can leave the release in a broken state.

3. **GitHub-hosted runner costs vary.** macOS runners consume 10x the billing minutes of Linux; Windows runners consume 2x. Always default to `ubuntu-latest`.

4. **`GITHUB_TOKEN` has write permissions by default.** Unless you declare `permissions` explicitly, the auto-generated token can write to the repository -- a security risk.

5. **Use `fail-fast: false` in matrix builds.** In multi-language or multi-platform matrix strategies, set `fail-fast: false` so one platform failure does not cancel all the others.

6. **Workflow files must be placed in `.github/workflows/`.** The file extension must be `.yml` or `.yaml`.

7. **Manual cache keys must include a lockfile hash.** When using `actions/cache` directly, the `key` field must contain `hashFiles('path/to/lockfile')`, otherwise the cache will never bust.

8. **Never hardcode credentials.** Always reference secrets via `${{ secrets.NAME }}` and configure them in GitHub Settings -> Secrets and variables -> Actions.

---

## 8. FAQ

### Q1: The project has no lockfile -- how should I configure caching?

**A:** Skip cache configuration. The built-in `cache` parameter on `setup-node`, `setup-python`, etc. relies on a lockfile to generate the cache key. Without one, caching will misbehave. Inform the user: "Commit your lockfile (`package-lock.json`, `pnpm-lock.yaml`, etc.) to the repository to enable dependency caching."

### Q2: How do I write a workflow that runs on a self-hosted runner?

**A:** Replace `runs-on: ubuntu-latest` with your runner's label (e.g., `self-hosted`, `linux-arm64`). Also note:
- Self-hosted runners require you to maintain the environment (tools, dependencies, etc.)
- Use multiple labels for precision: `runs-on: [self-hosted, linux, x64]`

### Q3: What happens when one job fails in a multi-job workflow?

**A:**
- By default, all downstream jobs that depend on the failed job via `needs` are skipped.
- If you need a job to run regardless (e.g., a cleanup step), add `if: always()` at the job level -- but each step inside it should still check its own preconditions.

---

## 9. Non-Negotiable Rules

**The following are absolutely forbidden:**

| # | Rule | Why |
|---|------|-----|
| 1 | Hardcoding tokens, API keys, or passwords in YAML | Never output real credentials -- even as placeholders |
| 2 | Recommending `pull_request_target` unless the user fully understands the risk | Known security vulnerability hotspot |
| 3 | Deleting or overwriting an existing workflow without user confirmation | Show a diff first; wait for explicit approval |
| 4 | Referencing `@main` or `@master` for any action | Always pin to a full SHA-1 commit hash |
| 5 | Setting `permissions: write-all` | Always declare the narrowest permissions needed |

---

## 10. Validation Checklist

### Functional Checklist

```
[ ] 1. Workflow includes a permissions block (workflow or job level)
[ ] 2. Workflow includes a concurrency block
[ ] 3. Every uses: reference includes a # vX.Y.Z version comment
[ ] 4. Cache configuration matches the project's language and package manager
[ ] 5. The on: trigger events match the user's stated scenario
[ ] 6. No hardcoded API keys, tokens, or passwords anywhere in the file
[ ] 7. runs-on defaults to ubuntu-latest (unless the user explicitly requested otherwise)
```

### Runtime Validation

```bash
# 1. YAML syntax check
yamllint .github/workflows/*.yml

# 2. Alternative: validate with Python
python3 -c "
import yaml, pathlib
for f in pathlib.Path('.github/workflows').glob('*.yml'):
    try:
        yaml.safe_load(f.read_text())
        print(f'{f}: OK')
    except Exception as e:
        print(f'{f}: FAIL -- {e}')
        exit(1)
"

# 3. Scan for accidentally committed secrets
rg -n 'sk-[a-zA-Z0-9]{20,}' .github/workflows/       # OpenAI key
rg -n 'ghp_[a-zA-Z0-9]{36}' .github/workflows/       # GitHub PAT
rg -n 'AKIA[a-zA-Z0-9]{16}' .github/workflows/       # AWS access key

# 4. Check for unpinned actions (should return empty)
rg -n '@main|@master' .github/workflows/
rg -n 'uses:\s+\S+@v\d+$' .github/workflows/

# 5. Verify mandatory configs are present
echo "=== permissions ===" && rg -c 'permissions:' .github/workflows/
echo "=== concurrency ===" && rg -c 'concurrency:' .github/workflows/
```
