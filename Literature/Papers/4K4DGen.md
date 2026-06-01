---
title: "4K4DGen: Panoramic 4D Generation at 4K Resolution"
short_name: "4K4DGen"
authors:
  - "Renjie Li"
  - "Panwang Pan"
  - "Bangbang Yang"
  - "Dejia Xu"
  - "Shijie Zhou"
  - "Xuanyang Zhang"
  - "Zeming Li"
  - "Achuta Kadambi"
  - "Zhangyang Wang"
  - "Zhengzhong Tu"
  - "Zhiwen Fan"
year: 2024
venue: "arXiv 2024"
url: "https://arxiv.org/abs/2406.13527"
doi: ""
arxiv: "2406.13527"
code_url: ""
topics: 
  - "3d-scene-generation"
  - "video-based-generation"
paper_type: "method"
task: "scene-generation"
tags:
  - literature
status: fully-summarized
summary_level: full
source_level: fulltext
priority: medium
read_date: "2026-06-01"
review_date:
related_projects: []
pdf_path: "Literature/PDFs/4K4DGen.pdf"
retrieved_from:
  - "https://arxiv.org/abs/2406.13527"
  - "https://arxiv.org/pdf/2406.13527.pdf"
retrieval_date: "2026-06-01"
read_version: "arXiv v3 PDF, 17 pages, dated 2024-10-03"
---


# 4K4DGen: Panoramic 4D Generation at 4K Resolution

## 基本信息

- 简称：4K4DGen
- 作者：Renjie Li; Panwang Pan; Bangbang Yang; Dejia Xu; Shijie Zhou; Xuanyang Zhang; Zeming Li; Achuta Kadambi; Zhangyang Wang; Zhengzhong Tu; Zhiwen Fan
- 发表时间：2024
- 会议或期刊：arXiv 2024
- 原文链接：https://arxiv.org/abs/2406.13527
- arXiv：2406.13527
- DOI：
- 代码仓库：未公开
- 本地 PDF 路径：Literature/PDFs/4K4DGen.pdf
- 所属方向：3d-scene-generation, video-based-generation
- 当前状态：fully-summarized
- 实际阅读版本：arXiv v3 PDF, 17 pages, dated 2024-10-03

## 一句话概括

4K4DGen 从单张 4096x2048 panorama 生成可交互浏览的 4D panoramic scene：先用 Panoramic Denoiser 把预训练 2D image-to-video diffusion prior 迁移到球面 latent 上生成一致的 panoramic video，再用 Dynamic Panoramic Lifting 将视频提升为 time-dependent 3D Gaussians。

## 研究问题

VR/AR 内容需要 6-DoF、360 度、高分辨率、动态的环境资产。现有 image/video/3D/4D generation 方法多面向 2D 显示、object-centric 4D、低分辨率动态物体，或者从单 perspective image outpaint，不能直接满足全景 4K 动态环境的实时漫游需求。核心难点有两个：高质量 panoramic 4D 数据稀缺；现有 2D diffusion/video model 主要训练在窄 FoV perspective image 上，直接用于 equirectangular panorama 会有畸变、跨视角 motion 不一致和显存/分辨率问题。

## 方法概述

1. 输入：单张静态 4K panorama，可通过用户交互或 mask 指定动态区域。输出：可在不同时间戳和视角渲染的 dynamic Gaussian Splatting scene。
2. 总体 pipeline：先生成 panoramic video，再将视频提升为一组 time-dependent 3D Gaussians。
3. spherical representation：把 panorama 从 equirectangular matrix 转为定义在球面 $S^2$ 上的 continuous field，减轻平面投影在极区的畸变。
4. Panoramic Denoiser：把球面 latent 在每个 denoising step 投影为多个 perspective latent，用预训练 perspective video denoiser 各自 denoise 一步，再融合回 spherical latent。这样既利用窄 FoV video prior，又保持 overlapping views 的一致性。
5. Dynamic Panoramic Lifting：把 panoramic video 切成多个 perspective videos，使用 monocular depth estimator 估计深度，并通过 Spatial-Temporal Geometry Alignment 融合成每一帧一致的 panoramic depth。
6. 4D representation：每个时间戳对应一组 3D Gaussians；优化时使用 RGB loss、temporal regularization、semantic distillation loss 和 geometric regularization，增强时空一致性。

## 关键公式

1. LDM forward process：

   $$
   q(x_t \mid x_{t-1})
   =
   \mathcal{N}(x_t;\sqrt{1-\beta_t}x_{t-1},\beta_t I).
   $$

2. LDM reverse process：

   $$
   p_\theta(x_{t-1}\mid x_t)
   =
   \mathcal{N}(\mu_\theta(x_t,t),\Sigma_\theta(x_t,t)).
   $$

3. equirectangular to spherical mapping：

   $$
   S_I(x,y,z)
   =
   E_I
   \left(
   \frac{1}{\pi}\arccos \frac{y}{\sqrt{1-z^2}},
   \frac{2}{\pi}\arcsin z
   \right).
   $$

4. perspective projection from panorama：

   $$
   E_P(x,y)=S_I \circ r(x,y,f,u,s,R).
   $$

5. Panoramic Denoiser 的 project-and-fuse step：

   $$
   \Psi(S_t,I)
   =
   \arg\min_S
   \mathbb{E}_{d\in S^2}
   \left\|
   \gamma(S,d)
   -
   \Phi(\gamma(S_t,d),\gamma(I,d))
   \right\|.
   $$

6. 4D Gaussian optimization：

   $$
   L=
   \lambda_{\mathrm{rgb}}L_{\mathrm{rgb}}
   +
   \lambda_{\mathrm{temporal}}L_{\mathrm{temporal}}
   +
   \lambda_{\mathrm{sem}}L_{\mathrm{sem}}
   +
   \lambda_{\mathrm{geo}}L_{\mathrm{geo}}.
   $$

## 实验设置

1. perspective views：均匀选取球面上 20 个方向，每个 perspective camera FoV 为 80 度，分辨率 512x512。
2. animator：使用 Animate-Anything 作为 perspective denoiser，其基于 SVD fine-tuning。
3. depth estimator：使用 MiDaS 进行 Spatial-Temporal Geometry Alignment。
4. 硬件：所有实验在单张 NVIDIA A100 80GB 上执行。
5. 数据：由于任务新颖，没有现成 4D panorama dataset；作者使用 text-to-panorama diffusion model 生成 16 个 panorama 作为评估集。静态 panorama 原始分辨率 6144x3072，双线性下采样到 4096x2048。
6. baseline：主要比较 3D-Cinemagraphy 的 zoom-in 和 circle 模式；SDS-based 4D 方法因多为 object-centric，不支持 outward-facing scene generation，被排除。
7. 指标：FID、KID、Q-Align image quality/aesthetic/video quality、user choice、user agreement。
8. user study：42 名用户，两组问卷共 84 份；Quality UC 共有 336 个选择问题，View-Consistency UA 共有 168 对视频判断。

## 主要结果

1. Table 1：4K4DGen 对比 3D-Cinemagraphy 明显更好。FID=16.59、KID=0.2、Q-Align IQ=0.66、IA=0.44、VQ=0.62、Quality UC=81%。3D-Cinemagraphy zoom-in 的 FID=57.05、KID=1.4、UC=7%；circle 的 FID=57.78、KID=1.3、UC=12%。
2. user study：在 336 个质量选择中，4K4DGen 被选中 272 次，3D-Cinemagraphy circle 40 次，zoom-in 24 次。
3. Table 2 animation ablation：Animate Pano 最大分辨率 2048x1024、Motion=0.30；Animate Perspective 支持 4096x2048 且 Motion=0.91，但 View-consistency UA 只有 33%；Ours 支持 4096x2048，Q-Align VQ=0.85、Motion=1.27、View-consistency UA=70%。
4. lifting ablation：去掉 temporal loss 会出现 frame artifacts/flicker；去掉 Spatial-Temporal Geometry Alignment 会降低几何结构质量。
5. 定性结果：相比 3D-Cinemagraphy，4K4DGen 在多个 view 和 timestamp 下能保持更好的空间/时间一致性，减少中间帧 ghosting artifacts。

## 论文贡献

1. 提出第一个面向 4K omnidirectional 4D assets 的 generation framework，不依赖 annotated 4D data。
2. 提出 Panoramic Denoiser，把预训练 2D perspective video diffusion prior 迁移到 panoramic spherical latent space。
3. 提出 Dynamic Panoramic Lifting，将 dynamic panoramic video 转成 dynamic Gaussians，并用 spatial-temporal regularization 保持跨帧一致。
4. 在 4K panorama-to-4D 任务上给出 baseline 比较、用户研究和 ablation。

## 局限性

作者明确指出：生成动态质量主要受预训练 I2V model 能力限制；由于 4D lifting 强调时空连续性，目前不能合成环境中的大幅变化，例如发光萤火虫或天气变化；高分辨率、时间相关的 4D representation 存储成本很高，未来需要 distillation、pruning 等压缩技术。

伦理与可复现性方面，作者指出该技术可用于 AR/VR、电影和游戏，但也可能被滥用来生成欺骗性内容或造成隐私风险；论文说提供了足够实现细节，并表示未来会 release code。

## 与其他论文的关系

- 前置工作：LDM/SVD/Animate-Anything、3DGS、4D Gaussian Splatting、3D Cinemagraphy、panorama generation、MiDaS depth estimation。
- 后续工作：panoramic 4D generation、world model、VR/AR asset generation、dynamic Gaussian scene generation。
- 相似方法：3D Cinemagraphy 从单图生成动态与相机运动，但不是 360 度 4K panorama，也不用 diffusion panoramic denoising；object-centric text-to-4D 方法不适合 outward-facing scene。
- 主要区别：4K4DGen 用 spherical latent denoising 解决 panoramic video 一致性，再用 dynamic Gaussians 支持 6-DoF 4D 浏览。

## 与当前研究方向的关系

在本库 3D Scene Generation / Video-based Scene Generation 中，4K4DGen 是从单图/全景到动态 4D scene 的关键节点。它把 2D video diffusion prior、panorama representation 和 Gaussian Splatting 组合起来，是 3D Cinemagraphy 到 panoramic 4D generation 的自然后继。

## 待确认问题

- [ ] 本次完整阅读的是 arXiv v3 PDF；正式发表 venue/DOI 未确认。
- [ ] 论文表示未来会发布代码，本次没有确认可用官方代码仓库。

## 阅读记录

### 摘要总结

4K4DGen 旨在从单张 4K panorama 生成动态、可浏览的 4D environment。摘要中最关键的两部分是 Panoramic Denoiser 和 Dynamic Panoramic Lifting：前者把 2D diffusion/video prior 迁移到球面全景视频生成，后者把 panoramic video 转成 time-dependent Gaussian representation，并用时空几何约束保持一致性。

### 粗读记录

粗读确认论文主要针对 VR/AR immersive environment，而不是 object-level 4D generation。作者没有训练专门 4D panorama model，而是把已有 perspective I2V denoiser 包装成球面 latent 的 project-and-fuse denoiser，再用 depth/3DGS 将视频提升为可渲染动态场景。

### 完整总结

完整阅读 arXiv v3 PDF 后，4K4DGen 的核心是“借用 2D prior，但在球面上融合”。直接 animate panorama 会受分辨率和畸变影响，逐 perspective view animate 又会跨视角不一致；Panoramic Denoiser 在每一步 denoising 中投影多个 view、各自 denoise、再融合回 spherical latent，解决了这一矛盾。Dynamic Panoramic Lifting 则把视频帧和 depth 估计转成 dynamic Gaussians。实验显示，和 3D Cinemagraphy 相比，4K4DGen 在 FID/KID/Q-Align/user choice 上明显占优，并且 ablation 支持 spherical denoising、STA 和 temporal loss 的必要性。

## 用户原始备注

> 暂无。

## 引用信息

```bibtex
@article{li2024k4dgen,
  title={4K4DGen: Panoramic 4D Generation at 4K Resolution},
  author={Li, Renjie and Pan, Panwang and Yang, Bangbang and Xu, Dejia and Zhou, Shijie and Zhang, Xuanyang and Li, Zeming and Kadambi, Achuta and Wang, Zhangyang and Tu, Zhengzhong and Fan, Zhiwen},
  journal={arXiv preprint arXiv:2406.13527},
  year={2024}
}
```
