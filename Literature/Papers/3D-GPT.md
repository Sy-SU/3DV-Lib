---
title: "3D-GPT: Procedural 3D Modeling with Large Language Models"
short_name: "3D-GPT"
authors:
  - "Chunyi Sun"
  - "Junlin Han"
  - "Weijian Deng"
  - "Xinlong Wang"
  - "Zishan Qin"
  - "Stephen Gould"
year: 2023
venue: "arXiv 2023"
url: "https://arxiv.org/abs/2310.12945"
doi: ""
arxiv: "2310.12945"
code_url: "https://github.com/Chuny1/3DGPT"
topics: 
  - "3d-scene-generation"
  - "procedural-generation"
paper_type: "method"
task: "scene-generation"
tags:
  - literature
status: fully-summarized
summary_level: full
source_level: fulltext
priority: medium
read_date: "2026-05-31"
review_date:
related_projects:
  - "https://chuny1.github.io/3DGPT/3dgpt.html"
  - "https://github.com/Chuny1/3DGPT"
pdf_path: "Literature/PDFs/3D-GPT.pdf"
retrieved_from:
  - "https://arxiv.org/abs/2310.12945"
  - "https://openreview.net/pdf?id=n04Dbq4Y8J"
  - "https://chuny1.github.io/3DGPT/3dgpt.html"
  - "https://github.com/Chuny1/3DGPT"
retrieval_date: "2026-05-31"
read_version: "OpenReview-hosted PDF, 11 pages; official project/code metadata from Batch 003"
---


# 3D-GPT: Procedural 3D Modeling with Large Language Models

## 基本信息

- 简称：3D-GPT
- 作者：Chunyi Sun; Junlin Han; Weijian Deng; Xinlong Wang; Zishan Qin; Stephen Gould
- 发表时间：2023
- 会议或期刊：arXiv 2023
- 原文链接：https://arxiv.org/abs/2310.12945
- arXiv：2310.12945
- DOI：
- 代码仓库：https://github.com/Chuny1/3DGPT
- 本地 PDF 路径：Literature/PDFs/3D-GPT.pdf
- 所属方向：3d-scene-generation, procedural-generation
- 当前状态：fully-summarized

## 一句话概括

3D-GPT 把大语言模型组织成 task dispatch、conceptualization 和 modeling 三个协作 agent，让 LLM 选择 Infinigen/Blender 程序化函数、补全场景描述中的隐含建模细节，并生成 Python 调用代码，从而用自然语言指令控制 3D 场景生成与连续编辑。

## 研究问题

论文针对程序化 3D 建模的可用性问题：传统程序化生成虽然可控、可复现、能直接接入图形引擎，但使用者必须理解生成规则、函数接口、参数含义和软件操作。直接让 LLM 从文本生成完整 3D 内容也不可靠，因为 LLM 缺少精细 3D 建模的专门训练，容易不知道应调用哪些元素、如何把自然语言映射到具体参数。

作者的目标不是训练新的 neural text-to-3D 表示，而是让 LLM 作为问题求解器和工具使用者，把自然语言指令转化为可执行的程序化建模过程，特别是支持初始场景生成和后续指令式编辑。

## 方法概述

3D-GPT 使用 Infinigen 作为 Python/Blender 程序化生成后端，并把可调用生成能力整理成函数集合。对每个函数，系统向 LLM 提供四类提示材料：函数文档、整理后的 API 代码、辅助参数信息、用法示例。这样做的目的是让 LLM 不只看到函数名，还能理解函数目标、参数类型、视觉属性和调用模式。

框架包含三个 agent：

- Task dispatch agent：读取用户指令和函数文档，从完整函数集合中选择需要调用的函数子集。初始场景通常使用完整函数集合，后续编辑则只选择相关函数，避免无关修改。
- Conceptualization agent：把简短用户描述扩展为更具体的外观与场景描述。例如用户只说某类花，agent 会补足花瓣形状、颜色、中心大小等建模参数相关信息。
- Modeling agent：结合扩展描述、函数代码、函数文档和示例，推断参数并生成 Python 代码，调用 Blender/Infinigen 完成建模和渲染。

论文强调 3D-GPT 的输出是可执行建模脚本和由 Blender 渲染的真实 3D 结果，而不是 NeRF、3D Gaussian 或其他神经表示。

## 关键公式

论文的任务定义使用指令序列表示交互式生成过程：

$$
L = \langle L_i \rangle
$$

其中 $L_0$ 是初始场景描述，$L_i, i > 0$ 是后续编辑指令。

程序化生成后端被表示为函数集合：

$$
F = \{F_j\}
$$

每个函数 $F_j$ 接收参数 $P_j$。对后续编辑，task dispatch agent 会选择相关函数子集：

$$
\hat{F} \subseteq F
$$

随后 conceptualization agent 为每个相关函数补充描述信息，modeling agent 推断对应 $P_j$ 并生成 Python 调用。论文没有提出新的优化目标或训练损失，核心是把 LLM 的规划、推理和工具调用能力嵌入程序化生成流程。

## 实验设置

实验主要验证 3D-GPT 是否能按文本生成和编辑 3D 场景、是否能进行细粒度对象控制、以及三个 agent 和提示组件的作用。

- 大场景生成：作者用 ChatGPT 生成 100 条自然场景描述，方式是用同一提示收集 10 次回复，每次要求 10 条不同自然场景描述。
- 单类对象控制：用随机提示要求 GPT 生成多种真实花卉类型，测试曲线、形状、颜色和外观关键特征的控制能力。
- 后续指令编辑：从初始场景开始，连续执行改变花色、删除树、改变天气、增加山上树木、转为冬季等指令，观察系统是否能保留上下文并只修改相关部分。
- 工具使用案例：以天空纹理函数为例，检查 modeling agent 是否能把“clear blue sky”“hazy”“sunset”等自然语言映射到 sun intensity、sun elevation、air density、dust density、ozone、cloud density 等间接参数。
- 消融实验：分别移除 task dispatch agent、conceptualization agent、提示组件，比较 CLIP Score、Failure Rate 和参数多样性。

## 主要结果

论文报告的关键量化结果来自消融实验。

- Task dispatch agent：在 100 条初始场景描述并追加后续指令的设置下，去掉 TDA 的 CLIP Score 为 22.79，完整方法为 29.16。
- Conceptualization agent：去掉 CA 时 CLIP Score 为 21.51、Failure Rate 为 3.6%、Parameter Diversity 为 6.32；完整方法分别为 30.30、0.8%、7.34。
- 提示组件：完整提示达到 CLIP Score 30.3、Failure Rate 0.8%、Parameter Diversity 7.34；去掉 documentation、API code、auxiliary parameter information、example 都会降低或扰动表现，其中去掉 documentation 和 auxiliary parameter information 对 CLIP Score 影响较大。
- LLM 类型：LLAMA2、GPT-4、GPT-3.5 的 CLIP Score 分别为 29.7、31.2、30.3；Failure Rate 分别为 1.4%、0.6%、0.8%。作者认为所用 LLM 类型没有显著改变 3D 生成结果的总体趋势。
- 示例数量：0-shot 的 CLIP Score 为 24.5、Failure Rate 为 3.4%；1、2、3 个示例的 CLIP Score 分别约为 30.3、30.1、30.2，说明少量示例足以显著改善调用稳定性。

定性结果显示，3D-GPT 能生成多种自然大场景、较好控制单类花卉外观、执行连续场景编辑，并在示例中比 DreamFusion 更适合生成包含大尺度环境元素的程序化场景。该比较主要是视觉展示，论文未给出对应量化指标。

## 论文贡献

- 提出一个 zero-shot、training-free 的 LLM 驱动程序化 3D 场景生成框架。
- 把 LLM 拆分为规划、概念补全和建模代码生成三个角色，降低单一 LLM 直接生成复杂 3D 内容的难度。
- 使用 Infinigen/Blender 作为可执行后端，使输出具备真实 3D 一致性、可渲染、可编辑和可进一步接入图形工作流的特点。
- 通过 agent 与提示组件消融说明：函数选择、概念补全、函数文档、辅助参数信息和示例都对生成质量或稳定性有贡献。

## 局限性

论文没有系统报告失败案例、运行耗时、人工评测规模或与更多 text-to-3D 方法的量化对比。方法依赖程序化后端已有的函数覆盖范围，因此能生成和编辑的内容受 Infinigen/Blender API 以及提示材料质量限制。它更适合可程序化描述的场景与对象，不能直接解决神经生成方法中的真实世界纹理、复杂资产建模和数据驱动分布学习问题。

## 与其他论文的关系

- 前置工作：依赖 Infinigen 这类程序化生成器，也利用 LLM 在规划、工具调用、属性描述和代码理解上的能力。
- 后续工作：与 SceneCraft、Holodeck、SceneX、CityX 等 LLM/agent 驱动的场景生成路线相邻。
- 相似方法：不同于 DreamFusion、Magic3D、Latent-NeRF 等通过优化神经表示或 3D 对象的方法，3D-GPT 直接生成 Blender/Python 程序化调用。
- 主要区别：它不是训练一个新的 3D generative model，而是把 LLM 放在工具编排层，让已有程序化生成系统获得自然语言控制能力。

## 与当前研究方向的关系

对 3D scene generation 知识图谱而言，3D-GPT 是 procedural generation 与 LLM agent 交叉方向的代表性方法。它适合作为理解“LLM 如何控制 3D 生成工具”的节点，并可连接到 Infinigen、procedural generation、text-to-3D、Blender automation、3D scene editing 等主题。

与 NeRF/3DGS 类方法相比，3D-GPT 的价值不在 photorealistic neural rendering，而在可编辑、可脚本化、可与传统图形引擎集成的生成流程。它也提示后续项目可以把 LLM 用作场景布局、资产参数推断、代码生成和交互式编辑的控制层。

## 待确认问题

- [ ] PDF 正文没有提供正式发表 venue 或 DOI；当前仍按 arXiv 2023 记录。
- [ ] 本次只验证了全文 PDF 与项目/代码链接元数据，未逐行审查 GitHub 仓库实现细节，因此 `source_level` 保持为 `fulltext`。

## 阅读记录

### 摘要总结

论文关注程序化 3D 建模门槛高、需要理解规则和参数的问题，提出用 LLM 作为问题求解器，将复杂建模任务拆分并交给不同 agent 协同完成。摘要中的核心模块包括任务分派 agent、概念化 agent 和建模 agent，目标是把简短场景描述扩展为可执行的详细描述，并提取参数连接到 3D 软件进行资产创建。

### 粗读记录

粗读重点是系统结构、agent 分工和消融结果。3D-GPT 的主要设计不是让 LLM 凭空生成 3D 资产，而是把程序化生成函数暴露给 LLM，并通过文档、代码、辅助参数和示例降低函数调用错误。实验结果显示 task dispatch agent 对连续编辑尤其重要，conceptualization agent 对对象外观和参数多样性影响明显。

### 完整总结

完整阅读 PDF 后，确认 3D-GPT 是一套 training-free 的 LLM-to-procedural-generation 框架。它用 task dispatch agent 选择函数，用 conceptualization agent 补足自然语言中缺失但建模所需的视觉属性，用 modeling agent 生成 Python 函数调用，并最终在 Blender 中渲染真实 3D 结果。论文通过大场景生成、单类对象控制、连续编辑、单函数控制和多项消融说明该 pipeline 的有效性。主要证据包括 TDA/CA 消融带来的 CLIP Score 和 failure rate 变化，以及提示组件、LLM 类型、示例数量的消融。局限在于评估仍偏定性，比较对象有限，生成能力受程序化后端函数覆盖和提示设计约束。

## 用户原始备注

> 暂无。

## 引用信息

```bibtex

```
