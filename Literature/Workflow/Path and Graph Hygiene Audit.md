# Path and Graph Hygiene Audit

## 当前项目根目录

```text
/Users/susenyang/Documents/Obsidian Vault/3DV-Lib
```

本次审计只读取和修改该目录内部文件。未访问网络，未下载论文，未初始化 Git，未修改父 Vault。

## 扫描范围

- `AGENTS.md`
- `README.md`
- `文献列表.md`
- `Literature/**/*.md`
- `Literature/**/*.base`
- `.obsidian/**/*`

扫描文件数量：134。

## 旧路径检查

发现与路径规则相关的绝对路径记录：

| 文件 | 行号 | 路径 | 处理 |
|---|---:|---|---|
| `AGENTS.md` | 225 | `/Users/susenyang/Documents/Obsidian Vault/3DV-Lib` | 当前项目根目录规则，保留 |
| `AGENTS.md` | 232 | `/Users/susenyang/Documents/Obsidian Vault` | 父 Vault 禁止修改规则，保留 |
| `Literature/Workflow/3DV-Lib Migration Report.md` | 11 | `/Users/susenyang/Documents/Obsidian Vault` | 历史迁移记录，不影响当前运行，保留 |
| `Literature/Workflow/3DV-Lib Migration Report.md` | 15 | `/Users/susenyang/Documents/3DV-Lib` | 历史迁移记录，不影响当前运行，保留 |
| `Literature/Workflow/Audit Report.md` | 5 | `/Users/susenyang/Documents/Obsidian Vault/3DV-Lib` | 当前项目根目录的历史审计记录，保留 |

旧迁移路径 `/Users/susenyang/Documents/3DV-Lib` 仅出现在历史迁移报告中，未作为当前可执行路径使用。

## YAML 路径检查

扫描范围：`Literature/Papers/*.md`。

检查字段：

- `pdf_path`
- `url`
- `doi`
- `arxiv`
- `code_url`
- `retrieved_from`

结果：

- YAML 解析错误：0
- 非法状态字段：0
- 旧绝对路径字段：0
- `pdf_path` 非项目相对路径：0
- 自动修复的 `pdf_path` 数量：0

所有已有 `pdf_path` 均保持仓库内相对路径格式，例如 `Literature/PDFs/DDPM.pdf`。

## `.base` 文件检查

扫描到 `.base` 文件：

- `Literature/Literature Dashboard.base`
- `Literature/Workflow/Retrieval Queue.base`

结果：

- YAML 解析错误：0
- 引用旧绝对路径：0
- 使用 `Literature/Papers/` 作为筛选范围：是
- 重复 Dashboard：否
- 重复 Retrieval Queue Base：否
- `未命名.base`：否
- 手工枚举全部论文链接：否
- 自动修复的 `.base` 路径数量：0

状态：可在新 Vault 中测试。

## `.obsidian/` 检查

扫描文件：

- `.obsidian/app.json`
- `.obsidian/appearance.json`
- `.obsidian/core-plugins.json`
- `.obsidian/graph.json`
- `.obsidian/workspace.json`

结果：

- 旧绝对路径：0
- CURRENT_VAULT_ROOT 占位符：0
- TARGET_PARENT_DIR 占位符：0
- 自动修复路径数量：0
- 疑似敏感配置文件：无

未批量重写 `.obsidian/`。

## Literature Index.md 调整

`Literature/Literature Index.md` 已经是导航页，未发现批量论文表格。

- 修复前指向 `Papers/` 的 Wikilink 数量：0
- 修复后指向 `Papers/` 的 Wikilink 数量：0
- 本次自动修改：否

## Retrieval Log.md 调整

`Literature/Workflow/Retrieval Log.md` 是历史日志，不应连接到每篇论文。

本次将日志表格第三列中的 10 个论文 Wikilink 改为普通文本：

- `CFG`
- `DDIM`
- `DDPM`
- `DreamBooth`
- `DreamFusion`
- `LDM`
- `MDM`
- `MotionGPT`
- `NeRF`
- `Null-text Inversion`

保留了日期、批次、来源、PDF 路径、修改文件和待确认问题。

## Retrieval Queue.md 调整

`Literature/Workflow/Retrieval Queue.md` 已经是工作流说明页，未发现批量论文表格。

本次补充执行规则：

- 日志中的论文名称使用普通文本，不使用 Wikilink。

## 备份文件检查

扫描范围：`Literature/Workflow/Backups/`。

结果：

- 仍为 `.md` 的迁移备份：0
- 已保留 `.md.bak` 备份：2
- 本次重命名为 `.md.bak` 的备份文件数量：0
- 删除备份文件：否
- 修改备份内容：否

## AGENTS.md 调整

已追加 `3DV-Lib Path and Graph Hygiene Rules`，记录：

- 当前项目根目录。
- 禁止修改父 Vault。
- YAML 和 Markdown 优先使用项目相对路径。
- `Retrieval Log.md` 中论文名称使用普通文本。
- Index、Queue、Log 不重新引入批量论文链接。
- Topics、Papers、Surveys 中的语义链接保留。

## 图谱边变化

| 项目 | 修复前 | 修复后 |
|---|---:|---:|
| `Literature/Literature Index.md` 指向 `Papers/` 的 Wikilink | 0 | 0 |
| `Literature/Workflow/Retrieval Log.md` 指向 `Papers/` 的 Wikilink | 10 | 0 |
| `Literature/Workflow/Retrieval Queue.md` 指向 `Papers/` 的 Wikilink | 0 | 0 |

其他统计：

- 被重命名为 `.md.bak` 的备份文件数量：0
- Topics 中保留的语义论文链接数量：270
- Papers 中保留的语义 Wikilink 数量：12
- 总计移除的管理型边数量：10

## 自动修复内容

- `Literature/Workflow/Retrieval Log.md`：将 10 个管理型论文 Wikilink 改为普通文本。
- `Literature/Workflow/Retrieval Queue.md`：补充日志使用普通文本的规则。
- `AGENTS.md`：追加路径兼容与图谱卫生规则。
- `Literature/Workflow/Path and Graph Hygiene Audit.md`：创建本审计报告。

## 未自动修复的问题

- `Literature/Workflow/3DV-Lib Migration Report.md` 中保留历史路径 `/Users/susenyang/Documents/Obsidian Vault` 与 `/Users/susenyang/Documents/3DV-Lib`，属于迁移历史记录，不影响当前运行。
- `Literature/Workflow/Audit Report.md` 中保留当前项目根目录绝对路径，属于历史审计记录。
- `AGENTS.md` 中保留当前项目根目录和父 Vault 禁止修改路径，属于工作边界规则。

## 是否建议继续执行联网文献整理

建议继续执行联网文献整理。

理由：

- 论文 YAML 路径检查通过。
- `.base` 文件检查通过。
- `.obsidian/` 未发现旧路径或疑似敏感配置。
- Index、Queue、Log 中未保留管理型批量论文边。
- Topics 和 Papers 中的语义链接已保留。

下一阶段仍应遵守：每批最多 5 篇、只访问当前批次相关来源、日志论文名称使用普通文本、不要向管理页重新写入批量论文链接。
