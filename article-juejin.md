# 我写了一个 AI Skill，让任何 AI 编程助手都能帮你写 GitHub Actions

> 别再手写 YAML 了。把 12 道工程质量关卡、5 套决策模板、3 项必含配置交给 AI。

---

## 痛点：GitHub Actions 写一次，改十次

你有没有经历过：

- PR 提上去才发现 CI 没跑缓存，npm install 每次跑 5 分钟
- deploy 到一半被新的 push 取消，留下一地鸡毛
- 用 `actions/checkout@v4` 写了半年，不知道这其实是供应链安全隐患
- 想加 CodeQL 安全扫描，但 YAML 配置项太多，抄都抄不对

**这些都不是你的问题，是 YAML 配置太容易出错了。** GitHub Actions 有 200+ 配置项，权限模型复杂，Runner 计费规则晦涩。大多数开发者靠"复制粘贴 + 试错"来写 workflow，效率极低。

## 解法：把经验封装成 Skill，让 AI 替你记住所有规则

我花了很长时间整理了一套 AI Skill —— 本质上是一份**可执行的指令集**，不是长提示词。它告诉 AI：

> "生成 Workflow 时，你必须先跑决策树匹配模板 → 注入三项必含配置 → 按语言匹配缓存策略 → 过 12 道质量关卡 → 输出带验证清单的 YAML。"

### 把它丢给任意 AI 编程助手（Claude Code / Cursor / Copilot / Windsurf），你只需要说一句话：

> "帮我给 Node.js 项目加 CI，PR 跑 eslint 和 jest"

AI 会自动：

1. 检测 `package.json` → 确定 Node.js 项目
2. 匹配模板 A（CI）
3. 注入 `permissions: contents: read`（最小权限）
4. 注入 `concurrency`（自动取消旧运行）
5. 配置 `cache: 'npm'`（节省 70% 安装时间）
6. 所有 Action 锁定到 SHA-1 哈希（防供应链攻击）
7. 输出前过 12 道验证清单

**5 秒出 YAML，零配置项遗漏。**

## 和其他方案的区别

| 方案 | 通用性 | 安全审查 | 缓存策略 | 质量关卡 |
|------|--------|---------|---------|---------|
| GitHub Actions 官方模板 | ✅ | ❌ | ❌ | ❌ |
| 自己手写 | ✅ | ❌ | ❌ | ❌ |
| ChatGPT 裸聊 | ✅ | ❌ | ❌ | ❌ |
| **本 Skill** | ✅ 全平台 | ✅ 5 红线 | ✅ 6 语言 | ✅ 12 关卡 |

ChatGPT 裸聊的问题是：你每次都要重新描述需求，它每次都可能遗漏 `permissions`、忘记缓存、不锁 Action 版本。而 Skill 把这些规则**执行化**了——AI 不是"建议"，而是"必须"。

## 决策引擎：5 条路径覆盖 95% 场景

```
用户请求 → 关键词匹配
├─ "PR / lint / 测试"        → CI Workflow
├─ "Tag / Release / 发布"    → Release Workflow
├─ "cron / 定期 / 依赖"      → Scheduled Workflow
├─ "安全 / 扫描 / CodeQL"    → Security Workflow
└─ "Issue / PR 自动 / 标签"  → Automation Workflow
```

每个模板都内置了 **权限最小化、并发控制、SHA-1 锁定** 三项强制配置。

## 安全意识：5 条红线自动拦截

Skill 内置的高频安全风险自动拦截：

1. ❌ 硬编码 Token/API Key → 拦截
2. ❌ 推荐 `pull_request_target` → 拦截（高危事件）
3. ❌ 覆盖已有 workflow 无确认 → 拦截
4. ❌ 引用 `@main` / `@master` → 拦截
5. ❌ `permissions: write-all` → 拦截

## 实际效果

```yaml
# ❌ 没有 Skill 时的典型输出（ChatGPT 裸聊）
name: CI
on: push
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4    # ← 没锁版本
      - run: npm install               # ← 没缓存
      - run: npm test
      # 没有 permissions、没有 concurrency

# ✅ 用 Skill 后的输出
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
      - uses: actions/setup-node@cd063736c72066a58f36dd95fb0460484b220fb7 # v4.3.0
        with:
          node-version: 20
          cache: 'npm'
      - run: npm ci
      - run: npx eslint .
  # ... 完整的 lint → test 流水线
```

## 全平台通用

不只是 Claude Code —— 任何一个 AI 编程助手都能用：

- **Claude Code**：复制 `SKILL.md` → `.claude/skills/`
- **Cursor**：复制 `PROMPT.md` → `.cursor/rules/`
- **GitHub Copilot**：复制 `PROMPT.md` → `.github/copilot-instructions.md`
- **Windsurf / Aider / Cline**：粘贴到 System Prompt
- **国际用户**：英文版 `PROMPT.en.md`

## 最后

仓库已开源在 GitHub：

👉 https://github.com/2B0748/github-actions-skill

如果你觉得有用，**给个 Star ⭐**。如果发现模板不够用或有新的语言需求，欢迎提 Issue 或 PR。
