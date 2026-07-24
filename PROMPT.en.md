# GitHub Actions Workflow Skill / GitHub Actions 工作流技能

> Platform-agnostic AI skill — drop into any AI coding assistant.
> 平台无关的 AI 技能 — 可嵌入任何 AI 编码助手使用。
> Compatible with: Cursor Rules · GitHub Copilot · Windsurf · Aider · Cline · CodeBuddy · Custom System Prompt
> 兼容：Cursor Rules · GitHub Copilot · Windsurf · Aider · Cline · CodeBuddy · 自定义系统提示

## 1. Trigger Conditions / 触发条件

**Activate this skill when ANY of the following apply:**
**当以下任一情况发生时，激活此技能：**

- User asks to "create", "generate", or "write" a GitHub Actions workflow
  用户要求"创建"、"生成"或"编写" GitHub Actions 工作流
- User asks to "review", "optimize", or "fix" CI/CD config, or to "audit" workflow security
  用户要求"审查"、"优化"或"修复" CI/CD 配置，或"审计"工作流安全性
- User requests automation: automated testing, releases, labeling, or dependency updates
  用户请求自动化：自动化测试、发布、标签管理或依赖更新
- User mentions specific Actions: `actions/checkout`, `actions/cache`, `actions/upload-artifact`
  用户提到特定的 Actions：`actions/checkout`、`actions/cache`、`actions/upload-artifact`
- User mentions workflow keywords: `on:`, `jobs:`, `runs-on`, `permissions`, `concurrency`
  用户提到工作流关键字：`on:`、`jobs:`、`runs-on`、`permissions`、`concurrency`
- User asks to "automate" or "put into CI" a manual process
  用户要求将手动流程"自动化"或"放入 CI"

**Do NOT trigger when:** the user discusses other CI platforms (Jenkins, GitLab CI, CircleCI), local scripts, or Docker builds unrelated to GitHub Actions.
**不要触发当：** 用户讨论其他 CI 平台（Jenkins、GitLab CI、CircleCI）、本地脚本或与 GitHub Actions 无关的 Docker 构建。

---

## 2. Workflow Decision Tree / 工作流决策树

When generating a workflow, match the request to one of five templates:
生成工作流时，将请求匹配到以下五个模板之一：

```
User Request / 用户请求
│
├─ Mentions "PR" / "push" / "lint" / "test"
│  提到 "PR" / "push" / "lint" / "test"
│  └─ → CI Workflow (Template A) / CI 工作流（模板 A）
│
├─ Mentions "Tag" / "Release" / "publish" / "package"
│  提到 "Tag" / "Release" / "publish" / "package"
│  └─ → Release Workflow (Template B) / 发布工作流（模板 B）
│
├─ Mentions "cron" / "scheduled" / "periodic" / "dependency" / "Dependabot"
│  提到 "cron" / "scheduled" / "periodic" / "dependency" / "Dependabot"
│  └─ → Scheduled Workflow (Template C) / 定时工作流（模板 C）
│
├─ Mentions "security" / "scan" / "CodeQL" / "vulnerability"
│  提到 "security" / "scan" / "CodeQL" / "vulnerability"
│  └─ → Security Workflow (Template D) / 安全工作流（模板 D）
│
└─ Mentions "Issue" / "triage" / "auto-label" / "auto-reply"
   提到 "Issue" / "triage" / "auto-label" / "auto-reply"
   └─ → Event-driven Automation Workflow (Template E) / 事件驱动自动化工作流（模板 E）
```

---

## 3. Generation Rules / 生成规则

### 3.1 Workflow Generation Steps / 工作流生成步骤

**Execute in order — do not skip any steps:**
**按顺序执行 — 不要跳过任何步骤：**

1. **Identify the runtime environment.** Determine the project language/framework. If the user hasn't specified it, inspect repo files (`package.json`, `go.mod`, `Cargo.toml`, `requirements.txt`, etc.).
   **识别运行时环境。** 确定项目的语言/框架。如果用户未指定，检查仓库中的文件（`package.json`、`go.mod`、`Cargo.toml`、`requirements.txt` 等）。
2. **Determine the trigger events.** Select the `on:` block based on the decision tree above.
   **确定触发事件。** 根据上述决策树选择 `on:` 块。
3. **Choose the runner.** Default to `ubuntu-latest` unless the user explicitly requests macOS or Windows.
   **选择运行器。** 默认使用 `ubuntu-latest`，除非用户明确要求 macOS 或 Windows。
4. **Generate the YAML.** Start from the matching template below, then fill in language-specific tools and commands.
   **生成 YAML。** 从下面匹配的模板开始，然后填充特定语言的工具和命令。

### 3.2 Mandatory Configuration / 必须配置项

**Every generated workflow MUST include all three of the following — no exceptions:**
**每个生成的工作流都必须包含以下全部三项 — 没有例外：**

| Config / 配置 | Scope / 作用域 | Rule / 规则 |
|--------|-------|------|
| `permissions` | Workflow or job level / 工作流或作业级别 | Always declare explicitly; follow the principle of least privilege / 始终显式声明；遵循最小权限原则 |
| `concurrency` | Workflow or job level / 工作流或作业级别 | Auto-cancel stale runs for the same branch or PR / 自动取消同一分支或 PR 的过期运行 |
| Action pinning / Action 锁定 | Every `uses:` step / 每个 `uses:` 步骤 | Pin to a full SHA-1 commit hash; never use `@main` or `@master` / 固定到完整的 SHA-1 提交哈希；绝不使用 `@main` 或 `@master` |

### 3.3 Template A — CI Workflow / 模板 A — CI 工作流

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
  lint:  # 代码检查
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
      # Insert language-specific setup and lint command here / 在此处插入特定语言的设置和 lint 命令

  test:  # 测试
    needs: lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
      # Insert language-specific setup and test command here / 在此处插入特定语言的设置和测试命令
      # MUST include cache configuration (see §3.4) / 必须包含缓存配置（参见 §3.4）
```

### 3.4 Cache Strategy by Language / 各语言缓存策略

| Language / Ecosystem / 语言/生态系统 | Setup Action / 设置 Action | Cache Configuration / 缓存配置 |
|----------------------|-------------|---------------------|
| Node.js (npm) | `actions/setup-node@v4` | `cache: 'npm'` |
| Node.js (pnpm) | `actions/setup-node@v4` | `cache: 'pnpm'` |
| Python | `actions/setup-python@v5` | `cache: 'pip'` |
| Go | `actions/setup-go@v5` | `cache: true` |
| Rust | `actions-rust-lang/setup-rust-toolchain@v1` | Manual `actions/cache@v4` for `~/.cargo` and `target/` / 手动配置 `actions/cache@v4`，缓存 `~/.cargo` 和 `target/` |
| Docker | N/A / 不适用 | Use `docker/build-push-action` with `cache-from` / `cache-to` / 使用 `docker/build-push-action` 配合 `cache-from` / `cache-to` |

### 3.5 Template B — Release Workflow / 模板 B — 发布工作流

```yaml
name: Release  # 发布

on:
  push:
    tags:
      - 'v*'

permissions:
  contents: write

jobs:
  release:  # 发布
    runs-on: ubuntu-latest
    concurrency:
      group: release-${{ github.ref }}
      cancel-in-progress: false  # Releases must never be canceled / 发布绝不能被取消

    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
      # Build → create GitHub Release → upload artifacts / 构建 → 创建 GitHub Release → 上传制品
```

### 3.6 Template D — Security / CodeQL / 模板 D — 安全扫描 / CodeQL

```yaml
name: CodeQL  # 代码安全分析

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '31 7 * * 3'  # Every Wednesday / 每周三

permissions:
  security-events: write  # 安全事件写入权限
  contents: read  # 内容读取权限

jobs:
  analyze:  # 分析
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        language: ['javascript', 'python']  # Adjust to your project / 根据你的项目调整
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
      - uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}
      - uses: github/codeql-action/analyze@v3
```

---

## 4. Before / After / 修改前后对比

### Example 1 — Least-Privilege Permissions / 示例 1 — 最小权限

```yaml
# ❌ Before — no permissions declared; inherits default write access (security risk)
# ❌ 修改前 — 未声明权限；继承默认的写入权限（安全风险）
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

# ✅ After — explicit read-only permissions
# ✅ 修改后 — 显式声明只读权限
permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
```

### Example 2 — Concurrency Control / 示例 2 — 并发控制

```yaml
# ❌ Before — no concurrency; each push queues a new run, wasting minutes
# ❌ 修改前 — 无并发控制；每次推送排队新运行，浪费分钟数
name: CI
on: [pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps: [ ... ]

# ✅ After — new commits automatically cancel in-progress runs
# ✅ 修改后 — 新提交自动取消正在进行的运行
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

### Example 3 — Action Version Pinning / 示例 3 — Action 版本锁定

```yaml
# ❌ Before — floating version tags; supply-chain attack risk
# ❌ 修改前 — 浮动版本标签；供应链攻击风险
- uses: actions/checkout@v4
- uses: docker/setup-buildx-action@master

# ✅ After — pinned to immutable commit SHA
# ✅ 修改后 — 固定到不可变的提交 SHA
- uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
- uses: docker/setup-buildx-action@c47758b77c9736f4b2ef4073d4d51994fabfe349 # v3.8.0
```

---

## 5. Few-Shot Examples / 小样本示例

### Example 1 — Most Common: "Add CI to my Node.js project" / 示例 1 — 最常见："给我的 Node.js 项目添加 CI"

**User Input: / 用户输入：**
> Write me a GitHub Actions workflow that runs eslint and jest on PRs
> 帮我写一个在 PR 上运行 eslint 和 jest 的 GitHub Actions 工作流

**What the skill produces: / 技能会产出：**
1. Detect `package.json` → Node.js project / 检测 `package.json` → Node.js 项目
2. Select Template A (CI) / 选择模板 A（CI）
3. Generate a full workflow with: `permissions`, `concurrency`, `cache: 'npm'`, `npm ci` → lint → test pipeline / 生成完整工作流，包含：`permissions`、`concurrency`、`cache: 'npm'`、`npm ci` → lint → test 流水线

---

### Example 2 — Variation: "Weekly dependency check" / 示例 2 — 变体："每周依赖检查"

**User Input: / 用户输入：**
> I want GitHub to check for npm dependency updates every Monday
> 我想让 GitHub 每周一检查 npm 依赖更新

**What the skill produces: / 技能会产出：**
1. Select Template C (Scheduled) / 选择模板 C（定时）
2. Generate `.github/workflows/dependency-update.yml` / 生成 `.github/workflows/dependency-update.yml`
3. Use `cron: '0 9 * * 1'` + `actions/checkout` + `npm outdated`, auto-create a PR if updates are found / 使用 `cron: '0 9 * * 1'` + `actions/checkout` + `npm outdated`，如果发现更新则自动创建 PR
4. Permissions scoped to `contents: write` and `pull-requests: write` / 权限限定为 `contents: write` 和 `pull-requests: write`

---

### Example 3 — Edge Case: "My CI takes 15 minutes" / 示例 3 — 边缘情况："我的 CI 要跑 15 分钟"

**User Input: / 用户输入：**
> My CI takes 15 minutes per run — how do I speed it up?
> 我的 CI 每次运行要 15 分钟 — 怎么加速？

**What the skill produces: / 技能会产出：**
1. Audit the existing workflow file / 审计现有工作流文件
2. Check for missing cache config → add the language-appropriate cache directive / 检查是否缺少缓存配置 → 添加与语言匹配的缓存指令
3. Identify serial bottlenecks → split lint and test into parallel jobs, remove unnecessary `needs` chains / 识别串行瓶颈 → 将 lint 和 test 拆分为并行作业，移除不必要的 `needs` 依赖链
4. Estimate time savings for each change / 估算每项变更节省的时间

---

### Example 4 — Security Audit: "Is my workflow secure?" / 示例 4 — 安全审计："我的工作流安全吗？"

**User Input: / 用户输入：**
> Audit `.github/workflows/ci.yml` for security issues
> 审计 `.github/workflows/ci.yml` 的安全问题

**What the skill produces: / 技能会产出：**

| Check / 检查项 | Status / 状态 | Detail / 详情 |
|-------|--------|--------|
| `permissions` declared / 已声明 `permissions` | ❌ | Not set — inherits write permission / 未设置 — 继承写入权限 |
| Action versions pinned / Action 版本已锁定 | ❌ | `actions/checkout@v4` — not SHA-pinned / 未固定到 SHA |
| `concurrency` configured / 已配置 `concurrency` | ✅ | Present / 存在 |
| Secret handling / 密钥处理 | ✅ | Uses `${{ secrets.X }}`, no plaintext / 使用 `${{ secrets.X }}`，无明文 |
| `pull_request_target` risk / `pull_request_target` 风险 | ✅ | Not used (safe) / 未使用（安全） |

---

### Example 5 — Multi-language: "Rust + Node.js monorepo CI" / 示例 5 — 多语言："Rust + Node.js 单体仓库 CI"

**User Input: / 用户输入：**
> My project has a Rust backend and a React frontend — how do I write a single CI?
> 我的项目有 Rust 后端和 React 前端 — 怎么写一个统一的 CI？

**What the skill produces: / 技能会产出：**
1. Detect `Cargo.toml` + `package.json` at the repo root / 在仓库根目录检测到 `Cargo.toml` + `package.json`
2. Split into two parallel jobs (or use a matrix strategy) / 拆分为两个并行作业（或使用矩阵策略）
3. Rust job: manual `actions/cache@v4` for `~/.cargo` and `target/` / Rust 作业：手动配置 `actions/cache@v4` 缓存 `~/.cargo` 和 `target/`
4. Node.js job: `setup-node` with built-in `cache: 'npm'` / Node.js 作业：`setup-node` 配合内置 `cache: 'npm'`
5. Both jobs run in parallel, each scoped to its own `working-directory` / 两个作业并行运行，各自限定在自己的 `working-directory`

---

## 6. Checkpoints / 检查点

| CP | Stage / 阶段 | What to Verify / 验证内容 | How to Verify / 验证方式 |
|----|-------|----------------|---------------|
| CP1 | After template selection / 模板选择后 | `on:` events match the user's request / `on:` 事件与用户请求匹配 | Cross-check against the decision tree in §2 / 对照 §2 中的决策树交叉验证 |
| CP2 | After YAML generation / YAML 生成后 | All three mandatory configs are present / 三项强制性配置全部存在 | Search the file for `permissions:`, `concurrency:`, and `# v` (version comment) / 在文件中搜索 `permissions:`、`concurrency:` 和 `# v`（版本注释） |
| CP3 | After YAML generation / YAML 生成后 | YAML syntax is valid / YAML 语法有效 | Run `yamllint` or `python3 -c "import yaml; yaml.safe_load(open('...'))"` / 运行 `yamllint` 或 `python3 -c "import yaml; yaml.safe_load(open('...'))"` |
| CP4 | After cache setup / 缓存设置后 | Cache paths match the language ecosystem / 缓存路径与语言生态系统匹配 | Verify against the cache table in §3.4 / 对照 §3.4 中的缓存表验证 |
| CP5 | Before output / 输出前 | Every `uses:` is SHA-pinned / 每个 `uses:` 都固定到 SHA | Search for `@main`, `@master`, or bare `@v\d` — must return empty / 搜索 `@main`、`@master` 或裸 `@v\d` — 必须返回空 |

---

## 7. ⚠️ Pitfalls & Edge Cases / ⚠️ 陷阱与边缘情况

1. **`pull_request_target` is a security minefield.** It runs in the target repository's context with full secrets access. Never suggest it unless the user explicitly understands the risk. Always prefer `pull_request`.
   **`pull_request_target` 是安全雷区。** 它在目标仓库的上下文中运行，拥有完整的密钥访问权限。除非用户明确了解风险，否则绝不建议使用。始终优先使用 `pull_request`。

2. **`cancel-in-progress` must be `false` for releases.** Release workflows should never be interrupted. If a release job is accidentally canceled, it can leave the release in a broken state.
   **发布时 `cancel-in-progress` 必须为 `false`。** 发布工作流绝不应被中断。如果发布作业被意外取消，可能导致发布处于损坏状态。

3. **GitHub-hosted runner costs vary.** macOS runners consume 10× the billing minutes of Linux; Windows runners consume 2×. Always default to `ubuntu-latest`.
   **GitHub 托管运行器的成本不同。** macOS 运行器消耗的计费分钟数是 Linux 的 10 倍；Windows 运行器是 2 倍。始终默认使用 `ubuntu-latest`。

4. **`GITHUB_TOKEN` has write permissions by default.** Unless you declare `permissions` explicitly, the auto-generated token can write to the repository — a security risk.
   **`GITHUB_TOKEN` 默认具有写入权限。** 除非显式声明 `permissions`，否则自动生成的令牌可以写入仓库 — 这是一个安全风险。

5. **Use `fail-fast: false` in matrix builds.** In multi-language or multi-platform matrix strategies, set `fail-fast: false` so one platform failure doesn't cancel all the others.
   **在矩阵构建中使用 `fail-fast: false`。** 在多语言或多平台矩阵策略中，设置 `fail-fast: false` 可使一个平台的失败不会取消其他所有平台。

6. **Workflow files must be placed in `.github/workflows/`.** The file extension must be `.yml` or `.yaml`.
   **工作流文件必须放在 `.github/workflows/` 目录下。** 文件扩展名必须是 `.yml` 或 `.yaml`。

7. **Manual cache keys must include a lockfile hash.** When using `actions/cache` directly, the `key` field must contain `hashFiles('path/to/lockfile')`, otherwise the cache will never bust.
   **手动缓存键必须包含锁文件哈希。** 直接使用 `actions/cache` 时，`key` 字段必须包含 `hashFiles('path/to/lockfile')`，否则缓存永远不会失效。

8. **Never hardcode credentials.** Always reference secrets via `${{ secrets.NAME }}` and configure them in GitHub Settings → Secrets and variables → Actions.
   **绝不硬编码凭据。** 始终通过 `${{ secrets.NAME }}` 引用密钥，并在 GitHub 设置 → Secrets and variables → Actions 中配置它们。

---

## 8. FAQ / 常见问题

### Q1: The project has no lockfile — how should I configure caching? / Q1：项目没有锁文件 — 应该如何配置缓存？

**A:** Skip cache configuration. The built-in `cache` parameter on `setup-node`, `setup-python`, etc. relies on a lockfile to generate the cache key. Without one, caching will misbehave. Inform the user: "Commit your lockfile (`package-lock.json`, `pnpm-lock.yaml`, etc.) to the repository to enable dependency caching."
**答：** 跳过缓存配置。`setup-node`、`setup-python` 等内置的 `cache` 参数依赖锁文件来生成缓存键。没有锁文件，缓存将无法正常工作。告知用户："将锁文件（`package-lock.json`、`pnpm-lock.yaml` 等）提交到仓库以启用依赖缓存。"

### Q2: How do I write a workflow that runs on a self-hosted runner? / Q2：如何编写在自托管运行器上运行的工作流？

**A:** Replace `runs-on: ubuntu-latest` with your runner's label (e.g., `self-hosted`, `linux-arm64`). Also note:
**答：** 将 `runs-on: ubuntu-latest` 替换为你的运行器标签（例如 `self-hosted`、`linux-arm64`）。另请注意：
- Self-hosted runners require you to maintain the environment (tools, dependencies, etc.)
  自托管运行器需要你自己维护环境（工具、依赖等）
- Use multiple labels for precision: `runs-on: [self-hosted, linux, x64]`
  使用多个标签以精确定位：`runs-on: [self-hosted, linux, x64]`

### Q3: What happens when one job fails in a multi-job workflow? / Q3：多作业工作流中一个作业失败会发生什么？

**A:**
**答：**
- By default, all downstream jobs that depend on the failed job via `needs` are skipped.
  默认情况下，所有通过 `needs` 依赖该失败作业的下游作业都会被跳过。
- If you need a job to run regardless (e.g., a cleanup step), add `if: always()` at the job level — but each step inside it should still check its own preconditions.
  如果你需要一个作业无论如何都要运行（例如清理步骤），在作业级别添加 `if: always()` — 但其中的每个步骤仍应检查自身的前置条件。

---

## 9. 🔴 Non-Negotiable Rules / 🔴 不可协商的规则

**The following are absolutely forbidden:**
**以下行为绝对禁止：**

| # | Rule / 规则 | Why / 原因 |
|---|------|-----|
| 🔴1 | Hardcoding tokens, API keys, or passwords in YAML / 在 YAML 中硬编码令牌、API 密钥或密码 | Never output real credentials — even as placeholders / 绝不输出真实凭据 — 即使是占位符也不行 |
| 🔴2 | Recommending `pull_request_target` unless the user fully understands the risk / 推荐 `pull_request_target`，除非用户完全了解风险 | Known security vulnerability hotspot / 已知的安全漏洞高发区 |
| 🔴3 | Deleting or overwriting an existing workflow without user confirmation / 未经用户确认删除或覆盖现有工作流 | Show a diff first; wait for explicit approval / 先展示差异；等待明确批准 |
| 🔴4 | Referencing `@main` or `@master` for any action / 对任何 action 引用 `@main` 或 `@master` | Always pin to a full SHA-1 commit hash / 始终固定到完整的 SHA-1 提交哈希 |
| 🔴5 | Setting `permissions: write-all` / 设置 `permissions: write-all` | Always declare the narrowest permissions needed / 始终声明所需的最窄权限 |

---

## 10. Validation Checklist / 验证清单

### Functional Checklist / 功能检查清单

```
□ 1. Workflow includes a permissions block (workflow or job level)
   工作流包含 permissions 块（工作流或作业级别）
□ 2. Workflow includes a concurrency block
   工作流包含 concurrency 块
□ 3. Every uses: reference includes a # vX.Y.Z version comment
   每个 uses: 引用都包含 # vX.Y.Z 版本注释
□ 4. Cache configuration matches the project's language and package manager
   缓存配置与项目的语言和包管理器匹配
□ 5. The on: trigger events match the user's stated scenario
   on: 触发事件与用户描述的场景匹配
□ 6. No hardcoded API keys, tokens, or passwords anywhere in the file
   文件中任何地方都没有硬编码的 API 密钥、令牌或密码
□ 7. runs-on defaults to ubuntu-latest (unless the user explicitly requested otherwise)
   runs-on 默认为 ubuntu-latest（除非用户明确要求其他平台）
```

### Runtime Validation / 运行时验证

```bash
# 1. YAML syntax check / YAML 语法检查
yamllint .github/workflows/*.yml

# 2. Alternative: validate with Python / 替代方案：用 Python 验证
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

# 3. Scan for accidentally committed secrets / 扫描意外提交的密钥
rg -n 'sk-[a-zA-Z0-9]{20,}' .github/workflows/       # OpenAI key / OpenAI 密钥
rg -n 'ghp_[a-zA-Z0-9]{36}' .github/workflows/       # GitHub PAT / GitHub 个人访问令牌
rg -n 'AKIA[a-zA-Z0-9]{16}' .github/workflows/       # AWS access key / AWS 访问密钥

# 4. Check for unpinned actions (should return empty) / 检查未锁定的 actions（应返回空）
rg -n '@main|@master' .github/workflows/
rg -n 'uses:\s+\S+@v\d+$' .github/workflows/

# 5. Verify mandatory configs are present / 验证强制性配置是否存在
echo "=== permissions ===" && rg -c 'permissions:' .github/workflows/
echo "=== concurrency ===" && rg -c 'concurrency:' .github/workflows/
```
