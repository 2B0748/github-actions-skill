# GitHub Actions Automation Workflow Skill

> Platform-agnostic AI skill — copy directly into any AI coding assistant.
> Compatible with: Cursor Rules · GitHub Copilot · Windsurf · Aider · Cline · CodeBuddy · Custom System Prompt

## 1. Description (Trigger Conditions)

**Activate this skill when ANY of the following conditions are met:**

- User requests "create/generate/write a GitHub Actions workflow"
- User requests "review/optimize/fix CI/CD config" or "audit workflow security"
- User requests automation: auto-test, auto-release, auto-label, auto-dependency-update
- User mentions specific Action names: `actions/checkout`, `actions/cache`, `actions/upload-artifact`
- User mentions workflow keywords: `on:`, `jobs:`, `runs-on`, `permissions`, `concurrency`
- User requests to "automate" or "put into CI" a manual process

**Do NOT trigger if:** user discusses other CI platforms (Jenkins/GitLab CI/CircleCI), local scripts, or Docker builds without Actions context.

---

## 2. Core Workflow Decision Tree

Use this decision tree to determine the skeleton when generating a Workflow:

```
User Request
│
├─ Contains "PR" / "push" / "commit" / "lint" / "test"
│  └─ → Generate CI Workflow (Template A)
│
├─ Contains "Tag" / "Release" / "publish" / "package"
│  └─ → Generate Release Workflow (Template B)
│
├─ Contains "cron" / "scheduled" / "periodic" / "dependency update" / "Dependabot"
│  └─ → Generate Scheduled Workflow (Template C)
│
├─ Contains "security" / "scan" / "vulnerability" / "secret" / "CodeQL"
│  └─ → Generate Security Workflow (Template D)
│
└─ Contains "Issue" / "PR auto" / "label" / "auto-reply" / "triage"
   └─ → Generate Event-driven Automation Workflow (Template E)
```

---

## 3. Generation Directives (Imperative)

### 3.1 Skeleton Generation Phase

**Execute in order — do NOT skip any step:**

1. **Confirm runtime environment.** Ask for the project language/framework (Node.js/Python/Go/Rust/Docker). If not provided, infer from repo files (check for `package.json`, `go.mod`, `Cargo.toml`, etc.).
2. **Confirm trigger events.** Determine `on:` configuration based on the decision tree.
3. **Confirm Runner.** Default to `ubuntu-latest` unless the user explicitly requests another platform.
4. **Generate YAML.** Select the skeleton from the appropriate template below and fill in the user's specific details.

### 3.2 Mandatory Configuration

**Every generated Workflow MUST include these three configurations — no exceptions:**

| Config | Placement | Requirement |
|--------|-----------|-------------|
| `permissions` | Job level | Explicitly declared; least-privilege principle |
| `concurrency` | Job level | Auto-cancel stale runs for same branch/PR |
| Action pinning | Step level | Use SHA-1 hashes; forbid `@main` / `@master` |

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
      # MUST include cache configuration
```

### 3.4 Cache Strategy Quick Reference

| Language/Ecosystem | setup Action | Cache Config |
|--------------------|-------------|--------------|
| Node.js (npm) | `actions/setup-node@v4` | `cache: 'npm'` |
| Node.js (pnpm) | `actions/setup-node@v4` | `cache: 'pnpm'` |
| Python | `actions/setup-python@v5` | `cache: 'pip'` |
| Go | `actions/setup-go@v5` | `cache: true` |
| Rust | No official setup | Manual `actions/cache@v4` for `target/` |
| Docker | None | Use `docker/build-push-action` `cache-from` / `cache-to` |

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
      cancel-in-progress: false  # Releases must NEVER be cancelled

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
    - cron: '31 7 * * 3'  # Weekly on Wednesday

permissions:
  security-events: write
  contents: read

jobs:
  analyze:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        language: ['javascript', 'python']  # Adjust per project
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4
      - uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}
      - uses: github/codeql-action/analyze@v3
```

---

## 4. Before/After Comparisons

### Example 1: Least-Privilege Permissions

```yaml
# ❌ Before — no permissions declaration (inherits default write, security risk)
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

# ✅ After — explicit least privilege
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
# ❌ Before — no concurrency, multiple pushes on the same PR pile up
name: CI
on: [pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps: [ ... ]

# ✅ After — new commits auto-cancel stale runs
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
# ❌ Before — floating versions, supply-chain attack risk
- uses: actions/checkout@v4
- uses: docker/setup-buildx-action@master

# ✅ After — pinned to exact commit SHA
- uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
- uses: docker/setup-buildx-action@c47758b77c9736f4b2ef4073d4d51994fabfe349 # v3.8.0
```

---

## 5. Few-Shot Examples

### Example 1: Most Common — "Add CI to my Node.js project"

**User Input:**
> Write me a GitHub Actions workflow that runs eslint and jest on PRs

**AI Output (per this Skill):**
1. Detect: `package.json` → Node.js project
2. Select template: Template A (CI)
3. Generate with: `permissions`, `concurrency`, `cache: 'npm'`, `npm ci` → lint → test pipeline

---

### Example 2: Variation — "Weekly dependency update check"

**User Input:**
> I want GitHub to auto-check npm dependency updates every Monday

**AI Output (per this Skill):**
1. Select template: Template C (Scheduled)
2. Generate `.github/workflows/dependency-update.yml`
3. Use `cron: '0 9 * * 1'` + `actions/checkout` + `npm outdated` + auto-create PR if updates found
4. Permissions limited to `contents: write` and `pull-requests: write`

---

### Example 3: Edge Case — "My CI takes 15 minutes"

**User Input:**
> My CI takes 15 minutes per run, how do I speed it up?

**AI Output (per this Skill):**
1. Audit the existing workflow file
2. Check cache config → if missing, insert language-appropriate cache
3. Check job parallelization → split lint and test into independent jobs, remove unnecessary serial `needs` chains
4. Provide estimated time savings

---

### Example 4: Security Audit — "Is my workflow secure?"

**User Input:**
> Audit `.github/workflows/ci.yml` for security issues

**AI Output (per this Skill):**

| Check Item | Status | Detail |
|------------|--------|--------|
| `permissions` explicitly declared | ❌ | Not set — inherits write permission |
| Action version pinning | ❌ | `actions/checkout@v4` — not SHA-pinned |
| `concurrency` configured | ✅ | Present |
| Secret references | ✅ | Uses `${{ secrets.X }}`, no plaintext |
| `pull_request_target` risk | ✅ | Not used (avoids high-risk event) |

---

### Example 5: Multi-language — "Rust + Node.js monorepo CI"

**User Input:**
> My project has a Rust backend and React frontend, how do I write one CI?

**AI Output (per this Skill):**
1. Detect: `Cargo.toml` + `package.json` at repo root
2. Generate matrix strategy or two parallel jobs
3. Rust: manually configure `actions/cache@v4` for `~/.cargo` and `target/`
4. Node.js: use `setup-node` with built-in `cache: 'npm'`
5. Both jobs run in parallel, each with its own `working-directory`

---

## 6. Checkpoints

| CP # | Stage | What to Verify | Command / Expected Output |
|------|-------|---------------|--------------------------|
| CP1 | After skeleton | `on:` events match user request | Cross-check against decision tree |
| CP2 | After YAML gen | Three mandatory configs present | Search for `permissions:`, `concurrency:`, `# v` (version comments) |
| CP3 | After YAML gen | YAML syntax is valid | `yamllint .github/workflows/` or Python `yaml.safe_load()` |
| CP4 | After cache setup | Cache paths match language ecosystem | Verify against §3.4 Cache Quick Reference |
| CP5 | Action pinning | All `uses:` are SHA-pinned | Search for `@main`, `@master`, `@v[0-9]$` — must return empty |

---

## 7. ⚠️ Pitfalls & Edge Cases

1. **`pull_request_target` is a security minefield.** This event runs in the target repo's context with secrets access. Never suggest it unless the user explicitly understands the risk. Always prefer `pull_request`.

2. **`cancel-in-progress: false` for Releases.** Release job concurrency MUST set `cancel-in-progress: false`, otherwise a tag push can be accidentally cancelled.

3. **GitHub free Runner resources are limited.** macOS Runners consume 10× billing minutes, Windows 2×. Always default to `ubuntu-latest`.

4. **`GITHUB_TOKEN` defaults to `write`.** Without explicit `permissions`, the token has repo write access — a security risk.

5. **Matrix strategy `fail-fast: false`.** For multi-language / multi-platform matrix tests, set `fail-fast: false` to prevent one platform failure from skipping others.

6. **Workflow file path is fixed.** All workflow files must be in `.github/workflows/` with `.yml` or `.yaml` extension.

7. **Cache key uniqueness.** When manually configuring `actions/cache`, the `key` MUST include `hashFiles('path/to/lockfile')`, otherwise the cache will never invalidate.

8. **Encrypted secrets only.** Never hardcode tokens or passwords in workflows. Use `${{ secrets.XXX }}` and configure in GitHub Settings → Secrets.

---

## 8. FAQ

### Q1: The repo has no lockfile (e.g., no `package-lock.json`) — how to configure caching?

**A:** Do NOT configure caching. `actions/setup-node`'s `cache` parameter depends on the lockfile to generate the cache key. If there's no lockfile, skip cache configuration and inform the user: "Consider committing your lockfile to enable dependency caching."

### Q2: How to write a workflow for self-hosted Runners?

**A:** Change `runs-on` to the Runner's label (e.g., `self-hosted`, `linux-arm64`). Also note:
- Self-hosted Runners require manual environment maintenance
- Recommend using `runs-on: [self-hosted, linux, x64]` for precise label matching

### Q3: What if one job fails in a multi-job workflow?

**A:**
- Default: remaining jobs with `needs` on the failed job are all skipped
- For "always-run" jobs (e.g., cleanup), add `if: always()` but check preconditions within each step

---

## 9. 🔴 Red Lines (Zero Tolerance)

**The following behaviors are absolutely forbidden:**

| # | Forbidden Behavior | Rationale |
|---|-------------------|-----------|
| 🔴1 | Hardcoding tokens, API keys, or passwords in YAML | Output must never contain example tokens as real config |
| 🔴2 | Recommending `pull_request_target` unless user fully understands risk | Known security vulnerability hotspot |
| 🔴3 | Deleting/overwriting existing workflows without user confirmation | Show diff first, wait for explicit approval |
| 🔴4 | Referencing `@main` / `@master` action versions | Always use SHA-1 hashes for pinning |
| 🔴5 | Setting `permissions: write-all` | Always declare specific permissions; forbid wildcard write |

---

## 10. Validation Checklist

### Functional Validation

```
□ 1. Generated YAML includes permissions (job level)
□ 2. Generated YAML includes concurrency config
□ 3. All uses: references include # vX.Y.Z version comments
□ 4. Cache config matches project language/package manager
□ 5. on: trigger events match user request scenario
□ 6. No hardcoded API keys, tokens, or passwords
□ 7. runs-on defaults to ubuntu-latest (unless specifially requested otherwise)
```

### Runtime Validation (copy-paste ready)

```bash
# 1. YAML syntax check
yamllint .github/workflows/*.yml

# 2. Fallback with Python
python3 -c "
import yaml, pathlib
for f in pathlib.Path('.github/workflows').glob('*.yml'):
    try:
        yaml.safe_load(f.read_text())
        print(f'{f}: OK')
    except Exception as e:
        print(f'{f}: FAIL — {e}')
        exit(1)
"

# 3. Secret leak scan
rg -n 'sk-[a-zA-Z0-9]{20,}' .github/workflows/       # OpenAI Key
rg -n 'ghp_[a-zA-Z0-9]{36}' .github/workflows/       # GitHub PAT
rg -n 'AKIA[a-zA-Z0-9]{16}' .github/workflows/       # AWS Access Key

# 4. Floating version check (should return empty)
rg -n '@main|@master' .github/workflows/

# 5. Mandatory config check
echo "=== permissions ===" && rg -c 'permissions:' .github/workflows/
echo "=== concurrency ===" && rg -c 'concurrency:' .github/workflows/
```
