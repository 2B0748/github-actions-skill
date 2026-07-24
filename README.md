# GitHub Actions Workflow Skill

> 🤖 AI-powered GitHub Actions workflow generation, review & optimization
>
> 🤖 AI 驱动的 GitHub Actions 工作流生成、审查与优化 —— 全平台通用

---

## 简介 / About

将 AI 变成 GitHub Actions 专家：生成生产级 CI/CD 流水线、审计工作流安全、优化缓存与并行化、强制执行最佳实践。**内置 12 道工程质量关卡，5 套决策模板，3 项必含配置零遗漏。**

Transforms any AI agent into a GitHub Actions expert — production-ready pipelines, security audits, caching & parallel optimization, and enforced best practices.

## 全平台支持 / Universal

| 平台 | 文件 | 用法 |
|------|------|------|
| **Claude Code** | `SKILL.md` | 复制到 `.claude/skills/` 目录 |
| **Cursor** | `PROMPT.md` | 复制到 `.cursor/rules/` 或粘贴到 Rules |
| **GitHub Copilot** | `PROMPT.md` | 重命名为 `.github/copilot-instructions.md` |
| **Windsurf / Aider / Cline / CodeBuddy** | `PROMPT.md` | 粘贴到 System Prompt 或 Rules 配置 |
| **任意 Agent** | `PROMPT.md` | 直接作为自定义指令 / Custom Instructions |

## 快速开始 / Quick Start

```bash
git clone https://github.com/2B0748/github-actions-skill.git

# Claude Code
cp github-actions-skill/SKILL.md .claude/skills/github-actions.md

# Cursor
cp github-actions-skill/PROMPT.md .cursor/rules/github-actions.md

# GitHub Copilot
cp github-actions-skill/PROMPT.md .github/copilot-instructions.md
```

触发关键词 / Triggers：`GitHub Actions` · `CI/CD` · `workflow` · `pipeline` · `Release` · `CodeQL`

## 能力矩阵 / Capabilities

| 能力 | 说明 |
|------|------|
| 🔧 CI 生成 | 自动生成 Lint → Test → Build 完整流水线 |
| 📦 自动发布 | Tag 触发自动构建、打包、生成 Release |
| 🛡️ 安全审查 | 权限检查、SHA-1 版本锁定、密钥泄露扫描 |
| ⚡ 性能优化 | 缓存策略、并行 Job 拆分、Runner 选型 |
| 📋 最佳实践 | 最小权限、并发控制、12 项验证清单 |

## 文件结构 / Files

| 文件 | 说明 |
|------|------|
| `SKILL.md` | Claude Code 格式（含 YAML 元数据） |
| `PROMPT.md` | 通用版（全平台兼容，纯 Markdown） |
| `README.md` | 本文件 |

## 许可 / License

MIT
