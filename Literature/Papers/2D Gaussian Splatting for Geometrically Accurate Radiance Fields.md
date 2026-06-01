---
title: "2D Gaussian Splatting for Geometrically Accurate Radiance Fields"
short_name: "2DGS"
authors:
  - "Binbin Huang"
  - "Zehao Yu"
  - "Anpei Chen"
  - "Andreas Geiger"
  - "Shenghua Gao"
year: 2024
venue: "SIGGRAPH 2024 Conference Papers"
url: "https://arxiv.org/abs/2403.17888"
doi: "10.1145/3641519.3657428"
arxiv: "2403.17888"
code_url: "https://github.com/hbb1/2d-gaussian-splatting"
topics: 
  - "3d-vision"
paper_type: "method"
task: "radiance-field-reconstruction"
tags:
  - literature
status: fully-summarized
summary_level: full
source_level: fulltext-and-code
priority: medium
read_date: "2026-06-01"
review_date:
related_projects:
  - "https://surfsplatting.github.io/"
  - "https://github.com/hbb1/2d-gaussian-splatting"
pdf_path: "Literature/PDFs/2D Gaussian Splatting for Geometrically Accurate Radiance Fields.pdf"
retrieved_from:
  - "https://arxiv.org/abs/2403.17888"
  - "https://arxiv.org/pdf/2403.17888.pdf"
  - "https://surfsplatting.github.io/"
  - "https://github.com/hbb1/2d-gaussian-splatting"
retrieval_date: "2026-06-01"
read_version: "arXiv v3 PDF, 13 pages, dated 2025-02-22; official project and code metadata from Batch 003/004"
---


# 2D Gaussian Splatting for Geometrically Accurate Radiance Fields

## 基本信息

- 简称：2DGS
- 作者：Binbin Huang; Zehao Yu; Anpei Chen; Andreas Geiger; Shenghua Gao
- 发表时间：2024
- 会议或期刊：SIGGRAPH 2024 Conference Papers
- 原文链接：https://arxiv.org/abs/2403.17888
- arXiv：2403.17888
- DOI：10.1145/3641519.3657428
- 代码仓库：https://github.com/hbb1/2d-gaussian-splatting
- 本地 PDF 路径：Literature/PDFs/2D Gaussian Splatting for Geometrically Accurate Radiance Fields.pdf
- 所属方向：3d-vision
- 当前状态：fully-summarized
- 实际阅读版本：arXiv v3 PDF, 13 pages, dated 2025-02-22; official project and code metadata from Batch 003/004

## 一句话概括

2DGS 将 3DGS 的体积高斯替换为有方向的 2D 平面高斯盘，用显式 ray-splat intersection、深度 distortion 和 normal consistency 约束把 radiance field 更紧地贴到真实表面上，从而在保持实时新视角渲染的同时显著改善几何重建。

## 研究问题

3D Gaussian Splatting 已经能高质量、实时地做 novel view synthesis，但 3D Gaussian 是体积 blob，会在不同视角下用不同的交点/投影平面评估颜色和深度，导致深度图、多视角几何和表面重建不稳定。对于需要三角网格、法线、几何一致性或可编辑表面的任务，仅用 3DGS 的 photometric optimization 容易产生噪声表面、floaters 和薄结构错误。

作者要解决的问题是：能否保留 Gaussian splatting 的训练/渲染效率，同时让 primitive 自身更接近 surface element，并且不用 GT geometry、深度传感器或已知 lighting 就能从多视角 RGB 图像中恢复可用表面。

## 方法概述

1. 表示：每个 primitive 是嵌入 3D 空间的 2D oriented Gaussian disk，而不是完整 3D volume。一个 2D Gaussian 由中心 $p_k$、切向量 $t_u,t_v$、法向 $t_w=t_u \times t_v$、尺度 $(s_u,s_v)$、opacity 和 spherical harmonics 颜色参数组成。
2. 渲染：不用 affine perspective approximation 直接投影 2D Gaussian，而是对每个 pixel ray 显式计算 ray-splat intersection，在 splat 的局部 $uv$ 坐标中评估 Gaussian 值。
3. 稳定性：对于斜视角下退化成线段的 splat，引入 object-space low-pass filter，使 splat 至少覆盖足够像素，避免优化中被漏采样。
4. 混合：沿用 3DGS 风格的 tile-based rasterization 和 front-to-back alpha blending，输出 RGB、depth 和 normal。
5. 几何正则：加入 depth distortion loss，把同一条 ray 上的 splat intersection 深度集中起来；加入 normal consistency loss，让 splat normal 与渲染深度梯度推得的 surface normal 对齐。
6. 网格提取：从优化后的 2D disks 渲染深度图，用 TSDF fusion 从多视角深度中提取三角网格。

## 关键公式

1. 2D Gaussian disk 的世界坐标参数化：

   $$
   P(u,v)=p_k+s_u t_u u+s_v t_v v
   =
   H(u,v,1,1)^T.
   $$

   其中 $H$ 是由旋转、尺度和中心组成的 homogeneous transform。与 3DGS 不同，这个 primitive 先天定义在局部切平面上。

2. 局部 2D Gaussian 值：

   $$
   G(u)=\exp\left(-\frac{u^2+v^2}{2}\right).
   $$

3. ray-splat intersection 使用 pixel ray 的两个 homogeneous plane 与 splat 局部平面求交。论文给出的局部坐标为：

   $$
   u(x)=
   \frac{h_u^2h_v^4-h_u^4h_v^2}
   {h_u^1h_v^2-h_u^2h_v^1},
   \quad
   v(x)=
   \frac{h_u^4h_v^1-h_u^1h_v^4}
   {h_u^1h_v^2-h_u^2h_v^1}.
   $$

4. depth distortion loss：

   $$
   L_d=\sum_{i,j}\omega_i\omega_j |z_i-z_j|.
   $$

   它直接拉近同一条 ray 上被 alpha blending 赋权的 splat intersection 深度。

5. normal consistency loss：

   $$
   L_n=\sum_i \omega_i(1-n_i^T N),
   \quad
   N(x,y)=
   \frac{\nabla_x p_s \times \nabla_y p_s}
   {|\nabla_x p_s \times \nabla_y p_s|}.
   $$

   $N$ 来自渲染深度的空间梯度，$n_i$ 是面向相机的 splat normal。

## 实验设置

1. 数据集：DTU、Tanks and Temples、Mip-NeRF 360，并在 supplement 中补充 Synthetic NeRF、TnT 和 Mip-NeRF360 的更多 PSNR 表。
2. 初始化：使用 COLMAP sparse point cloud；DTU 图像从 1600x1200 下采样到 800x600。
3. 对比方法：隐式重建方法 NeRF、VolSDF、NeuS、Geo-NeuS、Neuralangelo，以及显式/并发方法 3DGS、SuGaR。
4. 训练与硬件：所有实验在单张 RTX 3090 上进行。论文报告 15k 和 30k iteration 版本，并比较 reconstruction time 和 model size。
5. 评价指标：几何重建用 Chamfer distance、accuracy/completion/F1 等；新视角合成用 PSNR、SSIM、LPIPS。

## 主要结果

1. DTU Chamfer distance，Table 1/3：2DGS-30k 平均 CD=0.80、训练 10.9 分钟、模型 52MB；2DGS-15k 平均 CD=0.83、训练 5.5 分钟。对比 3DGS 为 CD=1.96、11.2 分钟、113MB；SuGaR 为 CD=1.33、约 1 小时、1247MB。
2. Tanks and Temples，Table 2：2DGS 平均 F1=0.32，明显高于 3DGS 的 0.09 和 SuGaR 的 0.19，但低于 Neuralangelo 的 0.50；训练时间 15.5 分钟，远快于多个 SDF baseline 的 24 小时以上。
3. Mip-NeRF 360，Table 4：2DGS 的外景 PSNR/SSIM/LPIPS 为 24.34/0.717/0.246，内景为 30.40/0.916/0.195；与 3DGS 的 24.64/0.731/0.234 和 30.41/0.920/0.189 接近，说明几何增强没有显著牺牲 NVS 质量。
4. Ablation，Table 5：完整模型 accuracy/completion/average 为 0.79/0.86/0.83；去掉 normal consistency 后 average 变为 1.24，去掉 depth distortion 后为 0.88，说明两个几何正则都有效，其中 normal consistency 对法线方向尤为关键。
5. Supplement：Synthetic NeRF 平均 PSNR=33.07，与 3DGS 的 33.32 接近；TnT 平均 PSNR=25.56，略高于 3DGS 的 25.33；Mip-NeRF360 平均 PSNR=27.03，略低于 3DGS 的 27.20。

## 论文贡献

1. 提出以 2D oriented Gaussian disks 表示 radiance field surface element，使 Gaussian splatting 更适合几何重建。
2. 设计 perspective-correct ray-splat intersection，避免 3DGS 和 affine splatting 在视角变化下的几何不一致。
3. 用 depth distortion 与 normal consistency 约束缓解 photometric-only optimization 的表面噪声。
4. 证明 2DGS 可在几分钟级训练时间内提供优于 3DGS/SuGaR 的表面重建，同时保持接近 3DGS 的实时 NVS 质量。

## 局限性

作者明确指出：2DGS 假设表面局部类似 2D plane，因此对半透明表面不擅长；当前 densification 过程主要用于重建而不是压缩，模型中仍有许多小 2D Gaussians，存储效率还有提升空间。

个人分析：2DGS 是“surface-biased 3DGS”，几何更强但仍依赖多视角标定和优化式重建；它不直接解决动态场景、反射/透明材质或大规模场景压缩问题。

## 与其他论文的关系

- 前置工作：NeRF、3DGS、surface splatting、differentiable point-based graphics、Mip-NeRF360 distortion regularization。
- 后续工作：可作为 Gaussian surface reconstruction、mesh extraction、geometry-aware splatting 方法的基础节点。
- 相似方法：SuGaR 同样试图从 3DGS 中恢复 surface；NeuS/VolSDF/Neuralangelo 用 SDF 隐式场恢复表面。
- 主要区别：2DGS 直接把 primitive 变成有方向的 2D surface disk，而不是事后从 3D volume Gaussian 中提取表面。

## 与当前研究方向的关系

在本库的 3D Vision / radiance field 方向中，2DGS 是连接 3DGS 与几何重建的重要论文。它说明 Gaussian splatting 不只是快速渲染表示，也可以通过 surface-aware primitive 和几何正则转向可网格化的 scene reconstruction。

## 待确认问题

- [ ] 本次完整阅读的是 arXiv v3 PDF；若后续需要引用最终 ACM 版本页码或 supplemental 细节，应再核对 ACM Digital Library 版本。
- [ ] 官方 GitHub 代码只沿用 Batch 003/004 已确认链接，本次未逐行审查实现。

## 阅读记录

### 摘要总结

论文指出 3DGS 虽然能实现高质量新视角合成和快速渲染，但 3D Gaussians 的多视角不一致会限制表面几何准确性。2DGS 的核心思路是用 2D oriented planar Gaussian disks 内在地表示表面，并用透视校正 splatting、深度 distortion 与法线一致性约束改善薄结构和几何恢复。

### 粗读记录

粗读确认论文主线是“把 Gaussian primitive surface 化”。它没有推翻 3DGS 的 tile rasterization 和 alpha blending，而是在 primitive 建模、交点计算和训练正则上做几何约束。结果显示 DTU 几何指标大幅优于 3DGS/SuGaR，Mip-NeRF360 新视角质量基本保持。

### 完整总结

完整阅读 arXiv v3 PDF 后，2DGS 的关键价值在于给 Gaussian splatting 加入显式表面归纳偏置：每个 Gaussian 是局部 2D disk，ray-splat intersection 给出一致 depth 和 normal，depth distortion 与 normal consistency 再把 splats 收紧到真实表面附近。实验上，DTU 平均 Chamfer 从 3DGS 的 1.96 降到 0.80，同时训练时间仍为 10.9 分钟级；TnT F1 显著优于显式 3DGS/SuGaR，但仍低于 Neuralangelo。这说明 2DGS 在“快速可渲染 radiance field”和“可提取 surface mesh”之间取得了实用折中。

## 用户原始备注

> 暂无。

## 引用信息

```bibtex
@inproceedings{huang2024twodgs,
  title={2D Gaussian Splatting for Geometrically Accurate Radiance Fields},
  author={Huang, Binbin and Yu, Zehao and Chen, Anpei and Geiger, Andreas and Gao, Shenghua},
  booktitle={SIGGRAPH Conference Papers},
  year={2024},
  doi={10.1145/3641519.3657428}
}
```
