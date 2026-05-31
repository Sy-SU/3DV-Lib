# Graph Hygiene Migration Report

## 问题说明

`Literature/Literature Index.md` 和 `Literature/Workflow/Retrieval Queue.md` 原本包含大量指向 `Literature/Papers/` 的批量 Obsidian 链接，导致管理文件在知识图谱中形成高度数枢纽。本次迁移将批量管理视图迁移到 Obsidian Bases，并保留 Topic 中人工整理的阅读路线。

## 备份文件

- `Literature/Workflow/Backups/Literature Index.before-base-migration.md`
- `Literature/Workflow/Backups/Retrieval Queue.before-base-migration.md`

## `.base` 文件检查结果

- `未命名.base`: 仅包含空表格骨架，保留，未复用。
- `Literature Dashboard.base`: 仅包含空表格骨架，路径在 vault 根目录，保留，未复用。

## 新建或复用的 Base 文件

- 新建：`Literature/Literature Dashboard.base`, `Literature/Workflow/Retrieval Queue.base`
- 复用：无
- 说明：新 Base 文件不手工列出论文链接，只基于 `Literature/Papers/` 文件的 Properties 过滤。

## Literature Index.md 调整

- 已移除批量论文表格。
- 保留 Topics 导航链接。
- 添加 `[[Literature Dashboard.base]]` 作为全部文献、状态筛选和阅读进度入口。

## Retrieval Queue.md 调整

- 已移除批量论文队列表格。
- 保留每批 5 篇、优先级、状态顺序、跳过 `summary_level: full` 和批次结束规则。
- 添加 `[[Retrieval Queue.base]]` 作为可视化队列入口。

## AGENTS.md 调整

- 已添加 Graph hygiene and management views 规则。
- 已添加 Retrieval queue selection 规则，后续批次改为扫描 `Literature/Papers/*.md` YAML Front Matter。

## Topics 文件检查

- `Literature/Topics/3D Scene Generation.md`: Papers 链接 106，阅读顺序：有，人工备注线索：有
- `Literature/Topics/3D Vision.md`: Papers 链接 6，阅读顺序：有，人工备注线索：有
- `Literature/Topics/Diffusion Model and Text-to-Image.md`: Papers 链接 34，阅读顺序：有，人工备注线索：有
- `Literature/Topics/Image-based Scene Generation.md`: Papers 链接 26，阅读顺序：有，人工备注线索：有
- `Literature/Topics/Interaction in 3D Generation.md`: Papers 链接 10，阅读顺序：有，人工备注线索：有
- `Literature/Topics/Mesh Generation.md`: Papers 链接 10，阅读顺序：有，人工备注线索：有
- `Literature/Topics/Neural 3D-based Scene Generation.md`: Papers 链接 36，阅读顺序：有，人工备注线索：有
- `Literature/Topics/Procedural Generation.md`: Papers 链接 12，阅读顺序：有，人工备注线索：有
- `Literature/Topics/Text-to-3D.md`: Papers 链接 6，阅读顺序：有，人工备注线索：有
- `Literature/Topics/Text-to-Human-Motion.md`: Papers 链接 4，阅读顺序：有，人工备注线索：有
- `Literature/Topics/Transformer and Recent Works.md`: Papers 链接 6，阅读顺序：有，人工备注线索：有
- `Literature/Topics/Video-based Scene Generation.md`: Papers 链接 14，阅读顺序：有，人工备注线索：有

未自动修改任何 Topic 文件。若后续希望进一步降低图谱噪声，可人工审查是否有 Topic 文件只是自动生成完整清单。

## 图谱边减少情况

- 迁移前 `Literature/Literature Index.md` 中指向 Papers/ 的链接数量：135
- 迁移后 `Literature/Literature Index.md` 中指向 Papers/ 的链接数量：0
- 迁移前 `Literature/Workflow/Retrieval Queue.md` 中指向 Papers/ 的链接数量：86
- 迁移后 `Literature/Workflow/Retrieval Queue.md` 中指向 Papers/ 的链接数量：0
- 总计移除的管理型链接数量：221
- Topics 文件中保留的语义链接数量：270
- Papers 文件之间保留的语义链接数量：0

## 仍需人工检查的问题

- 现有 `.base` 示例仅包含空表格骨架，无法从项目内确认完整 Bases 过滤/排序语法；已创建 YAML 合法、基于 Properties 的配置，建议在 Obsidian UI 中打开确认。
- 自定义枚举排序语法无法从现有示例确认；Base 中使用属性排序，目标优先级/状态顺序保留在 Queue 说明页和 AGENTS.md。

## 后续建议

- 在 Obsidian 中打开 `Literature/Literature Dashboard.base` 和 `Literature/Workflow/Retrieval Queue.base`，确认当前版本的 Bases 插件是否接受本地 YAML 中的过滤与排序表达式。
- 后续 Codex 检索批次从论文 YAML Properties 选择候选，不再读取 Markdown 队列表格。
- 保持 `Retrieval Log.md` 作为历史记录；若它未来变大，可另行决定是否将历史记录中的论文链接改为纯文本。

## 安全检查

- 是否保留全部论文笔记：是
- 是否保留全部 PDF：是
- 是否保留全部 Topic 文件：是
- 是否保留 Retrieval Log：是
- 是否创建备份：是
- 是否保留原有 .base 文件：是
- 是否存在疑似损坏内部链接：否
- 是否访问网络：否
- 是否下载论文：否
- 是否重新生成论文总结：否
- 是否删除论文文件：否
- 是否修改论文文件名：否
- 是否覆盖用户原始备注：否
- 是否删除 Retrieval Log：否
- 是否直接修改 .obsidian/：否
