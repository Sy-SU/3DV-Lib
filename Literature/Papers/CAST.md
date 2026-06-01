---
title: "CAST: Component-Aligned 3D Scene Reconstruction from an RGB Image"
short_name: "CAST"
authors:
  - "Kaixin Yao"
  - "Longwen Zhang"
  - "Xinhao Yan"
  - "Yan Zeng"
  - "Qixuan Zhang"
  - "Wei Yang"
  - "Lan Xu"
  - "Jiayuan Gu"
  - "Jingyi Yu"
year: 2025
venue: "arXiv 2025"
url: "https://arxiv.org/abs/2502.12894"
doi: ""
arxiv: "2502.12894"
code_url: ""
topics: 
  - "3d-scene-generation"
paper_type: "method"
task: "single-image-3d-scene-reconstruction"
tags:
  - literature
status: fully-summarized
summary_level: full
source_level: fulltext
priority: medium
read_date: "2026-06-01"
review_date:
related_projects:
  - "https://sites.google.com/view/cast4"
pdf_path: ""
retrieved_from:
  - "https://arxiv.org/abs/2502.12894"
  - "https://arxiv.org/html/2502.12894"
retrieval_date: "2026-06-01"
read_version: "arXiv v2 HTML, updated 2025-05-13"
---


# CAST: Component-Aligned 3D Scene Reconstruction from an RGB Image

## 基本信息

- 简称：CAST
- 作者：Kaixin Yao; Longwen Zhang; Xinhao Yan; Yan Zeng; Qixuan Zhang; Wei Yang; Lan Xu; Jiayuan Gu; Jingyi Yu
- 发表时间：2025
- 会议或期刊：arXiv 2025
- 原文链接：https://arxiv.org/abs/2502.12894
- arXiv：2502.12894
- DOI：
- 代码仓库：未确认公开
- 项目页：https://sites.google.com/view/cast4
- 本地 PDF 路径：
- 所属方向：3d-scene-generation
- 当前状态：fully-summarized
- 实际阅读版本：arXiv v2 HTML, updated 2025-05-13

## 一句话概括

CAST 从单张 RGB 图像重建 component-aligned 3D scene：先分解场景并独立生成每个物体的完整 mesh，再用生成式 pose alignment 和物理关系图优化物体位姿，让整体场景同时具备视觉对齐和物理合理性。

## 研究问题

单图 3D scene reconstruction 比单物体重建更难，因为场景中有多个物体、遮挡、尺度和姿态不确定，以及物体间的支撑、接触、悬挂等物理关系。已有方法常受限于特定室内数据集、检索库、低质量 object generation，或忽略物体间空间关系，导致生成结果出现浮空、穿模、尺度错配、视角外崩坏等问题。CAST 试图用 open-vocabulary scene decomposition、object-level 3D generation、pose alignment generation 和 physics-aware relation graph 组合出更可靠的单图 3D scene。

## 方法概述

1. Preprocessing：使用 2D foundation models 做 open-vocabulary recognition、localization、segmentation；使用 monocular depth estimator 得到 partial point clouds 和初始空间关系。
2. ObjectGen：一个 occlusion-aware 3D object generation module，基于 VAE/LDM 生成完整物体几何，并用 MAE 处理遮挡区域。
3. Canonical point cloud conditioning：把图像/深度估计得到的 partial point cloud 变换到 canonical space，作为 3D generation 的几何条件，缓解遮挡和尺度不稳定。
4. AlignGen：生成式 pose alignment module 输入 generated mesh latent 和 partial point cloud，生成 transformed partial point cloud，再估计 similarity transform。
5. Iterative generation：先生成每个 object，再对齐到 scene point cloud，之后 refinement。
6. Scene relation graph：用 GPT-4v 和 Set-of-Mark prompting 识别 stack、lean、hang、clamped、contained、edge/point 等细粒度关系，再映射到 support/contact graph。
7. Physics-aware correction：基于 SDF 和 relation graph 优化每个 object pose，减少穿模、浮空和不合理接触。

## 关键公式

1. 物理约束优化目标：

   $$
   \min_{\mathcal{T}=\{T_1,T_2,\ldots,T_N\}}
   \sum_{i,j}
   C(T_i,T_j;o_i,o_j).
   $$

2. contact relation 的双向成本示意：

   $$
   C(T_i,T_j)
   =
   C(T_i,T_j;o_i\rightarrow o_j)
   +
   C(T_i,T_j;o_j\rightarrow o_i).
   $$

3. support relation 的单向成本：

   $$
   C(T_i,T_j)
   =
   \left|
   \min_{p\in\partial o_j}
   D_i(p(T_j))
   \right|,
   \quad
   \mathrm{if}\ o_i\ \mathrm{supports}\ o_j.
   $$

4. flat support surface 的近接正则：

   $$
   C(T_i,T_j)
   =
   \frac{
   \sum_{p\in\partial o_j} D_i(p(T_j))\,\mathbb{I}(0<D_i(p)<\sigma)
   }{
   \sum_{p\in\partial o_j} \mathbb{I}(0<D_i(p)<\sigma)
   }.
   $$

## 实验设置

1. 定性比较：与 ACDC、Gen3DSR 等 single-image scene reconstruction 方法比较，包含 indoor/outdoor、close-up、AI-generated images 等 open-vocabulary cases。
2. 量化数据：使用 3D-FRONT 进行 object-level 和 scene-level quantitative evaluation；为公平起见，对其他方法替换为 ground-truth masks，减少 segmentation 差异影响。
3. 指标：object/scene Chamfer Distance、F-Score、scene IoU；另用 CLIP score、GPT-4 ranking、user study 的 Visual Quality 与 Physical Plausibility。
4. ObjectGen：基于 3DShape2VecSet / CLAY 风格的 VAE + LDM，24-layer transformer，总参数约 1.5B，使用 Objaverse 过滤后的约 500K assets。
5. AlignGen：24-layer transformer，feature dimension 512，约 150M parameters。
6. 消融：MAE occlusion module、partial point cloud conditioning、generative alignment、physics-aware correction / iterative refinement。

## 主要结果

1. Table 2：在 3D-FRONT 上，CAST 的 scene-level CD-S 为 0.052，FS-S 为 56.18；object-level CD-O 为 0.057，FS-O 为 56.50；IoU-B 为 0.603，均优于 ACDC、InstPIFU、Gen3DSR。
2. ACDC 的 CD-S/FS-S/IoU-B 为 0.104/39.46/0.541；Gen3DSR 为 0.083/38.95/0.459，CAST 提升明显。
3. Table 1 中，CAST 在 CLIP score、GPT-4 ranking、Visual Quality 和 Physical Plausibility 四类指标上均优于 ACDC 和 Gen3DSR。
4. MAE 消融：没有 MAE 时，遮挡区域物体会出现碎裂、缺失；MAE conditioning 能补全被遮挡区域。
5. partial point cloud conditioning：对复杂结构如不同尺寸书本堆叠，缺少 point cloud 条件会导致物体数量、尺度和细节错误。
6. AlignGen：相比 ICP 和 differentiable rendering，生成式 alignment 对 outliers、尺度不确定、对称物体和遮挡更稳健。
7. physics-aware correction：relation graph constraints 同时保持物理可行性和图像中的相对布局，避免仅靠 physics simulation 导致构图偏离原图。
8. Table 3 ablation：从 Vanilla 到 +MAE、+PCD、+iter.，CD-S 从 0.079 降至 0.052，FS-S 从 53.38 升至 56.18，IoU-B 从 0.515 升至 0.603。

## 论文贡献

1. 提出一个从单张 RGB 图像生成 component-aligned 3D scene 的 open-vocabulary pipeline。
2. 用 occlusion-aware ObjectGen 生成每个物体的完整几何，并用 partial point cloud conditioning 约束尺度和可见部分。
3. 用 AlignGen 将生成物体与 scene point cloud 对齐，避免直接 pose regression 的单峰和局部最优问题。
4. 用 GPT-4v 生成 scene relation graph，再通过 SDF constraint optimization 强化物体间物理关系。
5. 展示单图到可编辑、可仿真、可用于游戏/电影/机器人 real-to-sim 的 3D scene workflow。

## 局限性

1. 生成质量高度依赖底层 object generation model；当前几何细节和精度仍有限。
2. mesh representation 对 transparent glass、textiles、fabrics 等材质表达不自然。
3. 缺少 lighting estimation 和 background modeling；论文中视觉增强仍需手动 preset lighting 和 off-the-shelf HDR 工具。
4. 在复杂空间布局和密集物体场景中性能可能下降。
5. 项目页本次访问超时，代码仓库未确认公开。
6. 本地 PDF 下载超时，已删除 partial PDF；全文阅读来源为 arXiv HTML。

## 与其他论文的关系

- 前置工作：3DShape2VecSet、CLAY、Objaverse、ACDC、Gen3DSR、MIDI、GroundingDINO/SAM、GPT-4v relation reasoning。
- 后续工作：single-image-to-editable-3D-scene、real-to-sim dataset construction、physics-aware scene generation。
- 相似方法：ACDC 偏 retrieval，受数据库限制；Gen3DSR 使用 generation-based reconstruction，但缺少足够的 pose/physics consistency。
- 主要区别：CAST 显式把 scene 拆成 components，再分别解决 object geometry、pose alignment 和 inter-object physical relations。

## 与当前研究方向的关系

CAST 位于 image-based / component-based 3D scene generation 与 reconstruction 的交界。它不是从文本自由生成场景，而是从单张图像恢复可编辑 3D 场景；对于本库中 scene generation 主题，它强调“component alignment”和“physical relation graph”对于可用 3D scene asset 的重要性。

## 待确认问题

- [ ] 官方项目页 `https://sites.google.com/view/cast4` 本次连接超时，未能核对项目页内容。
- [ ] 未确认公开代码仓库。
- [ ] DOI 和正式发表 venue 未确认。
- [ ] 本地 PDF 下载超时，已删除 partial PDF；本次全文总结基于 arXiv HTML。

## 阅读记录

### 摘要总结

CAST 从单张 RGB 图像重建 3D scene。系统先用 2D foundation models 和 depth estimator 分解场景，再用 occlusion-aware object generation 生成每个物体，用 AlignGen 估计物体到 scene point cloud 的变换，最后用 GPT-4v 关系图和 SDF 优化修正物体间接触和支撑关系。

### 粗读记录

论文的关键不是单个模块，而是 pipeline 组合：scene decomposition、object-level generative prior、pose alignment、physical constraints 缺一不可。它面向实际可用的 mesh scene，因此很重视遮挡补全、物体尺度、scene relation 和 physical plausibility。

### 完整总结

完整阅读 arXiv HTML 后，CAST 的主要价值在于将单图 scene reconstruction 从“生成一堆看起来像的物体”推进到“物体在场景中对齐并物理合理”。ObjectGen 负责完整几何，PCD 负责可见尺度约束，AlignGen 负责 pose，relation graph 负责物理关系。消融显示这些模块逐步提升 CD/F-score/IoU。局限主要来自 object generator、材质/透明物体、光照背景和复杂场景鲁棒性。

## 用户原始备注

> 暂无。

## 引用信息

```bibtex
@article{yao2025cast,
  title={CAST: Component-Aligned 3D Scene Reconstruction from an RGB Image},
  author={Yao, Kaixin and Zhang, Longwen and Yan, Xinhao and Zeng, Yan and Zhang, Qixuan and Yang, Wei and Xu, Lan and Gu, Jiayuan and Yu, Jingyi},
  journal={arXiv preprint arXiv:2502.12894},
  year={2025}
}
```
