# 3DV-Lib

`3DV-Lib` 是一个独立的 Obsidian 文献管理 Vault，用于维护 3D Vision、生成模型、扩散模型、三维场景生成、Mesh Generation 等方向的论文笔记。

## 目录说明

- `AGENTS.md`：Codex 工作规则。
- `文献列表.md`：原始文献列表。
- `Literature/Papers/`：单篇论文笔记。
- `Literature/Topics/`：专题导航与阅读路线。
- `Literature/Surveys/`：跨论文总结。
- `Literature/Templates/`：论文笔记模板。
- `Literature/Workflow/`：检索队列、日志和审计报告。
- `Literature/PDFs/`：本地论文 PDF，不提交到 Git。
- `.obsidian/`：Obsidian 配置，完整同步到 Git。

## Git 同步规则

后续将整个 `3DV-Lib/` 作为独立 Git 仓库同步到 GitHub。

以下内容不提交：

- `Literature/PDFs/`
- `Literature/Workflow/Backups/`
- `.DS_Store`
- `Thumbs.db`
- `.trash/`
- 临时文件

`.obsidian/` 需要完整提交。
