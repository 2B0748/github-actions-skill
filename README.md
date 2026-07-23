# GitHub Actions Workflow Skill

> 🤖 AI-powered GitHub Actions workflow generation, review & optimization
>
> 🤖 AI 驱动的 GitHub Actions 工作流生成、审查与优化技能

---

## 简介 / About

**GitHub Actions Workflow Skill** is a Claude Code skill that transforms AI into a GitHub Actions expert. It generates production-ready CI/CD pipelines, audits workflow security, optimizes performance with caching & parallelization, and enforces best practices — all with zero-config defaults and 12 engineering quality gates.

**GitHub Actions Workflow Skill** 是一个 Claude Code 技能，将 AI 变成 GitHub Actions 专家。它能生成生产级 CI/CD 流水线、审查工作流安全性、通过缓存与并行化优化性能，并强制执行最佳实践——内置 12 道工程质量关卡。

## 快速开始 / Quick Start

```bash
git clone https://github.com/2B0748/github-actions-skill.git
cp github-actions-skill/SKILL.md .claude/skills/github-actions.md
```

Then just mention: / 然后只需提及：

`GitHub Actions` · `CI/CD` · `workflow` · `pipeline` · `Release` · `CodeQL`

## 能力矩阵 / Capabilities

| 能力 | Capability | 说明 |
|------|-----------|------|
| 🔧 CI 生成 | CI Generation | 自动生成 Lint → Test → Build 完整流水线 |
| 📦 自动发布 | Release Automation | Tag 触发自动构建、打包、生成 Release |
| 🛡️ 安全审查 | Security Audit | 权限检查、版本锁定、密钥泄露扫描 |
| ⚡ 性能优化 | Performance Tuning | 缓存策略、并行 Job 拆分、Runner 选型 |
| 📋 最佳实践 | Best Practices | 最小权限、并发控制、SHA-1 锁定 |

## 工作原理 / How It Works

```
用户请求 → 决策树(5模板匹配) → 骨架生成 → 必含配置注入
    → 缓存策略适配 → 安全检查 → 输出带验证清单的 YAML
```

## 技能结构 / Skill Structure

- `SKILL.md` — 完整的 AI 可执行指令集（含决策树、模板、FAQ、验证清单）

## 许可 / License

MIT — 自由使用、修改、分发。
