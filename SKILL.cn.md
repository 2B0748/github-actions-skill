---
name: github-actions
description: >
  GitHub Actions 工作流的生成、审查与优化。
  适用场景：创建 CI/CD Pipeline、配置自动化发布、添加安全扫描、审查现有 Workflow、优化缓存与并发策略。
  触发关键词：GitHub Actions、workflow、CI/CD、pipeline、YAML、actions/cache、release
  automation、Dependabot、concurrency、permissions、Runner、Job、Step。
---

# GitHub Actions 自动化工作流技能

## 1. 触发条件

**当用户请求满足以下任一条件时，立即激活此技能：**

- 请求"创建/生成/编写 GitHub Actions 工作流"
- 请求"审查/优化/修复 CI/CD 配置"或"检查工作流安全性"
- 请求添加自动化能力：自动测试、自动发布、自动标签、自动依赖更新
- 提到具体的 Action 名称：`actions/checkout`、`actions/cache`、`actions/upload-artifact`
- 提到工作流关键字：`on:`、`jobs:`、`runs-on`、`permissions`、`concurrency`
- 请求"自动化"某个手动流程，或"放到 CI 里"

**不触发的情况：** 用户讨论其他 CI 平台（Jenkins/GitLab CI/CircleCI），纯本地脚本，Docker 镜像构建（与 Actions 无关的上下文）。

---

## 2. 核心工作流决策树

生成工作流时，按以下决策树确定骨架：

```
用户请求
|
+-- 包含 "PR" / "push" / "commit" / "lint" / "test"
|   |
|   +-- --> 生成 CI 工作流（模板 A）
|
+-- 包含 "Tag" / "Release" / "publish" / "package"
|   |
|   +-- --> 生成发布工作流（模板 B）
|
+-- 包含 "scheduled" / "cron" / "periodic" / "dependency update" / "Dependabot"
|   |
|   +-- --> 生成定时工作流（模板 C）
|
+-- 包含 "security" / "scan" / "vulnerability" / "secret" / "CodeQL"
|   |
|   +-- --> 生成安全扫描工作流（模板 D）
|
+-- 包含 "Issue" / "PR auto" / "label" / "reply" / "triage"
    |
    +-- --> 生成事件驱动自动化工作流（模板 E）
```

---

## 3. 生成指令

### 3.1 骨架生成阶段

**按以下顺序执行，不得跳过任何步骤：**

1. **确认运行环境。** 询问项目语言/框架（Node.js/Python/Go/Rust/Docker）。若用户未提供，从仓库文件中推断（检测 `package.json`、`go.mod`、`Cargo.toml` 等）。
2. **确认触发事件。** 根据决策树确定 `on:` 配置。
3. **确认 Runner。** 默认使用 `ubuntu-latest`，除非用户明确要求其他平台。
4. **生成 YAML。** 从下方对应模板中选择骨架，填入用户具体信息。

### 3.2 必须包含的配置

**每个生成的工作流必须包含以下三项配置，缺一不可：**

| 配置项 | 作用域 | 规则 |
|--------|--------|------|
| `permissions` | Job 级别 | 显式声明，遵循最小权限原则 |
| `concurrency` | Job 级别 | 同一分支/PR 自动取消旧运行 |
| Action 版本锁定 | Step 级别 | 使用 SHA-1 哈希，禁止 `@main` / `@master` |

### 3.3 模板 A：CI 工作流

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
      # 插入语言相关的环境搭建和 lint 步骤

  test:
    needs: lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4
      # 插入语言相关的环境搭建和测试步骤
      # 必须包含缓存配置
```

### 3.4 缓存策略速查表

| 语言/生态 | 环境搭建 Action | 缓存配置 |
|-----------|----------------|----------|
| Node.js (npm) | `actions/setup-node@v4` | `cache: 'npm'` |
| Node.js (pnpm) | `actions/setup-node@v4` | `cache: 'pnpm'` |
| Python | `actions/setup-python@v5` | `cache: 'pip'` |
| Go | `actions/setup-go@v5` | `cache: true` |
| Rust | 无官方 setup | 手动用 `actions/cache@v4` 缓存 `target/` |
| Docker | 无 | 使用 `docker/build-push-action` 配合 `cache-from` / `cache-to` |

### 3.5 模板 B：发布工作流

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
      cancel-in-progress: false  # 发布流程不可被取消

    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4
      # 构建产物 -> 创建 Release -> 上传 Artifact
```

### 3.6 模板 D：安全扫描 / CodeQL

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
        language: ['javascript', 'python']  # 根据项目实际情况调整
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4
      - uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}
      - uses: github/codeql-action/analyze@v3
```

---

## 4. Before/After 对比示例

### 示例 1：权限加固

```yaml
# Before —— 未声明权限（继承默认 write 权限，存在安全风险）
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

# After —— 显式声明最小权限
permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4
```

### 示例 2：并发控制

```yaml
# Before —— 无并发控制，同一 PR 多次推送会堆积排队
name: CI
on: [pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps: [ ... ]

# After —— 新提交自动取消旧运行
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
# Before —— 浮动版本，存在供应链攻击风险
- uses: actions/checkout@v4
- uses: docker/setup-buildx-action@master

# After —— 精确锁定到提交 SHA
- uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
- uses: docker/setup-buildx-action@c47758b77c9736f4b2ef4073d4d51994fabfe349 # v3.8.0
```

---

## 5. Few-Shot 示例

### 示例 1：最常见场景 —— "给我的 Node.js 项目添加 CI"

**用户输入：**
> 写一个 GitHub Actions 工作流，在 PR 时运行 eslint 和 jest

**AI 输出（遵循本技能）：**
1. 检测项目：确认 `package.json` 存在，推断为 Node.js 项目
2. 确定模板：模板 A（CI）
3. 生成内容包含：permissions、concurrency、缓存 `cache: 'npm'`、npm ci -> lint -> test 流程

---

### 示例 2：轻微变体 —— "定时检查依赖更新"

**用户输入：**
> 我想让 GitHub 每周一自动检查 npm 依赖更新

**AI 输出（遵循本技能）：**
1. 确定模板：模板 C（定时任务）
2. 生成 `.github/workflows/dependency-update.yml`
3. 使用 `cron: '0 9 * * 1'` + `actions/checkout` + `npm outdated` + 发现有更新则自动创建 PR
4. 权限限制为 `contents: write` 和 `pull-requests: write`

---

### 示例 3：边缘情况 —— "工作流跑得太慢了"

**用户输入：**
> 我的 CI 每次跑 15 分钟，怎么优化？

**AI 输出（遵循本技能）：**
1. 审查现有工作流文件
2. 检查缓存配置 -> 若缺失，插入 `cache: 'npm'` 或对应语言的缓存配置
3. 检查 Job 是否可以拆分为并行执行 -> 将 lint 和 test 拆为独立 Job（通过 `needs` 去除不必要的串行依赖）
4. 给出预计节省时间的评估

---

### 示例 4：安全审查 —— "帮我检查工作流的安全问题"

**用户输入：**
> 帮我审查 `.github/workflows/ci.yml` 的安全性

**AI 输出（遵循本技能）：**

| 检查项 | 状态 | 说明 |
|--------|------|------|
| `permissions` 显式声明 | 缺失 | 未设置，默认继承 write 权限 |
| Action 版本锁定 | 缺失 | `actions/checkout@v4` 未锁定 SHA |
| `concurrency` 配置 | 已配置 | 已正确配置 |
| Secret 引用方式 | 正常 | 使用 `${{ secrets.X }}`，无明文 |
| `pull_request_target` 风险 | 正常 | 未使用（已规避高风险事件） |

---

### 示例 5：多语言项目 —— "Rust + Node.js 的 CI 怎么写"

**用户输入：**
> 我的项目有 Rust 后端和 React 前端，CI 怎么写？

**AI 输出（遵循本技能）：**
1. 检测项目根目录：识别 `Cargo.toml` + `package.json`
2. 生成矩阵策略（`strategy.matrix`）或拆分为两个 Job
3. Rust 部分：手动配置 `actions/cache@v4` 缓存 `~/.cargo` 和 `target/`
4. Node.js 部分：`setup-node` 原生支持 `cache: 'npm'`
5. 两个 Job 并行运行，使用不同的 `working-directory`

---

## 6. 检查点

| 检查点编号 | 时机 | 验证内容 | 验证命令 / 预期输出 |
|-----------|------|---------|-------------------|
| CP1 | 骨架生成后 | `on:` 事件是否正确匹配用户需求 | 对照决策树，确认触发事件与用户请求一致 |
| CP2 | YAML 生成后 | 三项必须配置是否齐全 | 在文件中搜索 `permissions:`、`concurrency:`、`# v`（版本注释） |
| CP3 | YAML 生成后 | YAML 语法是否有效 | `yamllint .github/workflows/` 或用 Python `yaml.safe_load()` |
| CP4 | 缓存配置后 | 缓存路径是否与语言生态匹配 | 逐项对照第 3.4 节缓存策略速查表 |
| CP5 | 版本锁定后 | 所有 `uses:` 是否已锁定 SHA | 搜索 `@main`、`@master`、`@v[0-9]$` —— 必须返回空 |

---

## 7. 注意事项

1. **`pull_request_target` 事件是安全雷区。** 该事件在目标仓库上下文中运行，可读取 secrets。除非用户明确要求且理解风险，否则绝不建议使用。优先使用 `pull_request`。

2. **发布流程中 `cancel-in-progress` 必须设为 `false`。** 在 Release Job 的并发配置中，`cancel-in-progress` 必须为 `false`，否则 Tag 推送可能被意外取消。

3. **GitHub 免费 Runner 资源有限。** macOS Runner 消耗 10 倍计费分钟，Windows 消耗 2 倍。优先使用 `ubuntu-latest`。

4. **`GITHUB_TOKEN` 默认权限是 `write`。** 若不显式设置 `permissions`，Token 对仓库具有写权限，存在安全风险。

5. **矩阵策略中设置 `fail-fast: false`。** 多语言/多平台矩阵测试时，设置 `fail-fast: false`，避免一个平台失败导致其他平台被跳过。

6. **工作流文件路径固定。** 所有工作流文件必须放在 `.github/workflows/` 目录下，文件名必须以 `.yml` 或 `.yaml` 结尾。

7. **缓存 Key 的唯一性。** 手动配置 `actions/cache` 时，`key` 必须包含 `hashFiles('lockfile-path')`，否则缓存永远不会失效。

8. **加密变量与机密信息。** 绝不在工作流中硬编码 Token/密码。使用 `${{ secrets.XXX }}` 并在 GitHub Settings -> Secrets 中配置。

---

## 8. 实用 FAQ

### Q1：用户仓库没有 lockfile（如 `package-lock.json`），如何配置缓存？

**回答：** 不配置缓存。`actions/setup-node` 的 `cache` 参数依赖 lockfile 生成缓存 Key。若无 lockfile，跳过缓存配置并提醒用户："建议将 lockfile 提交到仓库以启用依赖缓存。"

### Q2：用户希望工作流在自托管 Runner 上运行，怎么写？

**回答：** 将 `runs-on` 改为 Runner 的标签名（如 `self-hosted`、`linux-arm64`）。同时在输出中提醒：
- 自托管 Runner 需要自行维护运行环境，确保所需依赖已安装
- 建议使用 `runs-on: [self-hosted, linux, x64]` 多标签精确匹配

### Q3：工作流中有多个 Job，如何处理部分 Job 失败的情况？

**回答：**
- 默认行为：某 Job 失败后，所有通过 `needs` 依赖它的后续 Job 会被跳过
- 对于必须"始终运行"的 Job（如清理资源），添加 `if: always()`，但 Job 内部的 Step 需自行检查前置条件

---

## 9. 安全红线

**以下行为绝对禁止，违反任何一条即视为技能执行失败：**

| 红线编号 | 禁止行为 | 说明 |
|---------|---------|------|
| 1 | 在 YAML 中硬编码 Token / API Key / 密码 | 示例 Token 绝不可作为真实配置出现在输出中 |
| 2 | 在用户未充分理解风险时推荐 `pull_request_target` | 该事件是已知的安全漏洞高发区 |
| 3 | 未经用户确认即删除或覆盖已有工作流 | 删除操作必须先展示 diff，等待用户确认后方可执行 |
| 4 | 引用 `@main` / `@master` 版本的 Action | 始终使用 SHA-1 哈希锁定版本 |
| 5 | 设置 `permissions: write-all` | 始终声明具体权限，禁止通配写权限 |

---

## 10. 验证清单

### 功能验证

```
[ ] 1. 生成的 YAML 包含 permissions（Job 级别）
[ ] 2. 生成的 YAML 包含 concurrency 配置
[ ] 3. 所有 uses: 引用均包含 # vX.Y.Z 版本注释
[ ] 4. 缓存配置与项目语言/包管理工具匹配
[ ] 5. on: 触发事件与用户请求场景一致
[ ] 6. 无硬编码的 secrets、Token 或密码
[ ] 7. runs-on 默认为 ubuntu-latest（除非有特殊需求）
```

### 运行时验证（可直接复制执行）

```bash
# 1. YAML 语法校验
yamllint .github/workflows/*.yml

# 2. 若未安装 yamllint，使用 Python 替代
python3 -c "
import yaml, sys, pathlib
for f in pathlib.Path('.github/workflows').glob('*.yml'):
    try:
        yaml.safe_load(f.read_text())
        print(f'{f}: OK')
    except Exception as e:
        print(f'{f}: FAIL -- {e}')
        sys.exit(1)
"

# 3. 安全关键字扫描（检查是否有泄露的密钥）
rg -n 'sk-[a-zA-Z0-9]{20,}' .github/workflows/       # OpenAI Key
rg -n 'ghp_[a-zA-Z0-9]{36}' .github/workflows/       # GitHub PAT
rg -n 'AKIA[a-zA-Z0-9]{16}' .github/workflows/       # AWS Access Key

# 4. 检查浮动版本引用
rg -n '@main|@master' .github/workflows/              # 应返回空
rg -n 'uses:\s+\S+@v\d+$' .github/workflows/          # 应返回空（必须锁定 SHA）

# 5. 检查缺失的必须配置项
echo "=== 检查 permissions ===" && rg -c 'permissions:' .github/workflows/
echo "=== 检查 concurrency ===" && rg -c 'concurrency:' .github/workflows/
```
