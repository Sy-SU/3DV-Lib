# Literature

这是从 `文献列表.md` 初始化得到的 Obsidian Markdown 文献库，用于长期维护论文元数据、阅读状态、主题索引和后续总结。

## 目录结构

- `Papers/`：每篇论文一个 Markdown 主文件。
- `Topics/`：按主题维护阅读路线和文献导航。
- `Templates/Paper Template.md`：新建论文页时使用的模板。
- `Surveys/`：后续保存跨论文总结。
- `Literature Index.md`：总索引。
- `unresolved.md`：记录无法唯一识别或需要人工确认的条目。

## 状态字段

- `unread`：尚未处理。
- `reading`：正在阅读。
- `skimmed`：已粗读，了解大致内容。
- `needs-review`：读过，但需要重新检查部分内容。
- `fully-summarized`：已经完整阅读并完成结构化总结。
- `reference-only`：暂时只作为背景文献保留。

## 总结级别

- `none`：尚未形成总结。
- `abstract`：仅完成摘要级总结。
- `full`：已经形成完整总结。

## 材料级别

- `none`：尚未获得可用材料。
- `abstract`：仅获得摘要。
- `fulltext`：已获得论文全文。
- `fulltext-and-code`：已获得论文全文与代码仓库。

## 新增论文方法

1. 复制 `Templates/Paper Template.md` 到 `Papers/`。
2. 使用稳定文件名，优先使用原始资料中明确出现的简称。
3. 填写能够从原始来源可靠确认的元数据。
4. 在相关 `Topics/` 文件和 `Literature Index.md` 中添加 Obsidian 链接。

## 后续约束

- 不允许根据摘要推断全文细节。
- 不允许在没有原始来源时填写实验数字。
- 不允许根据常识补全作者、年份、会议、DOI、代码仓库或实验结论。
- 每篇论文只保留一个主文件；重复主题只增加索引链接和 topic 标签。
- 结构化总结应写入对应论文页的阅读记录区，不要写入总索引。

## Workflow and Retrieval Preparation

- `Literature/Workflow/Retrieval Queue.md` 用于管理待检索论文。
- `Literature/Workflow/Retrieval Log.md` 用于记录后续联网处理批次。
- `.base` 文件是 Obsidian Bases 视图配置，不是论文笔记。
- 论文笔记状态与总结级别分别管理。
- 只有完成完整结构化总结后，才能设置：

```yaml
status: fully-summarized
summary_level: full
```
