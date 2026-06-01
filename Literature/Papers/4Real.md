---
title: "4Real: Towards Photorealistic 4D Scene Generation via Video Diffusion Models"
short_name: "4Real"
authors:
  - "Heng Yu"
  - "Chaoyang Wang"
  - "Peiye Zhuang"
  - "Willi Menapace"
  - "Aliaksandr Siarohin"
  - "Junli Cao"
  - "Laszlo A. Jeni"
  - "Sergey Tulyakov"
  - "Hsin-Ying Lee"
year: 2024
venue: "NeurIPS 2024"
url: "https://arxiv.org/abs/2406.07472"
doi: ""
arxiv: "2406.07472"
code_url: ""
topics: 
  - "3d-scene-generation"
  - "video-based-generation"
paper_type: "method"
task: "text-to-4d-scene-generation"
tags:
  - literature
status: fully-summarized
summary_level: full
source_level: fulltext
priority: medium
read_date: "2026-06-01"
review_date:
related_projects:
  - "https://snap-research.github.io/4Real/"
pdf_path: "Literature/PDFs/4Real.pdf"
retrieved_from:
  - "https://arxiv.org/abs/2406.07472"
  - "https://openreview.net/pdf?id=SO1aRpwVLk"
  - "https://snap-research.github.io/4Real/"
retrieval_date: "2026-06-01"
read_version: "NeurIPS 2024 OpenReview PDF, 25 pages"
---


# 4Real: Towards Photorealistic 4D Scene Generation via Video Diffusion Models

## 基本信息

- 简称：4Real
- 作者：Heng Yu; Chaoyang Wang; Peiye Zhuang; Willi Menapace; Aliaksandr Siarohin; Junli Cao; Laszlo A. Jeni; Sergey Tulyakov; Hsin-Ying Lee
- 发表时间：2024
- 会议或期刊：NeurIPS 2024
- 原文链接：https://arxiv.org/abs/2406.07472
- arXiv：2406.07472
- DOI：
- 代码仓库：未确认公开
- 项目页：https://snap-research.github.io/4Real/
- 本地 PDF 路径：Literature/PDFs/4Real.pdf
- 所属方向：3d-scene-generation, video-based-generation
- 当前状态：fully-summarized
- 实际阅读版本：NeurIPS 2024 OpenReview PDF, 25 pages

## 一句话概括

4Real 是一个 text-to-4D scene generation pipeline：先用真实视频数据训练出的 text-to-video model 生成 reference video 和 freeze-time video，再把它们提升为 deformable 3D Gaussian Splatting，从而生成可改变视角和时间的近照片级动态场景。

## 研究问题

现有 4D generation 多依赖 image/multiview/video diffusion priors，其中 multiview model 通常由静态、合成、object-centric 3D 数据微调而来，导致生成结果偏物体中心、背景简单、照片真实感不足，并且很难表现物体与环境之间的真实交互。4Real 关注的是更接近真实视频的 text-to-4D scene generation：输入文本，输出包含动态物体、背景和光照的 4D scene，可在不同相机视角和不同时间渲染。

## 方法概述

1. 表示：使用 deformable 3D Gaussian Splatting。canonical 3DGS 表示静态参考场景，deformation field 表示每个时间点的位移和旋转。
2. 参考视频：用 text-to-video diffusion model 根据文本 prompt 生成目标动态视频，作为 4D 重建的 reference video。
3. freeze-time video：从 reference video 选定一帧作为条件，再通过 image-conditioned video generation、context embedding 和 prompt engineering 生成相机绕行但物体尽量静止的视频，用于恢复 canonical 3DGS。
4. 多视角不一致处理：由于 freeze-time video 仍可能有物体运动和几何不一致，论文把这些不一致建模为 per-frame deformation，并与 canonical 3DGS 一起联合优化。
5. canonical reconstruction：优化 RGB reconstruction loss、小运动正则项，并在后期加入 video SDS，让 canonical 3DGS 在采样相机轨迹下的渲染视频更符合视频扩散模型分布。
6. temporal reconstruction：固定 canonical 3DGS 后，从 reference video 拟合 temporal deformation；同时渲染 fixed-viewpoint video 和 freeze-time video 做 joint temporal and multi-view SDS，以兼顾时间运动和跨视角一致性。
7. 可选细节增强：将 Gaussian render 下采样再用 video model upsampler 重新上采样，作为重建目标微调 3DGS。

## 关键公式

1. deformable 3DGS 的 deformation field：

   $$
   w_{\mathrm{deform}}(x,t)=(\Delta x_t,\Delta q_t).
   $$

2. 时间 $t$ 的 Gaussian position 和 orientation：

   $$
   x_t=x+\Delta x_t,\qquad q_t=q+\Delta q_t.
   $$

3. freeze-time video 的 reconstruction loss：

   $$
   \mathcal{L}_{\mathrm{recon}}
   =
   \sum_k
   \left\lVert
   I_{\mathrm{GS}}(x+\Delta x_k,q+\Delta q_k,s,\alpha,c,P_k)-I_k
   \right\rVert_1.
   $$

4. per-frame deformation 的小运动正则：

   $$
   \mathcal{L}_{\mathrm{small}}
   =
   \sum_k
   \left(
   \left\lVert \Delta x_k \right\rVert_1
   +
   \left\lVert \Delta q_k \right\rVert_1
   \right).
   $$

5. video SDS 的简化实现：

   $$
   \mathcal{L}_{\mathrm{SDS}}
   =
   \left\lVert
   I^{k}_{\mathrm{can\mbox{-}GS}}
   -
   \left\lceil \hat{I}_k \right\rceil
   \right\rVert^2.
   $$

6. temporal deformation 的 alignment loss：

   $$
   \mathcal{L}_{\mathrm{align}}
   =
   \sum_t
   \left\lVert
   I^t_{\mathrm{GS}}-I_t
   \right\rVert
   +
   \mathrm{SSIM}(I^t_{\mathrm{GS}},I_t).
   $$

## 实验设置

1. 数据与 prompt：使用来自相关工作的 30 个文本 prompt 评估 text-to-4D generation。
2. video model：使用 Snap Video Model 及其 masked variant；SDS 中只用低分辨率 pixel-space stage，以降低显存与时间成本。
3. 表示与重建：采用 D-3DGS；camera pose 由 Colmap 初始化并在优化中微调。
4. baseline：由于没有现成 text-to-4D scene generation baseline，主要与 object-centric text-to-4D 方法 4Dfy、Dream-in-4D，以及 3D scene generation + object-centric 4D 的组合式 baseline 比较。
5. user study：Amazon Turk，每个 video pair 有 30 名 evaluator，比较 motion realism、foreground/background photo-realism、3D shape realism、general realism、motion significance、video-text alignment 等 7 个维度。
6. 自动指标：使用 X-CLIP 和 VideoScore，后者包含 visual quality、temporal consistency、dynamic degree、text-video alignment、factual consistency。

## 主要结果

1. 用户研究中，4Real 在与 4Dfy 和 Dream-in-4D 的七个维度比较里全部胜出。与 4Dfy 比较时，4Real 的 win rate 在各问题上约为 69.4% 到 77.1%；与 Dream-in-4D 比较时约为 66.9% 到 75.5%。
2. Table 1 显示，4Real 对 4Dfy 的 X-CLIP 从 20.03 提升到 24.23，Visual Quality 从 1.43 提升到 2.43，Temporal Consistency 从 1.49 提升到 2.17，T-V Alignment 从 2.26 提升到 2.91。
3. 对 Dream-in-4D，4Real 的 X-CLIP 为 24.77，而 baseline 为 19.52；Visual Quality 为 2.41，而 baseline 为 1.34；Factual Consistency 为 2.46，而 baseline 为 1.20。
4. 对 AYG，4Real 的 X-CLIP 为 23.09，高于 19.87；Temporal Consistency、Dynamic Degree、T-V Alignment 和 Factual Consistency 也更高，但 Visual Quality 略低于 AYG。
5. 组合式 baseline 需要先生成 3D background、再用 4Dfy 生成前景物体并人工插入，仍容易出现尺度、摆放、光照和背景交互不自然；4Real 的整体视频更连贯。
6. Ablation 显示 per-frame deformation、multi-view SDS、freeze-time video 和 joint temporal/multi-view SDS 都影响最终质量；去掉 freeze-time video 会系统性失败。

## 论文贡献

1. 提出面向 text-to-4D scene generation 的 4Real pipeline，减少对合成 object-centric multiview prior 的依赖。
2. 把 text-to-video diffusion model 生成的 reference video 与 freeze-time video 转化为 4D reconstruction 问题。
3. 针对 freeze-time video 的不一致，引入 per-frame deformation 与 canonical 3DGS 联合优化。
4. 在 temporal deformation 阶段同时使用 fixed-viewpoint 和 freeze-time video SDS，提升动态和跨视角一致性。
5. 相比纯 SDS 4D generation，论文报告 4Real 约 1.5 小时可生成样本，而部分竞争方法需要 10 小时以上。

## 局限性

1. 结果继承底层 video generation model 的限制，包括分辨率、快速运动时的 blur/artifact、细小物体问题和 text-video misalignment。
2. 从动态视频重建仍受 camera pose estimation、快速运动、物体突然出现/消失、突变光照等问题影响。
3. 3DGS 表示不直接产生高质量 mesh，难以满足某些需要 mesh asset 的生产流程。
4. 生成 2 秒 4D scene 仍需要超过 1 小时，交互式使用成本较高。
5. 代码仓库本次未确认公开，复现依赖 Snap Video Model 等未必开放的组件。

## 与其他论文的关系

- 前置工作：3D Gaussian Splatting、Dynamic 3DGS、video diffusion models、SDS、4Dfy、Dream-in-4D。
- 后续工作：更强视频模型驱动的 4D world generation、可编辑动态 3DGS、text-to-4D scene asset generation。
- 相似方法：4Dfy / Dream-in-4D 更偏 object-centric；4K4DGen 偏 panorama-to-4D；4Real 更强调 text prompt 到照片级 dynamic scene。
- 主要区别：4Real 不把 multiview diffusion 作为核心 3D prior，而是从真实视频风格的 video diffusion 输出中恢复 4D scene。

## 与当前研究方向的关系

在本库的 3D Scene Generation / Video-based Scene Generation 中，4Real 是从 object-centric 4D generation 迈向 dynamic scene-level generation 的关键节点。它与 4K4DGen 都使用 video prior 和 3DGS 表示，但 4Real 的输入是文本 prompt，目标是普通动态场景；4K4DGen 的输入是 4K panorama，目标是 360 度动态环境。

## 待确认问题

- [ ] 官方代码仓库未确认公开；项目页提供 demos，但本次未发现可用代码链接。
- [ ] OpenReview PDF 已可读；arXiv PDF/e-print 下载在本环境中超时，未作为阅读来源。

## 阅读记录

### 摘要总结

4Real 针对照片级 text-to-4D scene generation，核心思路是用 text-to-video model 生成 reference video，再生成相机运动但时间尽量冻结的 freeze-time video，用后者恢复 canonical 3DGS，用前者恢复 temporal deformation。论文通过 per-frame deformation、低分辨率 video SDS 与 joint temporal/multi-view SDS 处理视频先验的几何不一致和动态重建问题。

### 粗读记录

论文不是直接训练 4D generative model，也不是只做 object-centric 4D。它把预训练 video diffusion model 当作真实世界动态和外观先验，再通过 D-3DGS 把视频提升为可改变视角的 4D representation。方法依赖视频模型质量和相机估计，但相比纯 SDS 优化更快，并能生成多物体和背景更丰富的动态场景。

### 完整总结

完整阅读 NeurIPS 2024 OpenReview PDF 后，4Real 的关键在于把 4D generation 拆成“视频生成 + 4D 重建”。freeze-time video 提供多视角几何信息，但并不完美静止，因此作者把它的残余不一致当作 per-frame deformation；reference video 提供真实时间运动，再通过 temporal deformation 对齐。Video SDS 既用于 canonical 视角一致性，也用于 temporal motion regularization。实验表明，4Real 在 user preference、X-CLIP 和 VideoScore 上明显优于 object-centric 4D baselines，但复现受到非公开 video model 和代码状态限制。

## 用户原始备注

> 暂无。

## 引用信息

```bibtex
@inproceedings{yu20244real,
  title={4Real: Towards Photorealistic 4D Scene Generation via Video Diffusion Models},
  author={Yu, Heng and Wang, Chaoyang and Zhuang, Peiye and Menapace, Willi and Siarohin, Aliaksandr and Cao, Junli and Jeni, Laszlo A. and Tulyakov, Sergey and Lee, Hsin-Ying},
  booktitle={Advances in Neural Information Processing Systems},
  year={2024}
}
```
