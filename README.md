# GitHub Actions Workflow Skill

> AI-powered GitHub Actions workflow generation, review & optimization — 生成、审查与优化 GitHub Actions 工作流的 AI 技能

---

## What is this? / 这是什么？

A Claude Code Skill that turns AI into a GitHub Actions expert — generating production-ready CI/CD pipelines, reviewing security posture, and optimizing workflow performance.

一个 Claude Code 技能，将 AI 变成 GitHub Actions 专家 —— 生成生产级 CI/CD 流水线、审查安全配置、优化工作流性能。

## Quick Start / 快速开始

```bash
# Clone this repo
git clone https://github.com/2B0748/github-actions-skill.git

# Copy the skill to your Claude Code skills directory
cp github-actions-skill/SKILL.md .claude/skills/github-actions.md
```

## What it does / 能做什么

| Feature / 功能 | Description / 说明 |
|---|---|
| **CI/CD Generation** / 流水线生成 | Auto-generate CI, Release, Security & Scheduled workflows |
| **Security Audit** / 安全审查 | Check permissions, action pinning, secret handling |
| **Performance Tuning** / 性能优化 | Cache strategy, job parallelization, resource optimization |
| **Best Practices** / 最佳实践 | Least privilege, concurrency control, SHA-pinned actions |

## Triggers / 触发关键词

Mention any of these to activate the skill / 提及以下任一关键词激活技能：

`GitHub Actions` · `workflow` · `CI/CD` · `pipeline` · `YAML` · `actions/cache` · `Dependabot` · `concurrency` · `permissions` · `Release` · `CodeQL`

## Safety / 安全红线

- ❌ Never hardcode API keys or tokens
- ❌ Never suggest `pull_request_target` without explicit user understanding
- ❌ Never use `@main` / `@master` action references
- ✅ Always use `permissions` with least privilege
- ✅ Always pin actions to SHA-1 commit hashes

## License

MIT
