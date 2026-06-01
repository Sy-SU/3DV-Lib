---
title: "3D Cinemagraphy from a Single Image"
short_name: "3D Cinemagraphy"
authors:
  - "Xingyi Li"
  - "Zhiguo Cao"
  - "Huiqiang Sun"
  - "Jianming Zhang"
  - "Ke Xian"
  - "Guosheng Lin"
year: 2023
venue: "CVPR 2023"
url: "https://arxiv.org/abs/2303.05724"
doi: ""
arxiv: "2303.05724"
code_url: "https://github.com/xingyi-li/3d-cinemagraphy"
topics: 
  - "3d-scene-generation"
  - "image-based-generation"
paper_type: "method"
task: "single-image-animation"
tags:
  - literature
status: fully-summarized
summary_level: full
source_level: fulltext-and-code
priority: medium
read_date: "2026-06-01"
review_date:
related_projects:
  - "https://xingyi-li.github.io/3d-cinemagraphy/"
  - "https://github.com/xingyi-li/3d-cinemagraphy"
pdf_path: "Literature/PDFs/3D Cinemagraphy.pdf"
retrieved_from:
  - "https://arxiv.org/abs/2303.05724"
  - "https://arxiv.org/pdf/2303.05724.pdf"
  - "https://xingyi-li.github.io/3d-cinemagraphy/"
  - "https://github.com/xingyi-li/3d-cinemagraphy"
retrieval_date: "2026-06-01"
read_version: "arXiv v1 PDF, 11 pages, dated 2023-03-10; official project and code metadata from Batch 003/004"
---


# 3D Cinemagraphy from a Single Image

## 基本信息

- 简称：3D Cinemagraphy
- 作者：Xingyi Li; Zhiguo Cao; Huiqiang Sun; Jianming Zhang; Ke Xian; Guosheng Lin
- 发表时间：2023
- 会议或期刊：CVPR 2023
- 原文链接：https://arxiv.org/abs/2303.05724
- arXiv：2303.05724
- DOI：
- 代码仓库：https://github.com/xingyi-li/3d-cinemagraphy
- 本地 PDF 路径：Literature/PDFs/3D Cinemagraphy.pdf
- 所属方向：3d-scene-generation, image-based-generation
- 当前状态：fully-summarized
- 实际阅读版本：arXiv v1 PDF, 11 pages, dated 2023-03-10; official project and code metadata from Batch 003/004

## 一句话概括

3D Cinemagraphy 从单张静态图像生成既有局部动态又有相机运动的视频：它把单图转换为 feature-based layered depth images 和 feature point cloud，把 2D Eulerian motion 提升成 3D scene flow，再用双向点云位移与神经渲染合成带视差的 cinemagraph。

## 研究问题

传统 cinemagraph 或单图动画通常只在固定相机下产生 2D motion，缺少 parallax；单图 novel view synthesis/3D Photo 可以移动相机，但默认场景静态。直接串联“2D animation -> NVS”或“NVS -> 2D animation”会造成深度结构频繁变化、motion field 不一致、孔洞、flicker 或 jelly-like artifacts。

论文提出一个新任务：从单张图像生成 3D cinemagraph，也就是同时合成场景动态和相机运动，并让动态内容在不同视角下保持一致。

## 方法概述

1. 输入：单张 still image。输出：一个可沿相机轨迹观看的动态视频。
2. 运动估计：沿用 Holynski et al. 的假设，用 time-invariant constant-velocity Eulerian flow field $M$ 近似水、云、烟等常见流体运动。
3. 3D 表示：用 DPT 估计 monocular depth，再按 depth discontinuities 构造 layered depth images，并用 3D Photo 的 inpainting 填补遮挡区域；每层颜色图再编码为 feature map，形成 feature LDIs。
4. 提升到 3D：把 feature LDIs 根据深度反投影为 feature point cloud；把 2D displacement field 结合深度提升为 3D scene flow。
5. 3D symmetric animation：同时用正向 flow 和反向 flow 位移点云，让反向点云补齐正向运动产生的孔洞。
6. 神经渲染：把正向/反向 animated feature point clouds 分别 splat 到目标相机平面，按时间、alpha 和 depth 构造 blending weight，再把融合后的 feature map 和 depth map 输入 image decoder 合成目标帧。
7. 可控动画：motion estimator 可接收用户 mask 和 flow hints，从而交互式控制哪些区域运动以及运动方向。

## 关键公式

1. 常速 Eulerian flow：

   $$
   F_{t\to t+1}(\cdot)=M(\cdot).
   $$

2. Euler integration：

   $$
   x_{t+1}=x_t+M(x_t).
   $$

3. 从第 0 帧到第 $t$ 帧的递推 displacement：

   $$
   F_{0\to t}(x_0)
   =
   F_{0\to t-1}(x_0)
   +
   M(x_0+F_{0\to t-1}(x_0)).
   $$

4. 正向/反向 feature map 融合权重：

   $$
   W_t=
   \frac{(1-\frac{t}{N})\alpha_f e^{-D_f}}
   {(1-\frac{t}{N})\alpha_f e^{-D_f}+\frac{t}{N}\alpha_b e^{-D_b}}.
   $$

5. 融合后的 feature 和 depth：

   $$
   F_t=W_tF_f+(1-W_t)F_b,
   \quad
   D_t=W_tD_f+(1-W_t)D_b.
   $$

## 实验设置

1. 训练数据：使用 Holynski et al. 的流体运动短视频训练集。第一帧和预训练 optical flow 网络估计的 motion field 用于训练 motion estimator；3D Photo 生成 pseudo ground truth novel views 用于 NVS 训练。
2. 训练流程：两阶段训练。第一阶段训练 motion estimation network；第二阶段冻结 motion estimator，训练 feature extraction network 和 image decoder。
3. 网络：motion estimator 是 16 层卷积 U-Net generator，用 SPADE 替换 BatchNorm；feature extraction 和 decoder 采用 Wang et al. 的架构；训练使用 multi-scale discriminator。
4. 优化：Adam；motion network 约 120k iterations，batch size 16，generator learning rate $5\times 10^{-4}$，discriminator learning rate $2\times 10^{-3}$；animation stage 约 250k iterations，learning rate 从 $1\times 10^{-4}$ 指数衰减。
5. 硬件：单张 NVIDIA GeForce RTX 3090。
6. 评价数据：Holynski et al. validation set，包含 31 个 unique scenes、162 个 ground truth video clips。每个样本渲染 4 条 novel-view trajectories，每个样本 240 个 ground truth frames。
7. 指标：PSNR、SSIM、LPIPS；另有 108 名志愿者的 pairwise user study。

## 主要结果

1. Table 1：Ours 达到 PSNR=23.33、SSIM=0.776、LPIPS=0.197，优于所有 baseline。最强 baseline NVS -> 2D Anim. + moving average 为 PSNR=22.47、SSIM=0.718、LPIPS=0.261。
2. Table 2 user study：用户在所有 pairwise comparisons 中明显偏好本文方法。例如相对 2D Anim. -> NVS 的偏好为 87.5%，相对 NVS -> 2D Anim. 为 96.1%，相对 3D Photo 为 89.5%，相对 Holynski et al. 为 70.1%。
3. Table 3 ablation：完整模型 PSNR=23.33、SSIM=0.776、LPIPS=0.197；去掉 3D symmetric animation 后 PSNR=22.99、SSIM=0.768、LPIPS=0.199；去掉 inpainting 后 LPIPS 降到 0.216；去掉 feature representation 后 PSNR=21.50、SSIM=0.674、LPIPS=0.228。
4. 质性结果：2D animation -> NVS 容易产生 stripped flickering；NVS -> 2D animation 容易出现 jelly-like motion；naive point cloud animation 产生明显孔洞；本文方法在 validation set 和 in-the-wild photos 上更稳定。

## 论文贡献

1. 提出从单张图像生成 3D cinemagraph 的任务，把 image animation 与 novel view synthesis 结合。
2. 用 feature LDI + feature point cloud 表示动态场景，使运动和相机视角变化都在 3D 空间中处理。
3. 提出 3D symmetric animation，用正反向点云位移缓解运动产生的 holes。
4. 支持 mask 和 flow hints，实现更可控的局部动画。
5. 在定量指标和 user study 上验证比直接组合 2D animation / 3D Photo / point cloud animation 更好。

## 局限性

作者明确指出：方法依赖单目深度估计，薄结构或错误深度会导致失败；不合适的 motion field 可能把应冻结区域误判为运动；当前主要面向水、烟、云等常见流体或准流体动态，不适合复杂循环运动。

个人分析：这是一篇 image-based scene generation 的早期方法，优势是从单图做可移动视角动态展示，但动态模型仍是经验性的 Eulerian flow + feature point cloud，不是物理仿真或真实 4D 重建。

## 与其他论文的关系

- 前置工作：Holynski et al. 的 single-image animation，3D Photo，DPT，layered depth images，differentiable point rendering。
- 后续工作：4K4DGen 等 panoramic/4D generation 方法把类似“单图动态 + 3D 表示”的目标扩展到 360 度和 4D Gaussian。
- 相似方法：3D Photo 解决单图 novel view synthesis，但不做动态；2D cinemagraph 方法做动态但不支持 camera motion。
- 主要区别：本文把 motion lifting 到 3D scene flow，并通过 feature point cloud 和双向动画同时处理运动与视差。

## 与当前研究方向的关系

在本库中，3D Cinemagraphy 是 image-based scene generation 和 video/4D generation 之间的桥梁。它说明“从单图生成可浏览动态场景”可以先用 2D prior 估计 motion/depth，再提升到 3D 表示；后续 4K4DGen 则用 diffusion prior 和 dynamic Gaussians 扩展到 panoramic 4D。

## 待确认问题

- [ ] 本次完整阅读的是 arXiv v1 PDF；如需引用 CVPR OpenAccess 最终 PDF 的页码或 supplemental，应后续核对。
- [ ] 官方 GitHub 代码只沿用 Batch 003/004 已确认链接，本次未逐行审查实现。

## 阅读记录

### 摘要总结

论文目标是将单图动画与 3D 摄影结合，从一张输入图像生成既有视觉动态又有相机运动的视频。流程包括：预测深度并构建 feature-based layered depth images，反投影为 feature point cloud；估计 2D motion 并提升为 3D scene flow；对点云做双向位移，再投影和融合以合成目标视角。

### 粗读记录

粗读确认最关键的工程选择是 feature point cloud，而不是 RGB point cloud。作者用 feature representation 和 decoder 避免直接点云渲染的颜色/孔洞问题；3D symmetric animation 则用反向运动补正向运动留下的空洞。

### 完整总结

完整阅读 arXiv v1 PDF 后，3D Cinemagraphy 的定位很清楚：它不是通用 4D 重建，而是一个面向单图的动态新视角合成 pipeline。论文把传统 cinemagraph 的 2D motion 提升到 feature point cloud 上，用 DPT/LDI/3D Photo 建立可移动相机的 3D workspace，再通过正反向点云融合生成 looping animation。Table 1 和 Table 2 支持作者结论：这种 3D-space joint treatment 明显优于简单串联 2D animation 和 NVS。

## 用户原始备注

> 暂无。

## 引用信息

```bibtex
@inproceedings{li2023cinemagraphy,
  title={3D Cinemagraphy from a Single Image},
  author={Li, Xingyi and Cao, Zhiguo and Sun, Huiqiang and Zhang, Jianming and Xian, Ke and Lin, Guosheng},
  booktitle={Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition},
  year={2023}
}
```
