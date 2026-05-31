---
title: "Representing Scenes as Neural Radiance Fields for View Synthesis"
short_name: "NeRF"
authors:
  - "Ben Mildenhall"
  - "Pratul P. Srinivasan"
  - "Matthew Tancik"
  - "Jonathan T. Barron"
  - "Ravi Ramamoorthi"
  - "Ren Ng"
year: 2020
venue: "ECCV 2020"
url: "https://arxiv.org/abs/2003.08934"
doi: "10.48550/arXiv.2003.08934"
arxiv: "2003.08934"
code_url: "https://github.com/bmild/nerf"
topics:
  - "3d-vision"
paper_type: "foundation"
task: "novel-view-synthesis"
tags:
  - literature
status: fully-summarized
summary_level: full
source_level: fulltext-and-code
priority: medium
read_date: "2026-05-31"
review_date: "2026-05-31"
related_projects:
  - "https://github.com/bmild/nerf"
  - "https://www.matthewtancik.com/nerf"
pdf_path: "Literature/PDFs/NeRF.pdf"
retrieved_from:
  - "https://arxiv.org/abs/2003.08934"
  - "https://arxiv.org/pdf/2003.08934"
  - "https://github.com/bmild/nerf"
  - "https://www.matthewtancik.com/nerf"
retrieval_date: "2026-05-31"
read_version: "arXiv 2003.08934 / ECCV 2020 version, PDF retrieved 2026-05-31"
---

# Representing Scenes as Neural Radiance Fields for View Synthesis

## 基本信息

- 简称：NeRF
- 作者：Ben Mildenhall; Pratul P. Srinivasan; Matthew Tancik; Jonathan T. Barron; Ravi Ramamoorthi; Ren Ng
- 发表时间：2020
- 会议或期刊：ECCV 2020
- 原文链接：https://arxiv.org/abs/2003.08934
- arXiv 链接：https://arxiv.org/abs/2003.08934
- DOI：10.48550/arXiv.2003.08934
- 代码仓库：https://github.com/bmild/nerf
- 项目页：https://www.matthewtancik.com/nerf
- 本地 PDF 路径：Literature/PDFs/NeRF.pdf
- 所属方向：3d-vision
- 当前状态：fully-summarized
- 实际阅读版本：arXiv 2003.08934 / ECCV 2020 version, PDF retrieved 2026-05-31

## 一句话概括

NeRF 用一个 MLP 表示连续 5D radiance field，把空间坐标和观察方向映射到密度与视角相关颜色，并通过可微体渲染从多视角图像监督训练，成为神经场景表示和新视角合成的基础范式。

## 研究问题

1. 给定稀疏多视角图片和相机位姿，如何合成高质量 novel views。
2. 传统 voxel / MPI / mesh 表示在分辨率、存储和视角一致性之间有明显瓶颈。
3. 坐标 MLP 容易偏向低频函数，难以表示精细纹理和几何，需要合适的 positional encoding 和采样策略。

## 方法概述

1. 场景表示：使用全连接 MLP $F_\Theta$ 表示连续 radiance field，输入为空间位置 $\mathbf{x}=(x,y,z)$ 和观察方向 $\mathbf{d}$，输出体密度 $\sigma$ 和颜色 $\mathbf{c}$。
2. 渲染：沿相机 ray 采样 3D 点，查询 MLP 得到每个点的颜色和密度，再用 classical volume rendering 积分得到像素颜色。
3. Positional encoding：把低维坐标映射到多频 sinusoidal features，使 MLP 能表达高频几何和纹理。
4. Hierarchical sampling：先用 coarse network 采样，再根据 coarse weights 对重要深度区间进行 fine sampling，提高采样效率。
5. 训练：只需多视角 RGB 图像和已知 camera poses，最小化渲染颜色和真实像素颜色之间的 MSE。

## 关键公式

1. Radiance field：

   $$
   F_\Theta(\mathbf{x}, \mathbf{d})
   =
   (\mathbf{c}, \sigma).
   $$

   其中 $\sigma$ 是体密度，$\mathbf{c}$ 是 view-dependent emitted radiance。

2. 连续体渲染：

   $$
   C(\mathbf{r})
   =
   \int_{t_n}^{t_f}
   T(t)\sigma(\mathbf{r}(t))\mathbf{c}(\mathbf{r}(t),\mathbf{d})dt,
   $$

   $$
   T(t)
   =
   \exp
   \left(
   -
   \int_{t_n}^{t}
   \sigma(\mathbf{r}(s))ds
   \right).
   $$

3. 离散 alpha compositing：

   $$
   \hat{C}(\mathbf{r})
   =
   \sum_i
   T_i
   \left(
   1-\exp(-\sigma_i\delta_i)
   \right)
   \mathbf{c}_i.
   $$

4. Positional encoding：

   $$
   \gamma(p)
   =
   \left(
   \sin(2^0\pi p),\cos(2^0\pi p),\ldots,
   \sin(2^{L-1}\pi p),\cos(2^{L-1}\pi p)
   \right).
   $$

5. Coarse + fine 训练损失：

   $$
   L
   =
   \sum_{\mathbf{r}}
   \left[
   \left\|
   \hat{C}_c(\mathbf{r}) - C(\mathbf{r})
   \right\|_2^2
   +
   \left\|
   \hat{C}_f(\mathbf{r}) - C(\mathbf{r})
   \right\|_2^2
   \right].
   $$

## 实验设置

1. 数据集包括 Diffuse Synthetic 360、Realistic Synthetic 360 和 Real Forward-Facing scenes。
2. 对比方法包括 SRN、Neural Volumes 和 LLFF。
3. 评价指标为 PSNR、SSIM 和 LPIPS；PSNR/SSIM 越高越好，LPIPS 越低越好。
4. 每个场景单独优化一个 NeRF，论文报告 300k iterations，在单张 NVIDIA V100 上约 1-2 天收敛。
5. 标准设置使用 64 个 coarse samples 和 128 个 additional fine samples，位置 positional encoding 频率 $L=10$。

## 主要结果

1. Diffuse Synthetic 360：NeRF PSNR 40.15、SSIM 0.991、LPIPS 0.023，超过 LLFF 的 34.38/0.985/0.048。
2. Realistic Synthetic 360：NeRF PSNR 31.01、SSIM 0.947、LPIPS 0.081，超过 SRN、NV 和 LLFF。
3. Real Forward-Facing：NeRF PSNR 26.50、SSIM 0.811、LPIPS 0.250；LLFF 的 LPIPS 0.212 略好，但 NeRF 在 PSNR/SSIM 和多视角一致性上更强。
4. Ablation 表明 positional encoding 和 view dependence 是最大增益来源，hierarchical sampling 也有明显帮助。完整模型在 realistic synthetic 平均 PSNR 31.01，去掉 positional encoding 降至 28.77，去掉 view dependence 降至 27.66。
5. NeRF 权重每场景约 5 MB，相比 LLFF 每个场景超过 15 GB 的 MPI 表示有极大存储优势。

## 论文贡献

1. 提出以坐标 MLP 表示连续 neural radiance field 的新视角合成范式。
2. 将可微体渲染和多视角图像监督结合，使场景表示无需显式 3D ground truth。
3. 引入高频 positional encoding 和 hierarchical volume sampling，使 MLP 能恢复精细几何和纹理。
4. 奠定后续 neural fields、3D reconstruction、text-to-3D 和 radiance field editing 的基础。

## 局限性

1. 每个场景需要单独优化，训练时间长，标准设置约 1-2 天，难以实时建模。
2. 渲染也需要大量 MLP 查询，虽然 hierarchical sampling 提升效率，但仍远慢于传统 rasterization。
3. 假设静态场景和准确相机位姿；动态物体、光照变化、非刚体运动需要额外建模。
4. MLP 权重不如 voxel/mesh 易解释，论文也指出难以分析 failure modes 和预期渲染质量。

## 与其他论文的关系

- 前置工作：LLFF、Neural Volumes、SRN 等为新视角合成和神经场景表示提供基线。
- 后续工作：Mip-NeRF、Instant-NGP、PlenOctrees、TensoRF、3D Gaussian Splatting 等都在效率、抗锯齿、显式结构或实时渲染上改进 NeRF。
- 相似方法：SRN 也是 implicit representation，但没有 NeRF 的体密度积分和视角相关颜色建模。
- 主要区别：NeRF 使用连续 radiance field 加体渲染，从图像监督中直接恢复可渲染 3D 表示。

## 与当前研究方向的关系

NeRF 是 3D vision 和 text-to-3D 的基础概念之一。DreamFusion 等后续方法虽然不一定直接使用原始 NeRF 训练流程，但其 3D 表示、体渲染和从多视角一致性约束中优化场景的思想都与 NeRF 深度相关。

## 待确认问题

- 无。

## 阅读记录

### 摘要总结

NeRF 用坐标 MLP 加体渲染把多视角图片转成连续场景函数。它的关键技术是 positional encoding 解决高频表达，hierarchical sampling 提升采样效率。虽然训练慢，但质量和影响力极高。

### 粗读记录

重点读了公式 5D input、volume rendering、coarse/fine sampling 和 Table 1/2。NeRF 的结果不仅是指标高，更重要的是提供了一种可微、连续、紧凑的场景表示。

### 完整总结

NeRF 将场景表示为一个函数而不是显式网格或体素。给定位置和方向，MLP 输出密度和颜色；沿相机光线积分即可得到像素颜色。由于整个渲染过程可微，模型只需 RGB 图像和相机位姿即可优化。高频 positional encoding 让 MLP 能表示细节，coarse/fine hierarchical sampling 则把采样集中到可能有物体的深度范围。

NeRF 的历史意义在于把新视角合成从工程化显式表示推进到 neural field 表示。它牺牲了训练和渲染速度，换来紧凑存储、高质量视角一致性和可微优化接口。后续大量工作围绕“如何让 NeRF 更快、更泛化、更可编辑、更适合动态场景”展开。

## 用户原始备注

> 暂无。

## 引用信息

~~~bibtex
@inproceedings{mildenhall2020nerf,
  title = {NeRF: Representing Scenes as Neural Radiance Fields for View Synthesis},
  author = {Mildenhall, Ben and Srinivasan, Pratul P. and Tancik, Matthew and Barron, Jonathan T. and Ramamoorthi, Ravi and Ng, Ren},
  booktitle = {European Conference on Computer Vision},
  year = {2020}
}
~~~

