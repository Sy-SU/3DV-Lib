# Backup Graph Cleanup Report

## 问题说明

迁移前备份文件使用 `.md` 扩展名，且包含大量 `[[Papers/...]]` 链接。虽然这些文件只是备份，但 Obsidian 仍会把它们作为 Markdown 节点纳入知识图谱，继续制造管理型边。

## 扫描范围

- `Literature/Workflow/Backups/`
- 扫描备份文件数量：2
- 明确匹配的迁移前 Markdown 备份数量：2
- 其他 `.before-base-migration*.md` 文件数量：0
- 无法自动确认用途的 `.md` 文件数量：0

## 已重命名的备份文件

- `Literature/Workflow/Backups/Literature Index.before-base-migration.md` -> `Literature/Workflow/Backups/Literature Index.before-base-migration.md.bak`
- `Literature/Workflow/Backups/Retrieval Queue.before-base-migration.md` -> `Literature/Workflow/Backups/Retrieval Queue.before-base-migration.md.bak`

## 未修改的有效管理文件

- `Literature/Literature Index.md`
- `Literature/Workflow/Retrieval Queue.md`
- `Literature/Literature Dashboard.base`
- `Literature/Workflow/Retrieval Queue.base`
- `Literature/Workflow/Retrieval Log.md`

## 文件名冲突

- 无

## 无法自动确认的文件

- 无

## 图谱影响

- 重命名前仍作为 `.md` 存在的迁移备份数量：2
- 重命名后的 `.md.bak` 文件数量：2
- 被移出 Obsidian Markdown 索引范围的备份数量：2
- 备份文件中包含的 `[[Papers/...]]` 链接总数：221
- 是否仍存在迁移前 `.md` 备份：否
- 内容相同的重复备份组数量：0

## 安全检查

- 是否删除备份文件：否
- 是否覆盖备份文件：否
- 是否保留全部备份内容：是
- 是否修改当前有效 Literature Index.md：否
- 是否修改当前有效 Retrieval Queue.md：否
- 是否修改 .base 文件：否
- 是否修改 Papers 文件：否
- 是否修改 Topics 文件：否
- 是否修改 Retrieval Log：否
- 是否访问网络：否
- 是否修改论文总结：否

## AGENTS.md

- 是否更新备份约定：是
