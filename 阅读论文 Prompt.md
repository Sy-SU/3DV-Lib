请继续执行当前 Obsidian 文献库的联网检索、整理、Git 提交、远端分支推送和 Pull Request 创建任务。

当前项目根目录固定为：

text /Users/susenyang/Documents/Obsidian Vault/3DV-Lib 

本次只处理下一批最多 5 篇文献。

完成本批次文献整理、本地检查、Git 提交、远端分支推送和 Pull Request 创建后，立即停止。

不要自动处理下一批论文。

不要自动合并 Pull Request。

不要直接向 main 推送。

不要执行任何形式的强制推送。

# 一、总体执行顺序

严格按照以下顺序执行：

text 进入项目根目录 → 检查仓库根目录和本地工作区 → 在必要权限下验证 GitHub CLI、GitHub API 和 origin/main → 同步 origin/main → 阅读项目规则 → 扫描论文 YAML → 确定批次编号 → 选择最多 5 篇论文 → 创建本批次 Git 分支 → 在必要权限下联网检索并整理论文 → 更新论文笔记和 Retrieval Log → 执行本地完整性检查 → 检查 Git 变更 → 创建一个 commit → 在必要权限下 rebase 最新 origin/main → 推送本批次远端分支 → 创建 Pull Request → 输出报告 → 停止 

# 二、工作目录边界

进入：

bash cd "/Users/susenyang/Documents/Obsidian Vault/3DV-Lib" 

只允许修改当前项目目录内部的文件。

禁止：

1. 不要修改父目录：

   text    /Users/susenyang/Documents/Obsidian Vault    

2. 不要修改项目根目录之外的任何文件。
3. 不要在父目录执行 Git 命令。
4. 不要初始化新的 Git 仓库。
5. 不要修改 Git remote。
6. 不要输出凭据内容。
7. 不要保存 Token、密码、Cookie、SSH 私钥或其他敏感凭据。
8. 不要自动修改 ~/.codex/config.toml。
9. 不要自动执行 gh auth login。
10. 不要因为默认沙箱中的认证或网络错误，直接要求用户重新登录 GitHub。

# 三、沙箱、联网与凭据访问规则

部分命令在 Codex 默认沙箱中可能无法读取系统凭据，也可能无法访问网络。

对于以下命令：

bash gh auth status --hostname github.com gh api user --jq '.login' gh repo view "Sy-SU/3DV-Lib" --json nameWithOwner,defaultBranchRef git fetch origin main git pull --ff-only origin main git ls-remote --exit-code --heads origin "<分支名>" git push -u origin "<分支名>" gh pr create ... gh pr view ... 

执行以下规则。

## 3.1 默认沙箱失败不能直接视为登录失败

如果默认沙箱中出现以下情况：

text token-unreadable unable to read authentication token DNS resolution failed Could not resolve host unable to connect to api.github.com network access denied authentication failed credential store unavailable 

不要立即停止任务。

不要立即要求用户执行：

bash gh auth login 

不要把默认沙箱中的失败直接解释为“GitHub CLI 未登录”。

## 3.2 请求权限升级后重试

出现上述情况时：

1. 请求必要的权限升级；
2. 在允许网络访问和读取系统凭据的执行模式下重新运行原命令；
3. 只在权限升级后的命令仍然失败时，才停止任务并报告异常。

权限升级后的失败报告应区分：

text GitHub CLI 登录异常 GitHub API 无法访问 Git remote 配置异常 DNS 或网络仍不可用 权限升级未获批准 

不要混淆这些情况。

## 3.3 禁止输出敏感信息

禁止执行或输出：

bash gh auth token 

禁止打印：

text Token Cookie 密码 SSH 私钥 凭据文件内容 

如需报告认证状态，只输出：

text 已登录账号 目标主机 认证是否可用 

# 四、Git 与 GitHub 前置检查

在修改任何项目文件之前执行：

bash cd "/Users/susenyang/Documents/Obsidian Vault/3DV-Lib"  git rev-parse --show-toplevel git status --short git remote -v git branch --show-current 

## 4.1 仓库根目录检查

git rev-parse --show-toplevel 必须返回：

text /Users/susenyang/Documents/Obsidian Vault/3DV-Lib 

如果返回其他目录：

1. 不要继续。
2. 不要修改文件。
3. 输出实际仓库根目录。
4. 立即停止。

## 4.2 工作区检查

任务开始前，工作区必须干净。

如果：

bash git status --short 

存在任何输出：

1. 不要自动提交已有修改。
2. 不要覆盖已有修改。
3. 不要丢弃已有修改。
4. 不要执行 git stash。
5. 列出已修改文件。
6. 输出：

   text    工作区在批次开始前不是干净状态，需要人工检查。    

7. 立即停止。

## 4.3 GitHub CLI、API 和仓库验证

对以下命令请求必要的联网和凭据读取权限：

bash gh auth status --hostname github.com gh api user --jq '.login' gh repo view "Sy-SU/3DV-Lib" --json nameWithOwner,defaultBranchRef git fetch origin main 

在允许网络和凭据访问的执行模式下，确认：

1. gh auth status --hostname github.com 成功；
2. gh api user --jq '.login' 输出：

   text    Sy-SU    

3. gh repo view "Sy-SU/3DV-Lib" --json nameWithOwner,defaultBranchRef 成功；
4. 仓库名称为：

   text    Sy-SU/3DV-Lib    

5. 默认分支为：

   text    main    

6. git fetch origin main 成功；
7. origin 已正确配置。

如果默认沙箱执行失败，必须先按第三节请求权限升级并重试。

仅当权限升级后仍失败时，才停止任务。

不要要求用户重复执行 gh auth login，除非权限升级后的：

bash gh auth status --hostname github.com gh api user --jq '.login' 

仍明确显示认证无效。

# 五、同步基础分支

前置检查通过后执行：

bash git checkout main git pull --ff-only origin main git status --short 

对：

bash git pull --ff-only origin main 

请求必要的联网和凭据读取权限。

要求：

1. 不要执行：

   bash    git reset --hard    

2. 不要执行：

   bash    git push --force    

3. 不要执行：

   bash    git push --force-with-lease    

4. 如果本地 main 无法 fast-forward：
   - 不要自动解决；
   - 不要继续文献整理；
   - 输出错误；
   - 立即停止。

5. 同步后，如果：

   bash    git status --short    

   仍有输出，停止任务。

# 六、读取项目规则

完整阅读：

text AGENTS.md Literature/Workflow/Retrieval Queue.md Literature/Workflow/Retrieval Log.md 

同时扫描：

text Literature/Papers/*.md 

严格遵守 AGENTS.md 中已有的：

1. 文献管理规则；
2. 知识图谱清理规则；
3. Markdown 表格规则；
4. Obsidian 数学公式渲染规则；
5. PDF 本地保存规则；
6. .obsidian/ 同步规则；
7. Git 分支和 Pull Request 规则；
8. 仓库边界规则。

# 七、确定批次编号

读取：

text Literature/Workflow/Retrieval Log.md 

查找现有最大批次编号。

例如，如果日志中最大编号为：

text Batch 003 

则本次使用：

text Batch 004 

如果尚无历史批次，则使用：

text Batch 001 

批次编号统一使用三位数字：

text 001 002 003 ... 

不要覆盖已有日志。

# 八、选择本批次论文

不要从：

text Literature/Workflow/Retrieval Queue.md 

解析批量论文表格。

当前待处理队列由论文笔记中的 YAML Properties 动态决定。

扫描：

text Literature/Papers/*.md 

筛选尚未完成完整总结的论文。

跳过：

yaml summary_level: full 

无论其 source_level 为：

yaml fulltext 

还是：

yaml fulltext-and-code 

都跳过。

候选论文排序规则如下。

## 第一层：priority

text high medium low 

## 第二层：status

text needs-review reading skimmed unread reference-only 

## 第三层：稳定文件名

按字典序排列。

选择排序后的前 5 篇论文。

如果不足 5 篇，则处理剩余论文。

在创建分支之前，先输出：

text 本批次候选论文： 1. 2. 3. 4. 5. 

# 九、创建本批次分支

根据批次编号创建新分支。

例如，本批次编号为：

text Batch 004 

则基础分支名为：

text codex/literature-batch-004 

在创建前检查本地和远端是否已存在同名分支：

bash git show-ref --verify --quiet "refs/heads/codex/literature-batch-004" || true git ls-remote --exit-code --heads origin "codex/literature-batch-004" || true 

对：

bash git ls-remote --exit-code --heads origin "<分支名>" 

请求必要的联网和凭据读取权限。

如果本地或远端已存在同名分支：

1. 不要覆盖。
2. 不要复用旧分支。
3. 使用递增后缀：

   text    codex/literature-batch-004-02    codex/literature-batch-004-03    

创建分支：

bash git checkout -b "<实际分支名>" 

在报告中记录实际使用的分支名。

分支创建完成后，才开始联网检索和项目文件修改。

# 十、允许的文献检索联网范围

本次允许访问网络，但仅限当前批次中的最多 5 篇论文。

联网检索、PDF 下载、论文官方页面访问和官方代码仓库 README 阅读，均应请求必要的联网权限。

优先来源：

text arxiv.org openaccess.thecvf.com proceedings.neurips.cc proceedings.mlr.press aclanthology.org openreview.net dl.acm.org ieeexplore.ieee.org link.springer.com sciencedirect.com github.com 论文官方项目主页 作者主页 

来源优先级：

1. 论文官方页面；
2. arXiv 页面和 arXiv PDF；
3. 会议或期刊官方页面；
4. 作者主页；
5. 官方项目主页；
6. 官方 GitHub 仓库；
7. 其他可信公开来源。

禁止：

1. 不要绕过付费墙。
2. 不要访问需要登录的论文页面。
3. 不要从不明来源下载 PDF。
4. 不要将博客、社交媒体、搜索结果摘要或自动生成的二次解读作为论文全文依据。
5. 不要搜索与当前批次无关的论文。
6. 不要自动扩大当前批次。
7. 不要在当前批次结束后处理下一批。
8. 不要执行训练代码。
9. 不要安装依赖。
10. 不要下载模型权重。
11. 不要克隆体积较大的仓库。

# 十一、本地目录规则

论文笔记目录：

text Literature/Papers/ 

PDF 本地保存目录：

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

例如：

text Literature/Papers/DDIM.md Literature/PDFs/DDIM.pdf 

不要擅自重命名已有文件。

正常情况下，不要手工修改：

text Literature/Literature Dashboard.base Literature/Workflow/Retrieval Queue.base 

Base 文件会自动读取 YAML Properties。

# 十二、逐篇处理流程

对于本批次中的每篇论文，严格按照以下顺序执行。

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
5. 如果新信息与已有信息冲突，将差异写入：

   markdown    ## 待确认问题    

## Step 2：确认论文身份

联网搜索后，至少确认：

1. 标题是否匹配；
2. 简称是否匹配；
3. arXiv 编号是否匹配；
4. DOI 是否匹配；
5. 作者是否与标题一致；
6. 是否存在同名论文；
7. 是否存在预印本和正式发表版本；
8. 是否已有本地 PDF；
9. 是否已有官方代码仓库；
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

优先检查：

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

如果存在官方 GitHub 仓库、官方项目主页或作者公开代码：

1. 记录链接。
2. 阅读 README 或项目说明。
3. 只记录能够确认的信息。
4. 不执行代码。
5. 不安装依赖。
6. 不下载模型权重。
7. 不克隆大型仓库。

如果无法确认仓库是否官方，不填写：

yaml code_url: 

## Step 5：按材料级别生成总结

根据实际获得的材料决定总结级别。

# 十三、完整总结规则

只有在实际获取并读取完整 PDF 后，才允许设置：

yaml source_level: fulltext summary_level: full status: fully-summarized 

如果同时确认并阅读官方代码仓库 README 或项目文档，可以设置：

yaml source_level: fulltext-and-code 

完整总结至少填写：

markdown ## 基本信息  ## 一句话概括  ## 研究问题  ## 方法概述  ## 关键公式  ## 实验设置  ## 主要结果  ## 论文贡献  ## 局限性  ## 与其他论文的关系  ## 与当前研究方向的关系  ## 待确认问题  ## 阅读记录  ### 摘要总结  ### 粗读记录  ### 完整总结  ## 引用信息 

## 13.1 基本信息

记录：

text 简称 作者 发表年份 会议或期刊 原文链接 arXiv 链接 DOI 官方代码仓库 本地 PDF 路径 所属主题 实际阅读版本 

无法确认的信息保持为空，不要猜测。

## 13.2 一句话概括

使用 2 至 4 句话说明：

1. 论文解决的问题；
2. 提出的主要方法；
3. 最主要的作用或结果。

## 13.3 研究问题

说明：

1. 任务背景；
2. 已有方法的限制；
3. 作者处理的具体问题。

## 13.4 方法概述

说明：

1. 输入与输出；
2. 整体框架；
3. 核心模块；
4. 训练流程；
5. 推理流程；
6. 与已有工作的差异。

不要只翻译摘要。

## 13.5 关键公式

仅记录理解论文所必需的公式。

每个公式需要：

1. 说明公式含义；
2. 解释符号；
3. 说明公式在方法中的作用；
4. 尽量注明论文公式编号。

数学公式使用 Obsidian 兼容格式。

短公式使用：

markdown $t$ 

长公式使用：

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

除非原论文符号另有含义。

## 13.6 实验设置

记录：

1. 数据集；
2. 评价指标；
3. 对比方法；
4. 训练设置；
5. 消融实验。

无法确认的内容不要填写。

## 13.7 主要结果

仅记录重要结果。

每个数值注明：

1. 数据集；
2. 指标；
3. 实验条件；
4. 表格编号或章节位置。

不要脱离上下文堆砌数字。

## 13.8 论文贡献

使用编号列表总结。

## 13.9 局限性

区分：

text 作者明确说明的局限性 个人分析 

个人分析必须明确标记：

text 个人分析： 

不要把个人判断写成作者结论。

## 13.10 与其他论文的关系

记录：

text 前置工作 后续工作 相似方法 主要区别 

只记录可以确认的信息。

## 13.11 与当前研究方向的关系

结合论文所属 Topic，说明其用途，例如：

text 扩散模型理论基础 文生图方法基础 文生 3D 的关键方法 Mesh Tokenization 相关方法 3D 场景生成方法 物理交互建模方法 

不要写空泛评价。

# 十四、摘要级总结规则

如果只能获得可信摘要，设置：

yaml source_level: abstract summary_level: abstract status: skimmed 

只允许填写：

markdown ## 基本信息  ## 一句话概括  ## 阅读记录  ### 摘要总结  ## 待确认问题 

以下章节保持原样：

markdown ## 方法概述  > 尚未总结。  ## 关键公式  > 尚未总结。  ## 实验设置  > 尚未总结。  ## 主要结果  > 尚未总结。  ## 局限性  > 尚未总结。 

不要根据摘要推断正文细节。

# 十五、无法处理时的规则

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

# 十六、更新 YAML Front Matter

对于成功处理的论文，更新：

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
6. 如果新旧信息冲突，写入：

   markdown    ## 待确认问题    

7. pdf_path 必须使用项目内相对路径。
8. 不要写入本机绝对路径。

# 十七、队列更新规则

不要向：

text Literature/Workflow/Retrieval Queue.md 

写入批量论文表格。

该文件只保留工作流说明。

不要手工修改：

text Literature/Workflow/Retrieval Queue.base 

论文状态写回各自 YAML 后，Base 会自动更新。

每篇论文处理完成后，确认：

1. YAML 状态已经更新。
2. Retrieval Queue.base 可以通过 Properties 自动反映新状态。
3. summary_level: full 的论文会自动退出待处理视图。
4. 未完成论文仍然保留在待处理视图中。

# 十八、日志更新规则

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
10. 论文名称使用普通文本，例如：

   text    DDPM    DDIM    LDM    

11. 不要在 Retrieval Log.md 中为论文名称添加 Wikilink。
12. 不要写成：

   markdown    [[../Papers/DDPM\|DDPM]]    

13. 普通 URL 可以保留。
14. Markdown 表格列数必须保持一致。
15. 不要重新排列已有历史记录。

# 十九、索引和 Topic 文件规则

不要向：

text Literature/Literature Index.md 

重新写入批量论文状态表格。

不要手工修改：

text Literature/Literature Dashboard.base 

状态变化会自动从论文 YAML 中读取。

对于：

text Literature/Topics/*.md 

执行：

1. 保留人工整理的阅读顺序。
2. 保留有语义的论文链接。
3. 保留用户原始备注。
4. 不批量写入所有论文状态。
5. 不创建无意义的管理型链接。
6. 如果 Topic 文件中已经存在论文状态表格，只更新本批次对应行。
7. 如果 Topic 文件中没有状态表格，不要新增。
8. 表格内的 Obsidian 链接别名分隔符必须使用：

   markdown    \|    

# 二十、推荐文献规则

如果阅读本批次论文时发现值得补充的新论文：

1. 不要自动创建论文笔记。
2. 不要自动加入待处理队列。
3. 不要自动下载。
4. 只记录到：

   text    Literature/Workflow/Suggested Papers.md    

如果文件不存在，创建：

markdown # Suggested Papers  | 论文 | 推荐原因 | 来源论文 | 链接 | 是否加入队列 | |---|---|---|---|---| 

最后一列填写：

text 待用户确认 

# 二十一、批次结束后的本地检查

完成本批次后，执行以下检查。

## 21.1 论文文件

1. 是否创建重复论文笔记。
2. 是否下载重复 PDF。
3. PDF 是否能够读取。
4. YAML 是否合法。
5. 是否误删用户备注。
6. 是否误改稳定文件名。
7. 是否存在空白论文文件。

## 21.2 总结级别

1. summary_level: full 是否都有全文来源。
2. source_level: fulltext 是否都有本地 PDF。
3. source_level: fulltext-and-code 是否确实阅读过官方仓库说明。
4. source_level: abstract 是否没有伪造正文总结。
5. status: fully-summarized 是否存在结构化完整总结。
6. 是否存在非法状态值。

## 21.3 数学公式

1. 是否只使用 $...$ 或 $$...$$。
2. 是否误用了 \(...\)。
3. 是否误用了 \[...\]。
4. 是否存在未闭合的 $。
5. 是否存在未闭合的 $$。
6. 是否破坏 Markdown 表格。
7. 是否破坏 Obsidian 链接。

## 21.4 知识图谱

1. 是否向 Literature Index.md 添加批量 Papers 链接。
2. 是否向 Retrieval Queue.md 添加批量 Papers 链接。
3. 是否向 Retrieval Log.md 添加论文 Wikilink。
4. 是否生成新的 .md 备份文件并导致批量链接进入图谱。
5. 如果创建备份，是否使用：

   text    .md.bak    

6. 是否保留 Topics 中有意义的语义链接。
7. 是否避免生成新的管理型高度数节点。

## 21.5 链接

1. 是否存在损坏内部链接。
2. Topic 表格链接是否使用转义后的 \|。
3. 是否存在指向不存在文件的链接。
4. 是否创建孤立论文笔记。

# 二十二、PDF、备份和敏感文件检查

本项目允许将 PDF 保存到：

text Literature/PDFs/ 

但 PDF 仅供本地使用，禁止提交。

备份目录：

text Literature/Workflow/Backups/ 

也禁止提交。

执行：

bash git check-ignore -v "Literature/PDFs/" || true git check-ignore -v "Literature/Workflow/Backups/" || true git ls-files -- "Literature/PDFs" git ls-files -- "Literature/Workflow/Backups" 

要求：

1. .gitignore 必须排除：

   text    Literature/PDFs/    Literature/Workflow/Backups/    

2. git ls-files -- "Literature/PDFs" 应无输出。
3. git ls-files -- "Literature/Workflow/Backups" 应无输出。

4. 如果 PDF 或备份已经被 Git 跟踪：
   - 不要提交；
   - 不要自动执行 git rm --cached；
   - 列出文件；
   - 立即停止；
   - 等待人工处理。

禁止提交：

text *.pdf *.md.bak .DS_Store Thumbs.db .trash/ *.tmp *.swp *.swo *~ 

.obsidian/ 属于 Git 同步范围。

允许提交：

text .obsidian/ 

中的修改。

但是提交前必须检查是否存在疑似敏感文件。

检查文件名和路径中的常见敏感关键词：

text token secret password credential cookie private_key id_rsa .pem .env 

要求：

1. 不输出敏感内容全文。
2. 不读取或打印私钥内容。
3. 如果发现疑似敏感文件：
   - 只输出文件路径；
   - 不提交；
   - 立即停止；
   - 标记：

     text      需要人工检查，暂不推送。      

# 二十三、提交前 Git 检查

完成本地检查后执行：

bash git status --short git diff --stat git diff --check git ls-files --others --exclude-standard 

检查：

1. 是否只修改当前仓库中的文件。
2. 是否存在尾随空格、冲突标记或格式异常。
3. 是否误加入 PDF。
4. 是否误加入备份。
5. 是否误加入敏感文件。
6. 是否误创建新的 .md 管理型高度数节点。
7. 是否存在与本批次无关的大量修改。
8. 是否出现异常大型文件。
9. 是否出现无法解释的新增文件。

如果存在无法解释的新增文件：

1. 不要提交。
2. 列出文件。
3. 立即停止。
4. 等待人工检查。

如果本批次没有任何可提交修改：

1. 不创建空 commit。
2. 不推送空分支。
3. 不创建空 PR。
4. 输出：

   text    本批次没有产生可提交修改。    

5. 停止任务。

# 二十四、暂存文件

确认检查通过后执行：

bash git add -A git status --short git diff --cached --stat git diff --cached --check git diff --cached --name-only 

要求：

1. 暂存区不得包含：

   text    Literature/PDFs/    Literature/Workflow/Backups/    

2. 暂存区不得包含 PDF。
3. 暂存区不得包含 .md.bak。
4. 暂存区不得包含疑似敏感文件。
5. 暂存区不得包含异常大型文件。
6. 暂存区不得包含仓库外文件。

如果发现不应提交的文件：

1. 不要提交。
2. 不要删除文件。
3. 可以执行：

   bash    git restore --staged -- "<文件路径>"    

4. 输出文件路径。
5. 立即停止，等待人工检查。

# 二十五、创建本地 commit

暂存区检查通过后，只创建一个 commit。

提交信息格式：

text literature(batch-<三位批次编号>): summarize next papers 

例如：

text literature(batch-004): summarize next papers 

执行：

bash git commit -m "literature(batch-004): summarize next papers" git rev-parse HEAD 

要求：

1. 每批只创建一个 commit。
2. 不修改已有 commit。
3. 不执行：

   bash    git commit --amend    

4. 不执行历史重写。
5. 不执行交互式 rebase。

# 二十六、推送前同步最新远端 main

提交完成后执行：

bash git fetch origin main git rebase origin/main 

请求必要的联网和凭据读取权限。

如果默认沙箱失败，按第三节请求权限升级后重试。

如果 rebase 没有冲突，继续。

如果出现冲突：

1. 不要自动解决。
2. 输出冲突文件列表：

   bash    git diff --name-only --diff-filter=U    

3. 执行：

   bash    git rebase --abort    

4. 保留本地 commit。
5. 不要强制推送。
6. 输出：

   text    远端 main 已更新，自动 rebase 出现冲突。已中止 rebase，需要人工处理。    

7. 立即停止。

# 二十七、推送远端分支

确认 rebase 成功后执行：

bash git push -u origin "<实际分支名>" 

请求必要的联网和凭据读取权限。

如果默认沙箱失败，按第三节请求权限升级后重试。

例如：

bash git push -u origin "codex/literature-batch-004" 

要求：

1. 只推送当前批次分支。
2. 不要推送 main。
3. 不要推送其他分支。
4. 不要推送 tags。
5. 不要执行：

   bash    git push --force    git push --force-with-lease    

如果推送失败：

1. 不要回滚本地文献修改。
2. 不要删除本地 commit。
3. 不要强制推送。
4. 输出失败原因。
5. 立即停止。

# 二十八、创建 Pull Request

推送成功后，使用 GitHub CLI 创建 Pull Request。

PR 基础分支：

text main 

PR 来源分支：

text <实际分支名> 

PR 标题：

text Literature Batch <三位批次编号>: summarize next papers 

例如：

text Literature Batch 004: summarize next papers 

创建临时 PR 正文文件：

bash cat > "/tmp/3dv-lib-pr-body.md" <<'EOF' ## Batch Summary  - Batch: - Branch: - Processed papers: - Full-text summaries: - Abstract-only summaries: - Unresolved papers: - Confirmed official repositories:  ## Modified Files  - Paper notes: - Workflow files: - `.obsidian/` modified:  ## Local-only Files  - New local PDFs: - `Literature/PDFs/` was not committed. - `Literature/Workflow/Backups/` was not committed.  ## Checks  - YAML valid: - Obsidian math delimiters valid: - Markdown tables valid: - Graph hygiene rules preserved: - No PDF staged: - No backup staged: - No sensitive files staged:  ## Pending Review  -  This PR was created automatically after one literature-processing batch. It has not been merged automatically. EOF 

实际执行时，必须将正文中的占位内容替换为本批次真实信息。

然后执行：

bash gh pr create \   --repo "Sy-SU/3DV-Lib" \   --base "main" \   --head "<实际分支名>" \   --title "Literature Batch <三位批次编号>: summarize next papers" \   --body-file "/tmp/3dv-lib-pr-body.md" 

对：

bash gh pr create ... 

请求必要的联网和凭据读取权限。

如果默认沙箱失败，按第三节请求权限升级后重试。

要求：

1. 不自动合并 PR。
2. 不执行：

   bash    gh pr merge    

3. 不关闭已有 PR。
4. 不删除远端分支。
5. 不修改仓库设置。
6. 不修改 branch protection。
7. 不创建 GitHub Release。
8. 不创建 Issue，除非用户明确要求。
9. PR 创建结束后，删除临时文件：

   bash    rm -f "/tmp/3dv-lib-pr-body.md"    

如果 gh pr create 失败：

1. 保留已经推送的远端分支。
2. 不删除远端分支。
3. 输出失败原因。
4. 输出可以用于手工创建 PR 的分支名。
5. 删除临时 PR 正文文件。
6. 立即停止。

# 二十九、检查 Pull Request

PR 创建成功后执行：

bash gh pr view \   --repo "Sy-SU/3DV-Lib" \   --head "<实际分支名>" \   --json url,number,title,baseRefName,headRefName,state 

如果当前 GitHub CLI 版本不支持：

bash --head 

则使用：

bash gh pr view "<PR 编号>" \   --repo "Sy-SU/3DV-Lib" \   --json url,number,title,baseRefName,headRefName,state 

请求必要的联网和凭据读取权限。

如果默认沙箱失败，按第三节请求权限升级后重试。

检查：

1. baseRefName 是否为：

   text    main    

2. headRefName 是否为当前批次分支。
3. state 是否为：

   text    OPEN    

4. 获取 PR URL。
5. 不自动打开浏览器。
6. 不自动合并。

PR 创建完成后：

1. 保持当前批次分支不变。
2. 不切换回 main。
3. 不删除当前分支。
4. 不删除本地 PDF。
5. 不删除本地备份。
6. 不处理下一批论文。

# 三十、更新 AGENTS.md

检查：

text AGENTS.md 

如果以下规则尚不存在，在文件末尾追加。

不要重复添加相同规则。

markdown ## GitHub CLI Sandbox-Aware Workflow  ### Repository boundary  - The repository root is:   text
  /Users/susenyang/Documents/Obsidian Vault/3DV-Lib
  - Never run Git commands in the parent Obsidian Vault. - Never modify files outside this repository.  ### GitHub CLI authentication and sandbox behavior  - Default-sandbox failures must not be interpreted immediately as GitHub logout. - For GitHub CLI and remote Git commands, request the necessary network and credential-reading permissions when needed. - Commands that may require elevated permissions include:  bash
  gh auth status --hostname github.com
  gh api user --jq '.login'
  gh repo view "Sy-SU/3DV-Lib" --json nameWithOwner,defaultBranchRef
  git fetch origin main
  git pull --ff-only origin main
  git ls-remote --heads origin "<branch>"
  git push -u origin "<branch>"
  gh pr create ...
  gh pr view ...
  - If the default sandbox reports token-unreadable, DNS failure, inability to reach `api.github.com`, or credential-store access failure, request elevated permissions and retry. - Only report a GitHub CLI login failure if the elevated retry also fails. - Do not ask the user to run `gh auth login` based only on a default-sandbox failure. - Never run or print:  bash
  gh auth token
  - Never print, store, or commit credentials, tokens, passwords, cookies, or SSH private keys.  ### Batch branches  - Process each literature batch on a new branch created from the latest `origin/main`. - Use branch names such as:  text
  codex/literature-batch-004
  - If a branch name already exists, use an incrementing suffix. - Never push literature-processing changes directly to `main`.  ### Commit policy  - Create exactly one commit per literature batch. - Use commit messages such as:  text
  literature(batch-004): summarize next papers
  - Do not amend commits. - Do not rewrite Git history. - Do not use force push.  ### Pull Request policy  - Push the batch branch to `origin`. - Create a Pull Request targeting `main`. - Do not merge the Pull Request automatically. - Do not delete the remote branch automatically. - Report the PR URL after creation.  ### Local-only files  - Never stage, commit, or push:  text
  Literature/PDFs/
  Literature/Workflow/Backups/
  - `.obsidian/` is included in Git synchronization. - Check `.obsidian/` for suspicious credential-like files before committing.  ### Graph hygiene  - In `Literature/Workflow/Retrieval Log.md`, write paper names as plain text. - Do not add paper Wikilinks to `Retrieval Log.md`. - Do not add bulk `[[Papers/...]]` links to management Markdown files.

# 三十一、输出完整批次报告

完成后，在终端输出：

text 文献联网检索、整理与 PR 创建完成。  批次信息： - 批次编号： - 本批次计划处理论文数量： - 实际处理论文数量： - 获得全文的论文数量： - 仅获得摘要的论文数量： - 无法处理的论文数量： - 确认官方代码仓库的论文数量：  本批次论文： 1. 论文：    - 原始状态：    - 当前状态：    - 使用来源：    - 是否获得全文：    - PDF 路径：    - 是否获得官方代码仓库：    - 总结级别：    - 修改文件：    - 待确认问题：  2. 论文：    ...  文件变更： - 新下载 PDF： - 修改的论文笔记： - 修改的 Workflow 文件： - 是否修改 Literature Index.md： - 是否修改 Retrieval Queue.md： - 是否修改 Base 文件： - 是否修改 `.obsidian/`： - 新增推荐文献数量：  完整性检查： - 是否存在重复论文笔记： - 是否存在重复 PDF： - 是否存在损坏 PDF： - 是否存在非法 YAML： - 是否覆盖用户原始备注： - 是否存在未闭合公式： - 是否仍使用 `\(...\)`： - 是否仍使用 `\[...\]`： - 是否破坏 Markdown 表格： - 是否破坏 Obsidian 链接： - 是否存在无效内部链接： - 是否生成新的管理型高度数节点： - 是否存在未经确认却被补全的信息：  GitHub CLI 与沙箱： - 默认沙箱中 GitHub CLI 是否失败： - 是否请求权限升级： - 权限升级是否获批： - 权限升级后 `gh auth status --hostname github.com` 是否成功： - 权限升级后 `gh api user --jq '.login'` 输出： - 权限升级后 `gh repo view` 是否成功： - 权限升级后 `git fetch origin main` 是否成功： - 是否要求用户重新登录 GitHub： - 如果要求重新登录，依据是否来自权限升级后的明确认证失败：  Git 与 Pull Request： - 仓库根目录： - 基础分支： - 实际使用的批次分支： - 开始任务前工作区是否干净： - 是否成功同步 origin/main： - 是否发现被 Git 跟踪的 PDF： - 是否发现被 Git 跟踪的备份： - 是否发现疑似敏感文件： - 是否存在无法解释的新增文件： - 暂存文件数量： - Commit 是否创建成功： - Commit hash： - Commit message： - 是否成功 rebase origin/main： - 是否成功推送远端分支： - 是否成功创建 Pull Request： - PR 编号： - PR URL： - PR base： - PR head： - PR 状态： - 是否自动合并 PR：否 - 是否直接推送 main：否 - 是否执行 force push：否 - 是否提交 PDF：否 - 是否提交 Workflow Backups：否  安全检查： - 是否访问与本批次无关的网站： - 是否尝试绕过付费墙： - 是否删除用户文件： - 是否执行训练代码： - 是否安装依赖： - 是否下载大型模型权重： - 是否输出敏感凭据：否  下一步建议： - 建议人工重点检查的论文： - 剩余未完成论文数量： - 建议下一批处理的论文： - 等待用户在 GitHub 上审查并合并 PR。 

# 三十二、最终停止条件

完成以下步骤后立即停止：

1. 完成本批次最多 5 篇论文的联网检索与总结。
2. 完成本地完整性检查。
3. 创建本批次 Git 分支。
4. 创建一个本地 commit。
5. 推送当前批次远端分支。
6. 创建一个指向 main 的 Pull Request。
7. 输出 PR URL。
8. 输出完整批次报告。

禁止：

1. 不要自动处理下一批论文。
2. 不要自动合并 Pull Request。
3. 不要直接向 main 推送。
4. 不要删除远端分支。
5. 不要执行 force push。
6. 不要预先下载下一批论文。
7. 不要自动扩充文献列表。
8. 不要向管理型 Markdown 文件重新写入批量 Papers 链接。
9. 不要因为默认沙箱中的凭据或网络失败，直接要求用户重新登录 GitHub。
10. 不要输出任何敏感凭据。

等待用户在 GitHub 上检查并合并 PR 后，再执行下一批。