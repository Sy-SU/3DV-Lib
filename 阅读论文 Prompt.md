请继续执行当前 Obsidian 文献库的联网检索与整理任务。

执行任何操作前，先完整阅读：

text AGENTS.md Literature/Workflow/Retrieval Queue.md Literature/Workflow/Retrieval Log.md 

同时扫描：

text Literature/Papers/*.md 

严格遵守 AGENTS.md 中已有的文献管理规则、知识图谱清理规则、Markdown 表格规则和 Obsidian 数学公式渲染规则。

本次只处理下一批最多 5 篇文献。完成后立即停止，不要自动继续下一批。

# 一、任务目标

不要从 Retrieval Queue.md 中解析批量论文表格。

当前队列由论文笔记中的 YAML Properties 动态决定。请扫描：

text Literature/Papers/*.md 

筛选尚未完成完整总结的论文。

跳过满足以下条件的论文：

yaml summary_level: full 

无论其 source_level 为：

yaml fulltext 

还是：

yaml fulltext-and-code 

均跳过。

候选论文按照以下顺序排序。

第一层：priority

text high medium low 

第二层：status

text needs-review reading skimmed unread reference-only 

第三层：稳定文件名，按字典序排列。

选择排序后的前 5 篇论文。

如果不足 5 篇，则处理剩余论文后停止。

在开始联网前，先在终端列出：

text 本批次候选论文： 1. 2. 3. 4. 5. 

# 二、批次编号

读取：

text Literature/Workflow/Retrieval Log.md 

自动查找现有最大批次编号。

例如，日志中最大编号为：

text Batch 003 

则本次使用：

text Batch 004 

如果尚无历史批次，则使用：

text Batch 001 

不要覆盖已有日志记录。

# 三、允许的联网范围

本次允许访问网络，但仅限于当前批次中的论文。

优先来源：

text arxiv.org openaccess.thecvf.com proceedings.neurips.cc proceedings.mlr.press aclanthology.org openreview.net dl.acm.org ieeexplore.ieee.org link.springer.com sciencedirect.com github.com 论文官方项目主页 作者主页 

来源优先级：

1. 论文官方页面。
2. arXiv 页面和 arXiv PDF。
3. 会议或期刊官方页面。
4. 作者主页。
5. 官方项目主页。
6. 官方 GitHub 仓库。
7. 其他可信公开来源。

禁止：

1. 不要绕过付费墙。
2. 不要访问需要登录的页面。
3. 不要从不明来源下载 PDF。
4. 不要将博客、社交媒体、搜索结果摘要或自动生成的二次解读作为论文全文依据。
5. 不要搜索与当前批次无关的论文。
6. 不要自动扩大当前批次。
7. 不要在当前批次结束后继续处理下一批。
8. 不要执行训练代码。
9. 不要安装依赖。
10. 不要下载模型权重。
11. 不要克隆体积较大的仓库。

# 四、本地目录规则

论文笔记目录：

text Literature/Papers/ 

PDF 保存目录：

text Literature/PDFs/ 

工作流文件：

text Literature/Workflow/Retrieval Queue.md Literature/Workflow/Retrieval Queue.base Literature/Workflow/Retrieval Log.md Literature/Workflow/Suggested Papers.md 

问题记录文件：

text Literature/unresolved.md 

数据库视图：

text Literature/Literature Dashboard.base Literature/Workflow/Retrieval Queue.base 

每篇论文只能有一个主笔记文件：

text Literature/Papers/<稳定文件名>.md 

每篇论文最多对应一个 PDF：

text Literature/PDFs/<稳定文件名>.pdf 

PDF 文件名必须与论文 Markdown 文件名一致。

示例：

text Literature/Papers/DDIM.md Literature/PDFs/DDIM.pdf 

不要擅自重命名已有文件。

不要修改：

text Literature/Literature Dashboard.base Literature/Workflow/Retrieval Queue.base 

除非发现明确语法错误。正常情况下，Base 文件会自动从 YAML Properties 中读取状态，无需手工更新。

# 五、逐篇处理流程

对本批次中的每篇论文，严格按照以下顺序执行。

## Step 1：读取已有笔记

打开：

text Literature/Papers/<论文文件>.md 

读取 YAML Front Matter：

yaml title: short_name: authors: year: venue: url: doi: arxiv: code_url: topics: paper_type: task: status: summary_level: source_level: priority: pdf_path: retrieved_from: retrieval_date: read_version: 

同时读取正文中的：

markdown ## 用户原始备注  ## 待确认问题  ## 阅读记录  ### 摘要总结  ### 粗读记录  ### 完整总结 

要求：

1. 保留已有人工内容。
2. 不删除用户原始备注。
3. 不覆盖已有阅读记录。
4. 可以追加内容，但不要静默替换用户手工撰写的内容。
5. 如果新信息与已有信息冲突，将差异写入 ## 待确认问题。

## Step 2：确认论文身份

联网搜索后，至少确认：

1. 标题是否匹配。
2. 简称是否匹配。
3. arXiv 编号是否匹配。
4. DOI 是否匹配。
5. 作者是否与标题一致。
6. 是否存在同名论文。
7. 是否存在预印本和正式发表版本。
8. 是否已有本地 PDF。
9. 是否已有官方代码仓库。
10. 是否已有重复论文笔记。

如果无法唯一识别：

1. 不下载 PDF。
2. 不生成总结。
3. 将问题记录到：
   text    Literature/unresolved.md    
4. 在检索日志中标记：
   text    unresolved    
5. 继续处理下一篇。

## Step 3：获取全文

优先检查本地文件：

text Literature/PDFs/<稳定文件名>.pdf 

如果本地 PDF 已存在：

1. 检查是否能够正常读取。
2. 优先使用本地 PDF。
3. 不重复下载。

如果本地不存在：

1. 优先从论文官方页面或 arXiv 获取公开 PDF。
2. 保存到：
   text    Literature/PDFs/<稳定文件名>.pdf    
3. 检查 PDF 是否能够正常读取。
4. 记录实际下载来源。
5. 不创建重复 PDF。

如果只能获得摘要：

1. 不伪造全文总结。
2. 只生成摘要级总结。
3. 记录无法获取全文的原因。

## Step 4：检查官方代码仓库

如果论文存在官方 GitHub 仓库、官方项目主页或作者公开代码：

1. 记录链接。
2. 读取 README 或项目说明。
3. 只记录能够确认的信息。
4. 不执行代码。
5. 不安装依赖。
6. 不下载模型权重。
7. 不克隆大型仓库。

如果无法确认仓库是否官方，不填写 code_url。

## Step 5：按材料级别生成总结

根据实际获得的材料决定总结级别。

# 六、全文总结规则

只有在实际获得并读取完整 PDF 后，才允许设置：

yaml source_level: fulltext summary_level: full status: fully-summarized 

如果同时确认并阅读了官方代码仓库 README 或项目文档，可以设置：

yaml source_level: fulltext-and-code 

完整总结至少填写：

markdown ## 基本信息  ## 一句话概括  ## 研究问题  ## 方法概述  ## 关键公式  ## 实验设置  ## 主要结果  ## 论文贡献  ## 局限性  ## 与其他论文的关系  ## 与当前研究方向的关系  ## 待确认问题  ## 阅读记录  ### 摘要总结  ### 粗读记录  ### 完整总结  ## 引用信息 

## 基本信息

记录：

text 简称 作者 发表年份 会议或期刊 原文链接 arXiv 链接 DOI 官方代码仓库 本地 PDF 路径 所属主题 实际阅读版本 

无法确认的信息保持为空，不要猜测。

## 一句话概括

用 2 至 4 句话说明：

1. 论文解决的问题。
2. 提出的主要方法。
3. 最主要的作用或结果。

## 研究问题

说明：

1. 任务背景。
2. 已有方法的限制。
3. 作者处理的具体问题。

## 方法概述

说明：

1. 输入与输出。
2. 整体框架。
3. 核心模块。
4. 训练流程。
5. 推理流程。
6. 与已有工作的差异。

不要只翻译摘要。

## 关键公式

仅记录理解论文所必需的公式。

每个公式需要：

1. 说明公式含义。
2. 解释符号。
3. 说明公式在方法中的作用。
4. 尽量注明论文公式编号。

数学公式使用 Obsidian 兼容格式。

短公式：

markdown $t$ 

长公式：

markdown $$ ... $$ 

禁止使用：

markdown \( ... \) \[ ... \] 

条件概率优先使用：

latex \mid 

高斯分布优先使用：

latex \mathcal{N} 

数学期望优先使用：

latex \mathbb{E} 

多字母文本下标使用：

latex L_{\mathrm{simple}} 

除非原论文中的符号另有含义。

## 实验设置

记录：

1. 数据集。
2. 评价指标。
3. 对比方法。
4. 训练设置。
5. 消融实验。

无法确认的内容不填写。

## 主要结果

仅记录重要结果。

每个数值注明：

1. 数据集。
2. 指标。
3. 实验条件。
4. 表格编号或章节位置。

不要脱离上下文堆砌数字。

## 论文贡献

使用编号列表总结。

## 局限性

区分：

text 作者明确说明的局限性 个人分析 

个人分析必须写明：

text 个人分析： 

不要把个人判断写成作者结论。

## 与其他论文的关系

记录：

text 前置工作 后续工作 相似方法 主要区别 

只记录可以确认的信息。

## 与当前研究方向的关系

结合论文所属 Topic，说明其用途，例如：

text 扩散模型理论基础 文生图方法基础 文生 3D 的关键方法 Mesh Tokenization 相关方法 3D 场景生成方法 物理交互建模方法 

不要写空泛评价。

# 七、摘要级总结规则

如果只能获得可信摘要，设置：

yaml source_level: abstract summary_level: abstract status: skimmed 

只允许填写：

markdown ## 基本信息  ## 一句话概括  ## 阅读记录  ### 摘要总结  ## 待确认问题 

以下章节保持原样：

markdown ## 方法概述  > 尚未总结。  ## 关键公式  > 尚未总结。  ## 实验设置  > 尚未总结。  ## 主要结果  > 尚未总结。  ## 局限性  > 尚未总结。 

不要根据摘要推断正文细节。

# 八、无法处理时的规则

出现以下情况时：

text 论文身份无法确认 全文无法公开获得 PDF 链接失效 PDF 无法读取 存在多个版本且无法判断 官方页面无法访问 代码仓库身份无法确认 

执行：

1. 保留已有内容。
2. 在：
   text    Literature/unresolved.md    
   中记录问题。
3. 在：
   text    Literature/Workflow/Retrieval Log.md    
   中记录原因。
4. 继续处理下一篇。
5. 不要中止整个批次。
6. 不要自行猜测。

# 九、更新 YAML Front Matter

对于成功处理的论文，更新对应的：

text Literature/Papers/<稳定文件名>.md 

填写或更新：

yaml authors: year: venue: url: doi: arxiv: code_url: topics: paper_type: task: status: summary_level: source_level: priority: pdf_path: retrieved_from: retrieval_date: read_version: 

缺失字段可以补充。

示例：

yaml pdf_path: "Literature/PDFs/DDPM.pdf" retrieved_from:   - "https://arxiv.org/abs/2006.11239"   - "https://arxiv.org/pdf/2006.11239" retrieval_date: "YYYY-MM-DD" read_version: "arXiv" 

要求：

1. 只记录实际访问过的来源。
2. 不删除已有字段。
3. 保持 YAML 合法。
4. 不修改稳定文件名。
5. 不覆盖正确的人工内容。
6. 如果新旧信息冲突，写入 ## 待确认问题。

# 十、队列更新规则

不要向：

text Literature/Workflow/Retrieval Queue.md 

写入批量论文表格。

该文件现在只保留工作流说明。

不要手工修改：

text Literature/Workflow/Retrieval Queue.base 

论文状态写回各自 YAML 后，Base 会自动更新。

每篇论文处理完成后，只需确认：

1. YAML 状态已经更新。
2. Retrieval Queue.base 可以通过 Properties 自动反映新状态。
3. summary_level: full 的论文会自动退出待处理视图。
4. 未完成论文仍然保留在待处理视图中。

# 十一、日志更新规则

完成每篇论文后，在：

text Literature/Workflow/Retrieval Log.md 

新增一行。

至少记录：

markdown | 日期 | 批次 | 论文 | 使用来源 | 是否获得全文 | 是否获得代码 | 生成总结级别 | PDF 路径 | 修改文件 | 待确认问题 | 

要求：

1. 记录实际访问来源。
2. 记录是否获得 PDF。
3. 记录 PDF 保存路径。
4. 记录是否确认官方代码仓库。
5. 记录生成的是摘要总结还是完整总结。
6. 记录修改过的文件。
7. 记录未解决问题。
8. 记录当前批次编号。
9. 不覆盖已有日志。
10. Markdown 表格中的 Obsidian 链接使用转义形式：
   markdown    [[../Papers/DDPM\|DDPM]]    

# 十二、索引和 Topic 文件规则

不要向：

text Literature/Literature Index.md 

重新写入批量论文状态表格。

不要手工修改：

text Literature/Literature Dashboard.base 

状态变化会自动从论文 YAML 中读取。

对于：

text Literature/Topics/*.md 

执行以下规则：

1. 保留人工整理的阅读顺序。
2. 保留有语义的论文链接。
3. 保留用户原始备注。
4. 不批量写入所有论文状态。
5. 不创建无意义的管理型链接。
6. 如果 Topic 文件中已经存在论文状态表格，只更新本批次对应行。
7. 如果 Topic 文件中没有状态表格，不要新增。
8. 表格内的 Obsidian 链接别名分隔符必须使用：
   markdown    \|    

# 十三、推荐文献规则

阅读本批次论文时，如果发现值得补充的新论文：

1. 不要自动创建论文笔记。
2. 不要自动加入待处理队列。
3. 不要自动下载。
4. 只记录到：

text Literature/Workflow/Suggested Papers.md 

如果文件不存在，创建：

markdown # Suggested Papers  | 论文 | 推荐原因 | 来源论文 | 链接 | 是否加入队列 | |---|---|---|---|---| 

最后一列填写：

text 待用户确认 

# 十四、批次结束后的检查

完成本批次后，执行以下检查。

## 论文文件

1. 是否创建重复论文笔记。
2. 是否下载重复 PDF。
3. PDF 是否能够读取。
4. YAML 是否合法。
5. 是否误删用户备注。
6. 是否误改稳定文件名。
7. 是否存在空白论文文件。

## 总结级别

1. summary_level: full 是否都有全文来源。
2. source_level: fulltext 是否都有本地 PDF。
3. source_level: fulltext-and-code 是否确实阅读过官方仓库说明。
4. source_level: abstract 是否没有伪造正文总结。
5. status: fully-summarized 是否存在结构化完整总结。
6. 是否存在非法状态值。

## 数学公式

1. 是否只使用 $...$ 或 $$...$$。
2. 是否误用了 \(...\)。
3. 是否误用了 \[...\]。
4. 是否存在未闭合的 $。
5. 是否存在未闭合的 $$。
6. 是否破坏 Markdown 表格。
7. 是否破坏 Obsidian 链接。

## 知识图谱

1. 是否向 Literature Index.md 添加了批量 Papers 链接。
2. 是否向 Retrieval Queue.md 添加了批量 Papers 链接。
3. 是否生成新的 .md 备份文件并导致批量链接进入图谱。
4. 如果创建备份，是否使用：
   text    .md.bak    
   后缀。
5. 是否保留 Topics 中有意义的语义链接。
6. 是否避免生成新的管理型高度数节点。

## 链接

1. 是否存在损坏内部链接。
2. 表格链接是否使用转义后的 \|。
3. 是否存在指向不存在文件的链接。
4. 是否创建孤立论文笔记。

# 十五、输出批次报告

完成后，在终端输出：

text 文献联网检索与整理批次完成。  批次信息： - 批次编号： - 本批次计划处理论文数量： - 实际处理论文数量： - 获得全文的论文数量： - 仅获得摘要的论文数量： - 无法处理的论文数量： - 确认官方代码仓库的论文数量：  本批次论文： 1. 论文：    - 原始状态：    - 当前状态：    - 使用来源：    - 是否获得全文：    - PDF 路径：    - 是否获得官方代码仓库：    - 总结级别：    - 修改文件：    - 待确认问题：  2. 论文：    ...  文件变更： - 新下载 PDF： - 修改的论文笔记： - 修改的 Workflow 文件： - 是否修改 Literature Index.md： - 是否修改 Retrieval Queue.md： - 是否修改 Base 文件： - 新增推荐文献数量：  检查结果： - 是否存在重复论文笔记： - 是否存在重复 PDF： - 是否存在损坏 PDF： - 是否存在非法 YAML： - 是否覆盖用户原始备注： - 是否存在未闭合公式： - 是否仍使用 `\(...\)`： - 是否仍使用 `\[...\]`： - 是否破坏 Markdown 表格： - 是否破坏 Obsidian 链接： - 是否存在无效内部链接： - 是否生成新的管理型高度数节点： - 是否存在未经确认却被补全的信息：  安全检查： - 是否访问与本批次无关的网站： - 是否尝试绕过付费墙： - 是否删除用户文件： - 是否执行训练代码： - 是否安装依赖： - 是否下载大型模型权重：  下一步建议： - 建议人工重点检查的论文： - 剩余未完成论文数量： - 建议下一批处理的论文： 

# 十六、停止条件

完成本批次最多 5 篇论文后立即停止。

不要自动开始下一批。

不要预先下载下一批论文。

不要自动扩充文献列表。

不要向管理型 Markdown 文件重新写入批量 Papers 链接。

等待用户检查结果后，再执行下一批。