# GitHub Actions 自动化工作流 Skill（通用版）

> 本文件为平台无关的通用版本，可直接复制到任意 AI 编程助手中使用。
> 适用平台：Cursor Rules · GitHub Copilot · Windsurf · Aider · Cline · CodeBuddy · 自定义 System Prompt

## 1. 触发条件

**当用户的请求满足以下任一条件时，立即激活本 Skill：**

- 请求"创建/生成/写一个 GitHub Actions workflow"
- 请求"审查/优化/修复 CI/CD 配置"或"检查 workflow 安全性"
- 请求添加自动化能力：自动测试、自动发布、自动标签、自动依赖更新
- 提到具体 Action 名称：`actions/checkout`、`actions/cache`、`actions/upload-artifact`
- 提到 workflow 关键字：`on:`、`jobs:`、`runs-on`、`permissions`、`concurrency`
- 请求将某个手动流程"自动化"或"放到 CI 里"

**不触发的情况：** 用户讨论其他 CI 平台（Jenkins/GitLab CI/CircleCI）、纯本地脚本、Docker 镜像构建（无 Actions 上下文）。

---

## 2. 核心工作流决策树

当用户请求生成 Workflow 时，按以下决策树确定骨架：

```
用户请求
│
├─ 包含 "PR" / "push" / "提交" / "lint" / "测试"
│  └─ → 生成 CI Workflow（模板 A）
│
├─ 包含 "Tag" / "Release" / "发布" / "打包"
│  └─ → 生成 Release Workflow（模板 B）
│
├─ 包含 "定时" / "cron" / "定期" / "依赖更新" / "Dependabot"
│  └─ → 生成 Scheduled Workflow（模板 C）
│
├─ 包含 "安全" / "扫描" / "漏洞" / "密钥" / "CodeQL"
│  └─ → 生成 Security Workflow（模板 D）
│
└─ 包含 "Issue" / "PR 自动" / "标签" / "回复" / "分类"
   └─ → 生成 Event-driven Automation Workflow（模板 E）
```

---

## 3. 生成指令

### 3.1 骨架生成阶段

**按以下顺序执行，不要跳过任何步骤：**

1. **确认运行环境。** 询问用户项目语言/框架（Node.js/Python/Go/Rust/Docker），若用户未提供，从仓库文件推断（检测 `package.json`、`go.mod`、`Cargo.toml` 等）。
2. **确认触发事件。** 根据决策树确定 `on:` 配置。
3. **确认 Runner。** 默认使用 `ubuntu-latest`，除非用户明确要求其他平台。
4. **生成 YAML。** 从下方对应模板中选择骨架，填充用户具体信息。

### 3.2 必含配置

**每个生成的 Workflow 必须包含以下三项配置，缺一不可：**

| 配置项 | 位置 | 要求 |
|--------|------|------|
| `permissions` | Job 级别 | 显式声明，最小权限原则 |
| `concurrency` | Job 级别 | 同分支/PR 自动取消旧运行 |
| Action 版本锁定 | Step 级别 | 使用 SHA-1 哈希，禁止 `@main` / `@master` |

### 3.3 模板 A：CI Workflow

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
      # 根据语言插入对应的 setup 和 lint 步骤

  test:
    needs: lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
      # 根据语言插入对应的 setup 和 test 步骤
      # 必须包含缓存配置（见 §3.4）
```

### 3.4 缓存策略速查表

| 语言/生态 | setup Action | 缓存配置 |
|-----------|-------------|---------|
| Node.js (npm) | `actions/setup-node@v4` | `cache: 'npm'` |
| Node.js (pnpm) | `actions/setup-node@v4` | `cache: 'pnpm'` |
| Python | `actions/setup-python@v5` | `cache: 'pip'` |
| Go | `actions/setup-go@v5` | `cache: true` |
| Rust | `actions-rust-lang/setup-rust-toolchain@v1` | 手动 `actions/cache@v4` 缓存 `~/.cargo` 和 `target/` |
| Docker | 无 | 使用 `docker/build-push-action` 的 `cache-from` / `cache-to` |

### 3.5 模板 B：Release Workflow

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
      cancel-in-progress: false  # Release 不允许取消

    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
      # 构建产物 → 创建 Release → 上传 Artifact
```

### 3.6 模板 D：Security / CodeQL

```yaml
name: CodeQL

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '31 7 * * 3'  # 每周三运行

permissions:
  security-events: write
  contents: read

jobs:
  analyze:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        language: ['javascript', 'python']  # 根据项目调整
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
      - uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}
      - uses: github/codeql-action/analyze@v3
```

---

## 4. Before/After 对比示例

### 示例 1：权限收紧

```yaml
# ❌ Before — 无权限声明（继承默认写权限，安全风险）
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

# ✅ After — 显式最小权限
permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
```

### 示例 2：并发控制

```yaml
# ❌ Before — 无并发控制，同一 PR 多次推送会堆积运行
name: CI
on: [pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps: [ ... ]

# ✅ After — 新提交自动取消旧运行
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

### 示例 3：Action 版本锁定

```yaml
# ❌ Before — 浮动版本，供应链攻击风险
- uses: actions/checkout@v4
- uses: docker/setup-buildx-action@master

# ✅ After — 精确锁定到 commit SHA
- uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
- uses: docker/setup-buildx-action@c47758b77c9736f4b2ef4073d4d51994fabfe349 # v3.8.0
```

---

## 5. Few-Shot 示例

### 示例 1：最常见场景 — "帮我给 Node.js 项目加 CI"

**用户输入：**
> 帮我写一个 GitHub Actions，PR 的时候跑 eslint 和 jest

**AI 输出（按本 Skill 执行）：**
1. 检测项目：确认有 `package.json`，推断 Node.js 项目
2. 确定模板：模板 A（CI）
3. 生成包含：permissions、concurrency、缓存 `cache: 'npm'`、npm ci → lint → test 流程

---

### 示例 2：稍有变化 — "定时更新依赖"

**用户输入：**
> 我想让 GitHub 每个星期一自动检查 npm 依赖有没有更新

**AI 输出（按本 Skill 执行）：**
1. 确定模板：模板 C（Scheduled）
2. 生成 `.github/workflows/dependency-update.yml`
3. 使用 `cron: '0 9 * * 1'` + `actions/checkout` + `npm outdated` + 若发现更新则自动创建 PR
4. 权限仅给 `contents: write` 和 `pull-requests: write`

---

### 示例 3：边界情况 — "workflow 跑太慢了"

**用户输入：**
> 我的 CI 每次跑 15 分钟，怎么优化？

**AI 输出（按本 Skill 执行）：**
1. 审查现有 workflow 文件
2. 检查是否有缓存配置 → 若无，插入对应语言缓存
3. 检查是否可拆分 job 并行 → 将 lint 和 test 拆为独立 job
4. 给出预计节省时间评估

---

### 示例 4：安全审查 — "帮我检查 workflow 有没有安全问题"

**用户输入：**
> 帮我审查 `.github/workflows/ci.yml` 的安全性

**AI 输出（按本 Skill 执行）：**

| 检查项 | 状态 | 说明 |
|--------|------|------|
| `permissions` 显式声明 | ❌ | 未设置，默认有写权限 |
| Action 版本锁定 | ❌ | `actions/checkout@v4` 未锁 SHA |
| `concurrency` 配置 | ✅ | 已配置 |
| 密钥引用方式 | ✅ | 使用 `${{ secrets.X }}`，无明文 |
| `pull_request_target` 风险 | ✅ | 未使用（避免高危事件） |

---

### 示例 5：多语言项目 — "Rust + Node.js 的 CI"

**用户输入：**
> 我的项目有 Rust 后端和 React 前端，怎么写一个 CI？

**AI 输出（按本 Skill 执行）：**
1. 检测项目根目录：识别 `Cargo.toml` + `package.json`
2. 生成矩阵策略或拆分两个 job
3. Rust 部分：手动配置 `actions/cache@v4` 缓存 `~/.cargo` 和 `target/`
4. Node.js 部分：`setup-node` 自带 `cache: 'npm'`
5. 两个 job 并行运行，使用不同 `working-directory`

---

## 6. 检查点（Checkpoints）

| 检查点 # | 位置 | 验证内容 | 验证方式 |
|----------|------|----------|----------|
| CP1 | 骨架生成后 | `on:` 事件是否正确匹配用户需求 | 对照决策树确认 |
| CP2 | YAML 生成后 | 是否包含三项必含配置 | 搜索 `permissions:`、`concurrency:`、`# v` |
| CP3 | YAML 生成后 | YAML 语法是否合法 | `yamllint` 或 Python `yaml.safe_load()` |
| CP4 | 缓存配置后 | 缓存路径是否匹配语言生态 | 对照 §3.4 缓存策略表 |
| CP5 | 版本锁定后 | 所有 `uses:` 是否都锁定了 SHA | 搜索 `@main`、`@master`、`@v\d$` 不得出现 |

---

## 7. 注意事项（边界与易错点）

1. **`pull_request_target` 事件是安全雷区。** 该事件在目标仓库上下文运行，可以读取 secrets。除非用户明确要求且理解风险，否则永远不要建议使用它。优先使用 `pull_request`。

2. **`cancel-in-progress` 对 Release 设 `false`。** Release job 的 concurrency 中 `cancel-in-progress` 必须为 `false`，否则 tag push 可能被意外取消。

3. **GitHub 免费 Runner 资源有限。** macOS Runner 消耗 10x 计费分钟，Windows 消耗 2x。优先使用 `ubuntu-latest`。

4. **`GITHUB_TOKEN` 默认权限是 `write`。** 如果不显式设置 `permissions`，token 拥有仓库写权限，存在安全风险。

5. **矩阵策略 `fail-fast: false`。** 多语言/多平台矩阵测试中，设 `fail-fast: false`，防止一个平台失败导致其他平台被跳过。

6. **工作流文件路径固定。** 所有 workflow 文件必须放在 `.github/workflows/` 目录下，文件名必须以 `.yml` 或 `.yaml` 结尾。

7. **缓存 Key 的唯一性。** 手动配置 `actions/cache` 时，`key` 必须包含 `hashFiles('lock文件路径')`，否则缓存永远不会失效。

8. **加密变量和密钥。** 绝不在 workflow 中硬编码 Token/密码。使用 `${{ secrets.XXX }}`，并在 GitHub Settings → Secrets 中配置。

---

## 8. 实用 FAQ

### Q1：用户的仓库没有 lockfile（如 `package-lock.json`），怎么配缓存？

**答：** 不要配置缓存。`actions/setup-node` 的 `cache` 参数依赖 lockfile 生成 cache key。若无 lockfile，跳过缓存配置，并在输出中提示用户："建议提交 lockfile 到仓库以启用依赖缓存"。

### Q2：用户想让 workflow 在自托管 Runner 上运行，怎么写？

**答：** 将 `runs-on` 改为 Runner 的标签名（如 `self-hosted`、`linux-arm64`）。同时在输出中提示：
- 自托管 Runner 需自行维护环境，确保依赖已安装
- 建议使用 `runs-on: [self-hosted, linux, x64]` 多标签精确匹配

### Q3：workflow 有多个 job，部分 job 失败了怎么处理？

**答：**
- 默认行为：一个 job 失败，后续 `needs` 依赖它的 job 全部跳过
- 需要使用"总是执行"的 job（如清理资源），添加 `if: always()` 但内部步骤需自行检查前置条件

---

## 9. 安全红线

**以下行为绝对禁止，违反任意一条即视为 Skill 失效：**

| 红线 # | 禁止行为 | 说明 |
|--------|---------|------|
| 🔴1 | 在 YAML 中写死 Token / API Key / 密码 | 输出中绝不允许出现示例 Token 被当成真实配置 |
| 🔴2 | 建议使用 `pull_request_target` 除非用户完全理解风险 | 该事件是已知安全漏洞高发区 |
| 🔴3 | 删除或覆盖已有 workflow 前不请求用户确认 | 删除操作必须先展示 diff，等用户确认后再执行 |
| 🔴4 | 引用 `@main` / `@master` 版本的 Action | 始终使用 SHA-1 哈希锁定版本 |
| 🔴5 | 设置 `permissions: write-all` | 始终声明具体权限，禁止使用通配写权限 |

---

## 10. 验证清单

### 功能验证

```
□ 1. 生成的 YAML 包含 permissions（Job 级别）
□ 2. 生成的 YAML 包含 concurrency 配置
□ 3. 所有 uses: 引用均包含 # vX.Y.Z 版本注释
□ 4. 缓存配置与项目语言/包管理器匹配
□ 5. on: 触发事件与用户请求场景一致
□ 6. 没有硬编码的密钥、Token 或密码
□ 7. runs-on 默认使用 ubuntu-latest（除非有特殊需求）
```

### 运行验证

```bash
# YAML 语法校验
yamllint .github/workflows/*.yml

# 若未安装 yamllint，用 Python 替代
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

# 安全关键字扫描
rg -n 'sk-[a-zA-Z0-9]{20,}' .github/workflows/       # OpenAI Key
rg -n 'ghp_[a-zA-Z0-9]{36}' .github/workflows/       # GitHub PAT
rg -n 'AKIA[a-zA-Z0-9]{16}' .github/workflows/       # AWS Access Key

# 检查浮动版本引用（应返回空）
rg -n '@main|@master' .github/workflows/

# 检查必含配置
rg -c 'permissions:' .github/workflows/
rg -c 'concurrency:' .github/workflows/
```
