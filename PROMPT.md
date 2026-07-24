# GitHub Actions 自动化工作流 Skill（通用版）/ GitHub Actions Automation Workflow Skill (Universal)

> 本文件为平台无关的通用版本，可直接复制到任意 AI 编程助手中使用。
> This file is a platform-agnostic universal version that can be copied directly into any AI coding assistant.
> 适用平台：Cursor Rules · GitHub Copilot · Windsurf · Aider · Cline · CodeBuddy · 自定义 System Prompt
> Compatible with: Cursor Rules · GitHub Copilot · Windsurf · Aider · Cline · CodeBuddy · Custom System Prompt

## 1. 灵魂描述（触发条件） / Core Identity (Trigger Conditions)

**当用户的请求满足以下任一条件时，立即激活本 Skill：**
**Activate this Skill immediately when the user's request meets any of the following conditions:**

- 请求"创建/生成/写一个 GitHub Actions workflow"
- Request to "create/generate/write a GitHub Actions workflow"
- 请求"审查/优化/修复 CI/CD 配置"或"检查 workflow 安全性"
- Request to "review/optimize/fix CI/CD configuration" or "check workflow security"
- 请求添加自动化能力：自动测试、自动发布、自动标签、自动依赖更新
- Request to add automation capabilities: auto-test, auto-release, auto-label, auto-dependency-update
- 提到具体 Action 名称：`actions/checkout`、`actions/cache`、`actions/upload-artifact`
- Mentioning specific Action names: `actions/checkout`, `actions/cache`, `actions/upload-artifact`
- 提到 workflow 关键字：`on:`、`jobs:`、`runs-on`、`permissions`、`concurrency`
- Mentioning workflow keywords: `on:`, `jobs:`, `runs-on`, `permissions`, `concurrency`
- 请求将某个手动流程"自动化"或"放到 CI 里"
- Request to "automate" a manual process or "put it in CI"

**不触发的情况：** 用户讨论其他 CI 平台（Jenkins/GitLab CI/CircleCI）、纯本地脚本、Docker 镜像构建（无 Actions 上下文）。
**Non-triggering situations:** User discusses other CI platforms (Jenkins/GitLab CI/CircleCI), pure local scripts, Docker image builds (without Actions context).

---

## 2. 核心工作流决策树 / Core Workflow Decision Tree

当用户请求生成 Workflow 时，按以下决策树确定骨架：
When generating a workflow, follow this decision tree to determine the skeleton:

```
用户请求 / User Request
│
├─ 包含 "PR" / "push" / "提交" / "lint" / "测试"
│  Contains "PR" / "push" / "commit" / "lint" / "test"
│  └─ → 生成 CI Workflow（模板 A）/ Generate CI Workflow (Template A)
│
├─ 包含 "Tag" / "Release" / "发布" / "打包"
│  Contains "Tag" / "Release" / "publish" / "package"
│  └─ → 生成 Release Workflow（模板 B）/ Generate Release Workflow (Template B)
│
├─ 包含 "定时" / "cron" / "定期" / "依赖更新" / "Dependabot"
│  Contains "scheduled" / "cron" / "periodic" / "dependency update" / "Dependabot"
│  └─ → 生成 Scheduled Workflow（模板 C）/ Generate Scheduled Workflow (Template C)
│
├─ 包含 "安全" / "扫描" / "漏洞" / "密钥" / "CodeQL"
│  Contains "security" / "scan" / "vulnerability" / "secret" / "CodeQL"
│  └─ → 生成 Security Workflow（模板 D）/ Generate Security Workflow (Template D)
│
└─ 包含 "Issue" / "PR 自动" / "标签" / "回复" / "分类"
   Contains "Issue" / "PR auto" / "label" / "reply" / "triage"
   └─ → 生成 Event-driven Automation Workflow（模板 E）/ Generate Event-driven Automation Workflow (Template E)
```

---

## 3. 生成指令（祈使句） / Generation Instructions (Imperative)

### 3.1 骨架生成阶段 / Skeleton Generation Phase

**按以下顺序执行，不要跳过任何步骤：**
**Execute in the following order; do not skip any step:**

1. **确认运行环境。** 询问用户项目语言/框架（Node.js/Python/Go/Rust/Docker），若用户未提供，从仓库文件推断（检测 `package.json`、`go.mod`、`Cargo.toml` 等）。
   **Confirm the runtime environment.** Ask about the project language/framework (Node.js/Python/Go/Rust/Docker). If not provided, infer from repository files (detect `package.json`, `go.mod`, `Cargo.toml`, etc.).
2. **确认触发事件。** 根据决策树确定 `on:` 配置。
   **Confirm trigger events.** Determine the `on:` configuration based on the decision tree.
3. **确认 Runner。** 默认使用 `ubuntu-latest`，除非用户明确要求其他平台。
   **Confirm the Runner.** Default to `ubuntu-latest` unless the user explicitly requests another platform.
4. **生成 YAML。** 从下方对应模板中选择骨架，填充用户具体信息。
   **Generate YAML.** Select the skeleton from the corresponding template below and fill in user-specific details.

### 3.2 必含配置 / Required Configuration

**每个生成的 Workflow 必须包含以下三项配置，缺一不可：**
**Every generated Workflow MUST include the following three configurations — none can be omitted:**

| 配置项 / Config | 位置 / Scope | 要求 / Rule |
|--------|------|------|
| `permissions` | Job 级别 / Job-level | 显式声明，最小权限原则 / Explicitly declared, principle of least privilege |
| `concurrency` | Job 级别 / Job-level | 同分支/PR 自动取消旧运行 / Auto-cancel old runs for same branch/PR |
| Action 版本锁定 / Action version pinning | Step 级别 / Step-level | 使用 SHA-1 哈希，禁止 `@main` / `@master` / Use SHA-1 hash, forbid `@main` / `@master` |

### 3.3 模板 A：CI Workflow / Template A: CI Workflow

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
      # 根据语言插入对应的 setup 和 lint 步骤
      # Insert language-specific setup and lint steps

  test:
    needs: lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4
      # 根据语言插入对应的 setup 和 test 步骤
      # Insert language-specific setup and test steps
      # 必须包含缓存配置
      # Must include cache configuration
```

### 3.4 缓存策略速查表 / Cache Strategy Cheat Sheet

| 语言/生态 / Language/Ecosystem | setup Action | 缓存配置 / Cache Config |
|-----------|-------------|---------|
| Node.js (npm) | `actions/setup-node@v4` | `cache: 'npm'` |
| Node.js (pnpm) | `actions/setup-node@v4` | `cache: 'pnpm'` |
| Python | `actions/setup-python@v5` | `cache: 'pip'` |
| Go | `actions/setup-go@v5` | `cache: true` |
| Rust | 无官方 setup / No official setup | 手动 `actions/cache@v4` 缓存 `target/` / Manually cache `target/` with `actions/cache@v4` |
| Docker | 无 / None | 使用 `docker/build-push-action` 的 `cache-from` / `cache-to` / Use `cache-from` / `cache-to` from `docker/build-push-action` |

### 3.5 模板 B：Release Workflow / Template B: Release Workflow

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
      cancel-in-progress: false  # Release 不允许取消 / Release must not be cancelled

    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4
      # 构建产物 → 创建 Release → 上传 Artifact
      # Build artifacts → Create Release → Upload Artifact
```

### 3.6 模板 D：Security / CodeQL / Template D: Security / CodeQL

```yaml
name: CodeQL

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '31 7 * * 3'  # 每周三运行 / Runs every Wednesday

permissions:
  security-events: write
  contents: read

jobs:
  analyze:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        language: ['javascript', 'python']  # 根据项目调整 / Adjust based on project
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4
      - uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}
      - uses: github/codeql-action/analyze@v3
```

---

## 4. Before/After 对比示例 / Before/After Comparison Examples

### 示例 1：权限收紧 / Example 1: Permission Hardening

```yaml
# ❌ Before — 无权限声明（继承默认写权限，安全风险）
# ❌ Before — No permission declaration (inherits default write permission, security risk)
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

# ✅ After — 显式最小权限
# ✅ After — Explicit least privilege
permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4
```

### 示例 2：并发控制 / Example 2: Concurrency Control

```yaml
# ❌ Before — 无并发控制，同一 PR 多次推送会堆积运行
# ❌ Before — No concurrency control, multiple pushes to the same PR will pile up
name: CI
on: [pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps: [ ... ]

# ✅ After — 新提交自动取消旧运行
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

### 示例 3：Action 版本锁定 / Example 3: Action Version Pinning

```yaml
# ❌ Before — 浮动版本，供应链攻击风险
# ❌ Before — Floating version, supply-chain attack risk
- uses: actions/checkout@v4
- uses: docker/setup-buildx-action@master

# ✅ After — 精确锁定到 commit SHA
# ✅ After — Precisely pinned to commit SHA
- uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
- uses: docker/setup-buildx-action@c47758b77c9736f4b2ef4073d4d51994fabfe349 # v3.8.0
```

---

## 5. Few-Shot 示例 / Few-Shot Examples

### 示例 1：最常见场景 — "帮我给 Node.js 项目加 CI" / Example 1: Most Common Scenario — "Add CI to my Node.js project"

**用户输入 / User Input：**
> 帮我写一个 GitHub Actions，PR 的时候跑 eslint 和 jest
> Write a GitHub Actions workflow that runs eslint and jest on PRs

**AI 输出（按本 Skill 执行） / AI Output (following this Skill)：**
1. 检测项目：确认有 `package.json`，推断 Node.js 项目
   Detect project: confirm `package.json` exists, infer Node.js project
2. 确定模板：模板 A（CI）
   Determine template: Template A (CI)
3. 生成包含：permissions、concurrency、缓存 `cache: 'npm'`、npm ci → lint → test 流程
   Generate with: permissions, concurrency, cache `cache: 'npm'`, npm ci → lint → test flow

---

### 示例 2：稍有变化 — "定时更新依赖" / Example 2: Slight Variation — "Scheduled dependency updates"

**用户输入 / User Input：**
> 我想让 GitHub 每个星期一自动检查 npm 依赖有没有更新
> I want GitHub to automatically check for npm dependency updates every Monday

**AI 输出（按本 Skill 执行） / AI Output (following this Skill)：**
1. 确定模板：模板 C（Scheduled）
   Determine template: Template C (Scheduled)
2. 生成 `.github/workflows/dependency-update.yml`
   Generate `.github/workflows/dependency-update.yml`
3. 使用 `cron: '0 9 * * 1'` + `actions/checkout` + `npm outdated` + 若发现更新则自动创建 PR
   Use `cron: '0 9 * * 1'` + `actions/checkout` + `npm outdated` + auto-create PR if updates are found
4. 权限仅给 `contents: write` 和 `pull-requests: write`
   Permissions restricted to `contents: write` and `pull-requests: write`

---

### 示例 3：边界情况 — "workflow 跑太慢了" / Example 3: Edge Case — "Workflow runs too slow"

**用户输入 / User Input：**
> 我的 CI 每次跑 15 分钟，怎么优化？
> My CI takes 15 minutes per run, how can I optimize it?

**AI 输出（按本 Skill 执行） / AI Output (following this Skill)：**
1. 审查现有 workflow 文件
   Review existing workflow files
2. 检查是否有缓存配置 → 若无，插入 `cache: 'npm'` / 对应语言缓存
   Check for cache configuration → if none, insert `cache: 'npm'` or language-specific cache
3. 检查是否可拆分 job 并行 → 将 lint 和 test 拆为独立 job（`needs` 去除不必要的串行依赖）
   Check if jobs can be split for parallel execution → split lint and test into independent jobs (remove unnecessary serial dependencies via `needs`)
4. 给出预计节省时间评估
   Provide estimated time savings assessment

---

### 示例 4：安全审查 — "帮我检查 workflow 有没有安全问题" / Example 4: Security Review — "Check my workflow for security issues"

**用户输入 / User Input：**
> 帮我审查 `.github/workflows/ci.yml` 的安全性
> Help me review the security of `.github/workflows/ci.yml`

**AI 输出（按本 Skill 执行） / AI Output (following this Skill)：**

| 检查项 / Check Item | 状态 / Status | 说明 / Description |
|--------|------|------|
| `permissions` 显式声明 / `permissions` explicitly declared | ❌ | 未设置，默认有写权限 / Not set, defaults to write permission |
| Action 版本锁定 / Action version pinning | ❌ | `actions/checkout@v4` 未锁 SHA / `actions/checkout@v4` SHA not pinned |
| `concurrency` 配置 / `concurrency` configured | ✅ | 已配置 / Configured |
| 密钥引用方式 / Secret reference method | ✅ | 使用 `${{ secrets.X }}`，无明文 / Uses `${{ secrets.X }}`, no plaintext |
| `pull_request_target` 风险 / `pull_request_target` risk | ✅ | 未使用（避免高危事件）/ Not used (high-risk event avoided) |

---

### 示例 5：多语言项目 — "Rust + Node.js 的 CI" / Example 5: Multi-language Project — "CI for Rust + Node.js"

**用户输入 / User Input：**
> 我的项目有 Rust 后端和 React 前端，怎么写一个 CI？
> My project has a Rust backend and a React frontend, how do I write a CI?

**AI 输出（按本 Skill 执行） / AI Output (following this Skill)：**
1. 检测项目根目录：识别 `Cargo.toml` + `package.json`
   Detect project root: identify `Cargo.toml` + `package.json`
2. 生成矩阵策略（`strategy.matrix`）或拆分两个 job
   Generate matrix strategy (`strategy.matrix`) or split into two jobs
3. Rust 部分：手动配置 `actions/cache@v4` 缓存 `~/.cargo` 和 `target/`
   Rust part: manually configure `actions/cache@v4` to cache `~/.cargo` and `target/`
4. Node.js 部分：`setup-node` 自带 `cache: 'npm'`
   Node.js part: `setup-node` natively supports `cache: 'npm'`
5. 两个 job 并行运行，使用不同 `working-directory`
   Two jobs run in parallel, using different `working-directory`

---

## 6. 检查点（Checkpoints） / Checkpoints

| 检查点 # / CP # | 位置 / Location | 验证内容 / Verification | 验证命令 / 预期输出 / Verification Command / Expected Output |
|----------|------|----------|-------------------|
| CP1 | 骨架生成后 / After skeleton generation | `on:` 事件是否正确匹配用户需求 / Whether `on:` events correctly match user requirements | 对照决策树，确认触发事件与用户请求匹配 / Cross-reference the decision tree to confirm trigger events match user request |
| CP2 | YAML 生成后 / After YAML generation | 是否包含三项必含配置 / Whether the three required configurations are included | 搜索文件中的 `permissions:`、`concurrency:`、`# v` (版本注释) / Search for `permissions:`, `concurrency:`, `# v` (version comment) in the file |
| CP3 | YAML 生成后 / After YAML generation | YAML 语法是否合法 / Whether YAML syntax is valid | `yamllint .github/workflows/` 或 / or Python `yaml.safe_load()` |
| CP4 | 缓存配置后 / After cache configuration | 缓存路径是否匹配语言生态 / Whether cache paths match the language ecosystem | 对照 3.4 缓存策略速查表逐项核对 / Cross-reference each item against section 3.4 Cache Strategy Cheat Sheet |
| CP5 | Action 版本锁定 / Action version pinning | 所有 `uses:` 是否都锁定了 SHA / Whether all `uses:` have pinned SHA | 搜索 `@main`、`@master`、`@v[0-9]$` 不得出现 / Search for `@main`, `@master`, `@v[0-9]$` — must return empty |

---

## 7. 注意事项（边界与易错点） / Important Notes (Edge Cases and Pitfalls)

1. **`pull_request_target` 事件是安全雷区。** 该事件在目标仓库上下文运行，可以读取 secrets。除非用户明确要求且理解风险，否则永远不要建议使用它。优先使用 `pull_request`。
   **`pull_request_target` event is a security minefield.** This event runs in the target repository's context and can read secrets. Never suggest using it unless the user explicitly requests it and understands the risks. Prefer `pull_request`.

2. **`cancel-in-progress` 对 Release 设 `false`。** Release job 的 concurrency 中 `cancel-in-progress` 必须为 `false`，否则 tag push 可能被意外取消。
   **Set `cancel-in-progress` to `false` for Release.** In the Release job's concurrency config, `cancel-in-progress` must be `false`, otherwise the tag push could be unexpectedly cancelled.

3. **GitHub 免费 Runner 资源有限。** macOS Runner 消耗 10x 计费分钟，Windows 消耗 2x。优先使用 `ubuntu-latest`。
   **GitHub free Runner resources are limited.** macOS Runner consumes 10x billing minutes, Windows consumes 2x. Prefer `ubuntu-latest`.

4. **`GITHUB_TOKEN` 默认权限是 `write`。** 如果不显式设置 `permissions`，token 拥有仓库写权限，存在安全风险。
   **`GITHUB_TOKEN` default permission is `write`.** If `permissions` is not explicitly set, the token has write access to the repository, posing a security risk.

5. **矩阵策略 `fail-fast: false`。** 多语言/多平台矩阵测试中，设 `fail-fast: false`，防止一个平台失败导致其他平台被跳过。
   **Matrix strategy `fail-fast: false`.** In multi-language/multi-platform matrix tests, set `fail-fast: false` to prevent one platform failure from skipping others.

6. **工作流文件路径固定。** 所有 workflow 文件必须放在 `.github/workflows/` 目录下，文件名必须以 `.yml` 或 `.yaml` 结尾。
   **Workflow file path is fixed.** All workflow files must be placed in the `.github/workflows/` directory, and filenames must end with `.yml` or `.yaml`.

7. **缓存 Key 的唯一性。** 手动配置 `actions/cache` 时，`key` 必须包含 `hashFiles('lock文件路径')`，否则缓存永远不会失效。
   **Cache key uniqueness.** When manually configuring `actions/cache`, the `key` must include `hashFiles('lockfile-path')`, otherwise the cache will never be invalidated.

8. **加密变量和密钥。** 绝不在 workflow 中硬编码 Token/密码。使用 `${{ secrets.XXX }}`，并在 GitHub Settings → Secrets 中配置。
   **Encrypted variables and secrets.** Never hardcode tokens/passwords in the workflow. Use `${{ secrets.XXX }}` and configure them in GitHub Settings → Secrets.

---

## 8. 实用 FAQ / Practical FAQ

### Q1：用户的仓库没有 lockfile（如 `package-lock.json`），怎么配缓存？ / Q1: The user's repository has no lockfile (e.g. `package-lock.json`), how to configure caching?

**答：** 不要配置缓存。`actions/setup-node` 的 `cache` 参数依赖 lockfile 生成 cache key。若无 lockfile，跳过缓存配置，并在输出中提示用户："建议提交 lockfile 到仓库以启用依赖缓存"。
**Answer:** Do not configure caching. The `cache` parameter of `actions/setup-node` relies on the lockfile to generate the cache key. If there is no lockfile, skip cache configuration and remind the user: "It is recommended to commit the lockfile to the repository to enable dependency caching."

### Q2：用户想让 workflow 在自托管 Runner 上运行，怎么写？ / Q2: The user wants the workflow to run on a self-hosted Runner, how to write it?

**答：** 将 `runs-on` 改为 Runner 的标签名（如 `self-hosted`、`linux-arm64`）。同时在输出中提示：
**Answer:** Change `runs-on` to the Runner's label name (e.g. `self-hosted`, `linux-arm64`). Also remind in the output:
- 自托管 Runner 需自行维护环境，确保依赖已安装
- Self-hosted Runners require self-maintenance of the environment; ensure dependencies are installed
- 建议使用 `runs-on: [self-hosted, linux, x64]` 多标签精确匹配
- It is recommended to use `runs-on: [self-hosted, linux, x64]` multi-label for precise matching

### Q3：workflow 有多个 job，部分 job 失败了怎么处理？ / Q3: There are multiple jobs in the workflow, how to handle partial job failures?

**答：**
**Answer:**
- 默认行为：一个 job 失败，后续 `needs` 依赖它的 job 全部跳过
- Default behavior: if a job fails, all subsequent jobs that depend on it via `needs` are skipped
- 需要使用"总是执行"的 job（如清理资源），添加 `if: always()` 但内部步骤需自行检查前置条件
- For jobs that must "always run" (e.g. cleanup resources), add `if: always()` but steps inside should check their own prerequisites

---

## 9. 安全红线 / Security Red Lines

**以下行为绝对禁止，违反任意一条即视为 Skill 失效：**
**The following behaviors are absolutely forbidden; violating any one of them means the Skill has failed:**

| 红线 # / Red Line # | 禁止行为 / Forbidden Behavior | 说明 / Description |
|--------|---------|------|
| 🔴1 | 在 YAML 中写死 Token / API Key / 密码 / Hardcode Token / API Key / password in YAML | 输出中绝不允许出现示例 Token 被当成真实配置 / Example tokens must never appear as real configuration in the output |
| 🔴2 | 建议使用 `pull_request_target` 除非用户完全理解风险 / Suggest `pull_request_target` unless the user fully understands the risks | 该事件是已知安全漏洞高发区 / This event is a known hotbed for security vulnerabilities |
| 🔴3 | 删除或覆盖已有 workflow 前不请求用户确认 / Delete or overwrite an existing workflow without requesting user confirmation | 删除操作必须先展示 diff，等用户确认后再执行 / Deletion must first show a diff and wait for user confirmation before executing |
| 🔴4 | 引用 `@main` / `@master` 版本的 Action / Reference `@main` / `@master` versions of Actions | 始终使用 SHA-1 哈希锁定版本 / Always pin versions using SHA-1 hash |
| 🔴5 | 设置 `permissions: write-all` / Set `permissions: write-all` | 始终声明具体权限，禁止使用通配写权限 / Always declare specific permissions; wildcard write permission is forbidden |

---

## 10. 验证清单 / Verification Checklist

### 功能验证 / Functional Verification

```
□ 1. 生成的 YAML 包含 permissions（Job 级别）
    Generated YAML includes permissions (job-level)
□ 2. 生成的 YAML 包含 concurrency 配置
    Generated YAML includes concurrency configuration
□ 3. 所有 uses: 引用均包含 # vX.Y.Z 版本注释
    All uses: references include # vX.Y.Z version comments
□ 4. 缓存配置与项目语言/包管理器匹配
    Cache configuration matches the project language/package manager
□ 5. on: 触发事件与用户请求场景一致
    on: trigger events are consistent with the user's request scenario
□ 6. 没有硬编码的密钥、Token 或密码
    No hardcoded secrets, tokens, or passwords
□ 7. runs-on 默认使用 ubuntu-latest（除非有特殊需求）
    runs-on defaults to ubuntu-latest (unless there are special requirements)
```

### 运行验证（可复制粘贴） / Runtime Verification (Copy-Paste Ready)

```bash
# 1. YAML 语法校验 / YAML syntax validation
yamllint .github/workflows/*.yml

# 2. 若未安装 yamllint，用 Python 替代 / If yamllint is not installed, use Python instead
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

# 3. 安全关键字扫描（检查是否误写密钥） / Security keyword scan (check for leaked secrets)
rg -n 'sk-[a-zA-Z0-9]{20,}' .github/workflows/       # OpenAI Key
rg -n 'ghp_[a-zA-Z0-9]{36}' .github/workflows/       # GitHub PAT / GitHub 个人访问令牌
rg -n 'AKIA[a-zA-Z0-9]{16}' .github/workflows/       # AWS Access Key / AWS 访问密钥

# 4. 检查是否存在浮动版本引用 / Check for floating version references
rg -n '@main|@master' .github/workflows/              # 应返回空 / Should return empty
rg -n 'uses:\s+\S+@v\d+$' .github/workflows/          # 应返回空（必须锁 SHA） / Should return empty (must pin SHA)

# 5. 检查是否缺少必含配置 / Check for missing required configurations
echo "=== 检查 permissions / Check permissions ===" && rg -c 'permissions:' .github/workflows/
echo "=== 检查 concurrency / Check concurrency ===" && rg -c 'concurrency:' .github/workflows/
```
