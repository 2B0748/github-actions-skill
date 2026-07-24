# 贡献指南 / Contributing

欢迎贡献！本仓库的目标是打造**最全面、最严谨的 GitHub Actions AI Skill**。

Welcome! The goal of this repository is to build the **most comprehensive and rigorous GitHub Actions AI Skill**.

## 贡献方式 / Ways to Contribute

| 方式 / Method | 说明 / Description |
|------|------|
| 🐛 **提交 Issue / Submit Issue** | 报告 Bug、建议新模板、请求新语言支持 / Report bugs, suggest new templates, request new language support |
| ✨ **提交 PR / Submit PR** | 新增模板、优化决策树、补充 FAQ、修复错误 / Add templates, optimize decision tree, supplement FAQ, fix errors |
| 🌍 **翻译 / Translation** | 将 `PROMPT.md` 翻译到其他语言 / Translate `PROMPT.md` into other languages |
| 📝 **示例 / Examples** | 向 `examples/` 提交真实场景的工作流 / Submit real-world workflow examples to `examples/` |

## Skill 开发规范 / Skill Development Standards

提交到 `SKILL.md` 和 `PROMPT.md` 的修改，必须遵循 **12 项编写原则**：

Modifications to `SKILL.md` and `PROMPT.md` must follow the **12 writing principles**:

1. 决策树驱动（新场景 → 新分支） / Decision-tree driven (new scenario → new branch)
2. 祈使句指令 / Imperative instructions
3. Before/After 对比 / Before/After comparison
4. Few-Shot 覆盖（常见 + 边界） / Few-Shot coverage (common + edge cases)
5. 检查点验证 / Checkpoint verification
6. 安全红线不破 / Security red lines unbroken

## PR 流程 / PR Process

1. Fork 仓库 / Fork the repository
2. 创建分支 (`feat/xxx` 或 `fix/xxx`) / Create a branch (`feat/xxx` or `fix/xxx`)
3. 修改后对照 [验证清单 / Verification Checklist](SKILL.md#10-验证清单) 自检 / Self-check against the checklist after changes
4. 提交 PR，附上变更说明 / Submit PR with change description

## 许可 / License

MIT — 贡献即同意以 MIT 许可发布。 / MIT — contributing means you agree to release under the MIT license.
