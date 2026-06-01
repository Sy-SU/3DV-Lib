---
title: "Articulate-Anything: Automatic Modeling of Articulated Objects via a Vision-Language Foundation Model"
short_name: "Articulate-Anything"
authors:
  - "Long Le"
  - "Jason Xie"
  - "William Liang"
  - "Hung-Ju Wang"
  - "Yue Yang"
  - "Yecheng Jason Ma"
  - "Kyle Vedder"
  - "Arjun Krishna"
  - "Dinesh Jayaraman"
  - "Eric Eaton"
year: 2025
venue: "ICLR 2025"
url: "https://arxiv.org/abs/2410.13882"
doi: ""
arxiv: "2410.13882"
code_url: "https://github.com/vlongle/articulate-anything"
topics: 
  - "3d-scene-generation"
  - "interaction-in-3d-generation"
paper_type: "method"
task: "articulated-object-modeling"
tags:
  - literature
status: fully-summarized
summary_level: full
source_level: fulltext-and-code
priority: medium
read_date: "2026-06-01"
review_date:
related_projects:
  - "https://articulate-anything.github.io/"
pdf_path: ""
retrieved_from:
  - "https://arxiv.org/abs/2410.13882"
  - "https://arxiv.org/html/2410.13882"
  - "https://articulate-anything.github.io/"
  - "https://github.com/vlongle/articulate-anything"
retrieval_date: "2026-06-01"
read_version: "arXiv v4 HTML, updated 2025-06-02; official project page and code page"
---


# Articulate-Anything: Automatic Modeling of Articulated Objects via a Vision-Language Foundation Model

## 基本信息

- 简称：Articulate-Anything
- 作者：Long Le; Jason Xie; William Liang; Hung-Ju Wang; Yue Yang; Yecheng Jason Ma; Kyle Vedder; Arjun Krishna; Dinesh Jayaraman; Eric Eaton
- 发表时间：2025
- 会议或期刊：ICLR 2025
- 原文链接：https://arxiv.org/abs/2410.13882
- arXiv：2410.13882
- DOI：
- 代码仓库：https://github.com/vlongle/articulate-anything
- 项目页：https://articulate-anything.github.io/
- 本地 PDF 路径：
- 所属方向：3d-scene-generation, interaction-in-3d-generation
- 当前状态：fully-summarized
- 实际阅读版本：arXiv v4 HTML, updated 2025-06-02; official project page and code page

## 一句话概括

Articulate-Anything 用 VLM actor-critic 把文本、图像或视频输入转成可编译为 URDF 的 Python articulation program，从而自动生成可交互 articulated object digital twin，并能用生成资产训练机器人操作策略。

## 研究问题

可交互 3D 环境和机器人仿真需要大量 articulated objects，但人工标注 links、joints、joint limits 与 URDF 成本高、专业性强，公开 articulated object 数据也远少于静态 3D object 数据。已有自动 articulation 方法输入贫乏，通常依赖 bounding boxes、静态图像或特定类别，难以处理复杂物体、歧义关节和 in-the-wild video。Articulate-Anything 试图让 foundation VLM 通过代码生成和自我评估，把多模态输入自动转成可仿真的 articulated asset。

## 方法概述

1. 表示：输出目标是 URDF tree，包含 links 和 joints；为降低直接写 URDF/XML 的难度，论文设计了高层 Python API，再编译为 URDF。
2. Mesh retrieval：视觉输入通过 VLM 识别目标物体，并用 CLIP embedding / divide-and-conquer tournament 从 PartNet-Mobility 等资产库检索最相似 mesh；文本输入先由 LLM densify prompt、规划 parts 和尺寸，再检索并缩放 mesh。
3. Link placement：VLM actor 生成 Python 代码摆放 links；critic 渲染预测结果并与 ground truth 或输入线索比较，给出反馈，actor 迭代修正。
4. Joint prediction：VLM actor 预测 joint type、axis、origin、limit；critic 检查 render / motion 是否合理，并通过 actor-critic loop 改进。
5. Targeted affordance extraction：当输入视频展示特定运动时，系统不必生成所有可能 joints，而是聚焦视频中展示的 affordance。
6. Robotics application：用生成 articulated assets 在 Robosuite 中训练 PPO 操作策略，再迁移到真实 Franka Panda 机械臂。

## 关键公式

1. articulation objective：

   $$
   \mathcal{L}_{\mathrm{total}}
   =
   \mathcal{L}_{\mathrm{mesh}}
   +
   \mathcal{L}_{\mathrm{link}}
   +
   \mathcal{L}_{\mathrm{joint}}.
   $$

2. mesh discrepancy：

   $$
   \mathcal{L}_{\mathrm{mesh}}
   =
   \sum_{i=1}^{N}
   \mathrm{Chamfer}(\hat{P}_i,P_i^*).
   $$

3. link placement error：

   $$
   \mathcal{L}_{\mathrm{link}}
   =
   \sum_{i=1}^{N}
   \left\lVert \hat{x}_i-x_i^* \right\rVert_2
   +
   2\arccos\left(\left|\hat{q}_i\cdot q_i^*\right|\right).
   $$

4. joint error decomposition：

   $$
   \mathcal{L}_{\mathrm{joint}}
   =
   \mathcal{L}_{\mathrm{joint\ type}}
   +
   \mathcal{L}_{\mathrm{joint\ axis}}
   +
   \mathcal{L}_{\mathrm{joint\ origin}}
   +
   \mathcal{L}_{\mathrm{joint\ limit}}.
   $$

5. link orientation geodesic error：

   $$
   e_{\mathrm{orient}}
   =
   2\arccos\left(\left|q_p\cdot q_g\right|\right).
   $$

## 实验设置

1. 数据集：PartNet-Mobility，约 2.3K objects、1.9K revolute joints、7.6K prismatic joints。
2. 任务：给定 ground-truth articulated URDF，遮掉 link/joint 信息，让方法重建 link placement 和 joint prediction。
3. 阈值：position threshold 50 mm，angular threshold 0.25 radian（约 14.3 度）。
4. Baselines：URDFormer 和 Real2Code。二者只在 PartNet-Mobility 的 5 个类别上训练或微调；论文分别评估 ID 和 OOD 类。
5. VLM：主系统使用 Gemini Flash-1.5，约 20 个 in-context examples；物理计算用 PyBullet，渲染用 Sapien，视频 motion trace 用 CoTracker。
6. Robotics：用 PPO 在 Robosuite 训练 4 个细粒度操作任务，真实 Franka arm 部署。

## 主要结果

1. 摘要和贡献中报告，Articulate-Anything 将 articulation success rate 从 prior work 的 8.7% 到 12.2% 提升到 75%。
2. Baseline 对比：论文指出 Articulate-Anything 在 joint prediction 上显著优于 URDFormer 和 Real2Code，并且不区分 ID/OOD 类进行 few-shot prompting。
3. 输入模态 ablation：text、image、video 中，更 grounded 的视觉/视频输入能持续提高成功率；视频尤其能消解静态输入中的运动歧义，如窗户是 sliding 还是 tipping。
4. Failure breakdown：URDFormer 有 59.1% 输出无效，Real2Code 有 17.5% 输出无效，包括循环结构、重复 links 或语法错误。
5. Iterative refinement：critic 迭代带来约 5.8% link placement 和 2.4% joint prediction accuracy 提升。
6. Robotics：使用 Articulate-Anything 资产和人工标注资产训练的策略在真实环境中均达到 100% success rate，但自动流程平均每个任务节省 40.58 分钟人工标注时间。

## 论文贡献

1. 提出 VLM actor-critic 形式的 automatic articulation pipeline，支持文本、图像、视频输入。
2. 将 articulation 重构为 program synthesis：生成高层 Python code，再编译为 URDF。
3. 用视觉 grounded input 和 critic feedback 处理复杂、歧义、in-the-wild articulated objects。
4. 在 PartNet-Mobility 全类别评估中显著超过 prior work。
5. 展示自动生成 articulated assets 对机器人 sim-to-real 操作训练的实际价值。

## 局限性

1. 当前方法依赖 mesh retrieval 来保证高质量几何；若资产库中没有合适 mesh，几何 fidelity 会受限。
2. 集成 upstream mesh reconstruction/generation model 仍处于初步结果阶段，尚不是主系统的完全替代。
3. 依赖 VLM 能力、prompt examples 和渲染反馈；critic 会与 ground truth 高相关但也可能高估成功。
4. 作者提供 code，但本次未逐行审查实现，不能确认所有实验细节完全可复现。
5. 使用 Gemini Flash-1.5 等 VLM 可能带来闭源依赖和 API 变动风险。

## 与其他论文的关系

- 前置工作：PartNet-Mobility、Sapien、URDFormer、Real2Code、CAGE、NAP、articulated object reconstruction/generation。
- 后续工作：automatic digital twin creation、interactive 3D assets、robot simulation asset generation、VLM program synthesis for 3D。
- 相似方法：Real2Code 也使用 code generation，但依赖 oriented bounding boxes 和复现的 LLM pipeline；URDFormer 更像直接预测结构。
- 主要区别：Articulate-Anything 把 VLM 的多模态理解和 actor-critic self-refinement 放在 articulation pipeline 中，且以 URDF program 为输出。

## 与当前研究方向的关系

在 3D Scene Generation / Interaction in 3D Generation 中，Articulate-Anything 连接了“生成静态几何”和“生成可交互资产”。对本库来说，它强调 scene/object generation 不应只停留在 mesh 或 radiance field，而应包含 joint、affordance、simulation-ready behavior 等交互结构。

## 待确认问题

- [ ] 本地 PDF 下载超时，已删除 partial PDF；本次全文总结基于 arXiv HTML 和官方项目页。
- [ ] 代码仓库已确认存在，但未逐行审查实现细节。
- [ ] DOI 未在 arXiv 元数据或项目页中确认。

## 阅读记录

### 摘要总结

Articulate-Anything 通过 VLM actor-critic 自动生成 articulated object 的 URDF program。系统先检索或生成 mesh，再放置 links、预测 joints，并用 critic 对渲染/运动进行反馈修正。论文在 PartNet-Mobility 上显著超过 URDFormer/Real2Code，并展示生成资产可以支持机器人仿真训练和真实部署。

### 粗读记录

论文的核心不是从零生成 mesh，而是将“让物体可动”拆成 mesh retrieval、link placement、joint prediction 和 targeted affordance extraction。视频输入特别重要，因为许多关节运动从静态图像无法唯一判断。actor-critic 循环是系统能处理复杂和歧义对象的关键。

### 完整总结

完整阅读 arXiv HTML 后，Articulate-Anything 更像一个 VLM-driven asset-authoring system，而不是单一神经网络。它把 articulation 任务转成可调试的 Python program synthesis，用 VLM actor 写代码，用视觉 critic 查错和迭代。实验显示，基于视频/图像/文本的多模态输入和 critic refinement 对结果都有帮助；机器人实验说明 articulation 资产的价值在于可训练细粒度 manipulation，而不只是可视化。

## 用户原始备注

> 暂无。

## 引用信息

```bibtex
@inproceedings{le2025articulate,
  title={Articulate-Anything: Automatic Modeling of Articulated Objects via a Vision-Language Foundation Model},
  author={Le, Long and Xie, Jason and Liang, William and Wang, Hung-Ju and Yang, Yue and Ma, Yecheng Jason and Vedder, Kyle and Krishna, Arjun and Jayaraman, Dinesh and Eaton, Eric},
  booktitle={International Conference on Learning Representations},
  year={2025}
}
```
