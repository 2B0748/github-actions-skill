# GitHub Actions Workflow Skill

> Platform-agnostic AI skill — drop into any AI coding assistant.
> Compatible with: Cursor Rules · GitHub Copilot · Windsurf · Aider · Cline · CodeBuddy · Custom System Prompt

## 1. Core Identity (Trigger Conditions)

**Activate this Skill immediately when the user's request meets any of the following conditions:**

- Request to "create/generate/write a GitHub Actions workflow"
- Request to "review/optimize/fix CI/CD configuration" or "check workflow security"
- Request to add automation capabilities: auto-test, auto-release, auto-label, auto-dependency-update
- Mentioning specific Action names: `actions/checkout`, `actions/cache`, `actions/upload-artifact`
- Mentioning workflow keywords: `on:`, `jobs:`, `runs-on`, `permissions`, `concurrency`
- Request to "automate" a manual process or "put it in CI"

**Non-triggering situations:** User discusses other CI platforms (Jenkins/GitLab CI/CircleCI), pure local scripts, Docker image builds (without Actions context).

---

## 2. Core Workflow Decision Tree

When generating a workflow, follow this decision tree to determine the skeleton:

```
User Request
│
├─ Contains "PR" / "push" / "commit" / "lint" / "test"
│  └─ → Generate CI Workflow (Template A)
│
├─ Contains "Tag" / "Release" / "publish" / "package"
│  └─ → Generate Release Workflow (Template B)
│
├─ Contains "scheduled" / "cron" / "periodic" / "dependency update" / "Dependabot"
│  └─ → Generate Scheduled Workflow (Template C)
│
├─ Contains "security" / "scan" / "vulnerability" / "secret" / "CodeQL"
│  └─ → Generate Security Workflow (Template D)
│
└─ Contains "Issue" / "PR auto" / "label" / "reply" / "triage"
   └─ → Generate Event-driven Automation Workflow (Template E)
```

---

## 3. Generation Instructions (Imperative)

### 3.1 Skeleton Generation Phase

**Execute in the following order; do not skip any step:**

1. **Confirm the runtime environment.** Ask about the project language/framework (Node.js/Python/Go/Rust/Docker). If not provided, infer from repository files (detect `package.json`, `go.mod`, `Cargo.toml`, etc.).
2. **Confirm trigger events.** Determine the `on:` configuration based on the decision tree.
3. **Confirm the Runner.** Default to `ubuntu-latest` unless the user explicitly requests another platform.
4. **Generate YAML.** Select the skeleton from the corresponding template below and fill in user-specific details.

### 3.2 Required Configuration

**Every generated Workflow MUST include the following three configurations — none can be omitted:**

| Config | Scope | Rule |
|--------|------|------|
| `permissions` | Job-level | Explicitly declared, principle of least privilege |
| `concurrency` | Job-level | Auto-cancel old runs for same branch/PR |
| Action version pinning | Step-level | Use SHA-1 hash, forbid `@main` / `@master` |

### 3.3 Template A: CI Workflow

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
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4
      # Insert language-specific setup and lint steps

  test:
    needs: lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4
      # Insert language-specific setup and test steps
      # Must include cache configuration
```

### 3.4 Cache Strategy Cheat Sheet

| Language/Ecosystem | Setup Action | Cache Config |
|-----------|-------------|---------|
| Node.js (npm) | `actions/setup-node@v4` | `cache: 'npm'` |
| Node.js (pnpm) | `actions/setup-node@v4` | `cache: 'pnpm'` |
| Python | `actions/setup-python@v5` | `cache: 'pip'` |
| Go | `actions/setup-go@v5` | `cache: true` |
| Rust | No official setup | Manually cache `target/` with `actions/cache@v4` |
| Docker | None | Use `cache-from` / `cache-to` from `docker/build-push-action` |

### 3.5 Template B: Release Workflow

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
      cancel-in-progress: false  # Release must not be cancelled

    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4
      # Build artifacts → Create Release → Upload Artifact
```

### 3.6 Template D: Security / CodeQL

```yaml
name: CodeQL

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '31 7 * * 3'  # Runs every Wednesday

permissions:
  security-events: write
  contents: read

jobs:
  analyze:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        language: ['javascript', 'python']  # Adjust based on project
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4
      - uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}
      - uses: github/codeql-action/analyze@v3
```

---

## 4. Before/After Comparison Examples

### Example 1: Permission Hardening

```yaml
# ❌ Before — No permission declaration (inherits default write permission, security risk)
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

# ✅ After — Explicit least privilege
permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4
```

### Example 2: Concurrency Control

```yaml
# ❌ Before — No concurrency control, multiple pushes to the same PR will pile up
name: CI
on: [pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps: [ ... ]

# ✅ After — New commits automatically cancel old runs
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

### Example 3: Action Version Pinning

```yaml
# ❌ Before — Floating version, supply-chain attack risk
- uses: actions/checkout@v4
- uses: docker/setup-buildx-action@master

# ✅ After — Precisely pinned to commit SHA
- uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
- uses: docker/setup-buildx-action@c47758b77c9736f4b2ef4073d4d51994fabfe349 # v3.8.0
```

---

## 5. Few-Shot Examples

### Example 1: Most Common Scenario — "Add CI to my Node.js project"

**User Input:**
> Write a GitHub Actions workflow that runs eslint and jest on PRs

**AI Output (following this Skill):**
1. Detect project: confirm `package.json` exists, infer Node.js project
2. Determine template: Template A (CI)
3. Generate with: permissions, concurrency, cache `cache: 'npm'`, npm ci → lint → test flow

---

### Example 2: Slight Variation — "Scheduled dependency updates"

**User Input:**
> I want GitHub to automatically check for npm dependency updates every Monday

**AI Output (following this Skill):**
1. Determine template: Template C (Scheduled)
2. Generate `.github/workflows/dependency-update.yml`
3. Use `cron: '0 9 * * 1'` + `actions/checkout` + `npm outdated` + auto-create PR if updates are found
4. Permissions restricted to `contents: write` and `pull-requests: write`

---

### Example 3: Edge Case — "Workflow runs too slow"

**User Input:**
> My CI takes 15 minutes per run, how can I optimize it?

**AI Output (following this Skill):**
1. Review existing workflow files
2. Check for cache configuration → if none, insert `cache: 'npm'` or language-specific cache
3. Check if jobs can be split for parallel execution → split lint and test into independent jobs (remove unnecessary serial dependencies via `needs`)
4. Provide estimated time savings assessment

---

### Example 4: Security Review — "Check my workflow for security issues"

**User Input:**
> Help me review the security of `.github/workflows/ci.yml`

**AI Output (following this Skill):**

| Check Item | Status | Description |
|--------|------|------|
| `permissions` explicitly declared | ❌ | Not set, defaults to write permission |
| Action version pinning | ❌ | `actions/checkout@v4` SHA not pinned |
| `concurrency` configured | ✅ | Configured |
| Secret reference method | ✅ | Uses `${{ secrets.X }}`, no plaintext |
| `pull_request_target` risk | ✅ | Not used (high-risk event avoided) |

---

### Example 5: Multi-language Project — "CI for Rust + Node.js"

**User Input:**
> My project has a Rust backend and a React frontend, how do I write a CI?

**AI Output (following this Skill):**
1. Detect project root: identify `Cargo.toml` + `package.json`
2. Generate matrix strategy (`strategy.matrix`) or split into two jobs
3. Rust part: manually configure `actions/cache@v4` to cache `~/.cargo` and `target/`
4. Node.js part: `setup-node` natively supports `cache: 'npm'`
5. Two jobs run in parallel, using different `working-directory`

---

## 6. Checkpoints

| CP # | Location | Verification | Verification Command / Expected Output |
|----------|------|----------|-------------------|
| CP1 | After skeleton generation | Whether `on:` events correctly match user requirements | Cross-reference the decision tree to confirm trigger events match user request |
| CP2 | After YAML generation | Whether the three required configurations are included | Search for `permissions:`, `concurrency:`, `# v` (version comment) in the file |
| CP3 | After YAML generation | Whether YAML syntax is valid | `yamllint .github/workflows/` or Python `yaml.safe_load()` |
| CP4 | After cache configuration | Whether cache paths match the language ecosystem | Cross-reference each item against section 3.4 Cache Strategy Cheat Sheet |
| CP5 | Action version pinning | Whether all `uses:` have pinned SHA | Search for `@main`, `@master`, `@v[0-9]$` — must return empty |

---

## 7. Important Notes (Edge Cases and Pitfalls)

1. **`pull_request_target` event is a security minefield.** This event runs in the target repository's context and can read secrets. Never suggest using it unless the user explicitly requests it and understands the risks. Prefer `pull_request`.

2. **Set `cancel-in-progress` to `false` for Release.** In the Release job's concurrency config, `cancel-in-progress` must be `false`, otherwise the tag push could be unexpectedly cancelled.

3. **GitHub free Runner resources are limited.** macOS Runner consumes 10x billing minutes, Windows consumes 2x. Prefer `ubuntu-latest`.

4. **`GITHUB_TOKEN` default permission is `write`.** If `permissions` is not explicitly set, the token has write access to the repository, posing a security risk.

5. **Matrix strategy `fail-fast: false`.** In multi-language/multi-platform matrix tests, set `fail-fast: false` to prevent one platform failure from skipping others.

6. **Workflow file path is fixed.** All workflow files must be placed in the `.github/workflows/` directory, and filenames must end with `.yml` or `.yaml`.

7. **Cache key uniqueness.** When manually configuring `actions/cache`, the `key` must include `hashFiles('lockfile-path')`, otherwise the cache will never be invalidated.

8. **Encrypted variables and secrets.** Never hardcode tokens/passwords in the workflow. Use `${{ secrets.XXX }}` and configure them in GitHub Settings → Secrets.

---

## 8. Practical FAQ

### Q1: The user's repository has no lockfile (e.g. `package-lock.json`), how to configure caching?

**Answer:** Do not configure caching. The `cache` parameter of `actions/setup-node` relies on the lockfile to generate the cache key. If there is no lockfile, skip cache configuration and remind the user: "It is recommended to commit the lockfile to the repository to enable dependency caching."

### Q2: The user wants the workflow to run on a self-hosted Runner, how to write it?

**Answer:** Change `runs-on` to the Runner's label name (e.g. `self-hosted`, `linux-arm64`). Also remind in the output:
- Self-hosted Runners require self-maintenance of the environment; ensure dependencies are installed
- It is recommended to use `runs-on: [self-hosted, linux, x64]` multi-label for precise matching

### Q3: There are multiple jobs in the workflow, how to handle partial job failures?

**Answer:**
- Default behavior: if a job fails, all subsequent jobs that depend on it via `needs` are skipped
- For jobs that must "always run" (e.g. cleanup resources), add `if: always()` but steps inside should check their own prerequisites

---

## 9. Security Red Lines

**The following behaviors are absolutely forbidden; violating any one of them means the Skill has failed:**

| Red Line # | Forbidden Behavior | Description |
|--------|---------|------|
| 1 | Hardcode Token / API Key / password in YAML | Example tokens must never appear as real configuration in the output |
| 2 | Suggest `pull_request_target` unless the user fully understands the risks | This event is a known hotbed for security vulnerabilities |
| 3 | Delete or overwrite an existing workflow without requesting user confirmation | Deletion must first show a diff and wait for user confirmation before executing |
| 4 | Reference `@main` / `@master` versions of Actions | Always pin versions using SHA-1 hash |
| 5 | Set `permissions: write-all` | Always declare specific permissions; wildcard write permission is forbidden |

---

## 10. Verification Checklist

### Functional Verification

```
□ 1. Generated YAML includes permissions (job-level)
□ 2. Generated YAML includes concurrency configuration
□ 3. All uses: references include # vX.Y.Z version comments
□ 4. Cache configuration matches the project language/package manager
□ 5. on: trigger events are consistent with the user's request scenario
□ 6. No hardcoded secrets, tokens, or passwords
□ 7. runs-on defaults to ubuntu-latest (unless there are special requirements)
```

### Runtime Verification (Copy-Paste Ready)

```bash
# 1. YAML syntax validation
yamllint .github/workflows/*.yml

# 2. If yamllint is not installed, use Python instead
python3 -c "
import yaml, sys, pathlib
for f in pathlib.Path('.github/workflows').glob('*.yml'):
    try:
        yaml.safe_load(f.read_text())
        print(f'{f}: OK')
    except Exception as e:
        print(f'{f}: FAIL — {e}')
        sys.exit(1)
"

# 3. Security keyword scan (check for leaked secrets)
rg -n 'sk-[a-zA-Z0-9]{20,}' .github/workflows/       # OpenAI Key
rg -n 'ghp_[a-zA-Z0-9]{36}' .github/workflows/       # GitHub PAT
rg -n 'AKIA[a-zA-Z0-9]{16}' .github/workflows/       # AWS Access Key

# 4. Check for floating version references
rg -n '@main|@master' .github/workflows/              # Should return empty
rg -n 'uses:\s+\S+@v\d+$' .github/workflows/          # Should return empty (must pin SHA)

# 5. Check for missing required configurations
echo "=== Check permissions ===" && rg -c 'permissions:' .github/workflows/
echo "=== Check concurrency ===" && rg -c 'concurrency:' .github/workflows/
```
