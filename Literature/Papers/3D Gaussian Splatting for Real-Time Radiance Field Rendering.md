---
title: "3D Gaussian Splatting for Real-Time Radiance Field Rendering"
short_name: "3DGS"
authors:
  - "Bernhard Kerbl"
  - "Georgios Kopanas"
  - "Thomas Leimkühler"
  - "George Drettakis"
year: 2023
venue: "ACM Transactions on Graphics 42(4), SIGGRAPH 2023"
url: "https://arxiv.org/abs/2308.04079"
doi: ""
arxiv: "2308.04079"
code_url: "https://github.com/graphdeco-inria/gaussian-splatting"
topics: 
  - "3d-vision"
paper_type: "method"
task: "novel-view-synthesis"
tags:
  - literature
status: fully-summarized
summary_level: full
source_level: fulltext-and-code
priority: medium
read_date: "2026-06-01"
review_date:
related_projects:
  - "https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/"
  - "https://github.com/graphdeco-inria/gaussian-splatting"
pdf_path: "Literature/PDFs/3D Gaussian Splatting for Real-Time Radiance Field Rendering.pdf"
retrieved_from:
  - "https://arxiv.org/abs/2308.04079"
  - "https://arxiv.org/pdf/2308.04079.pdf"
  - "https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/"
  - "https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/3d_gaussian_splatting_low.pdf"
  - "https://github.com/graphdeco-inria/gaussian-splatting"
retrieval_date: "2026-06-01"
read_version: "INRIA low-resolution author PDF, 14 pages, ACM TOG/SIGGRAPH 2023 version; official project and code metadata from Batch 003/004"
---


# 3D Gaussian Splatting for Real-Time Radiance Field Rendering

## 基本信息

- 简称：3DGS
- 作者：Bernhard Kerbl; Georgios Kopanas; Thomas Leimkühler; George Drettakis
- 发表时间：2023
- 会议或期刊：ACM Transactions on Graphics 42(4), SIGGRAPH 2023
- 原文链接：https://arxiv.org/abs/2308.04079
- arXiv：2308.04079
- DOI：
- 代码仓库：https://github.com/graphdeco-inria/gaussian-splatting
- 本地 PDF 路径：Literature/PDFs/3D Gaussian Splatting for Real-Time Radiance Field Rendering.pdf
- 所属方向：3d-vision
- 当前状态：fully-summarized
- 实际阅读版本：INRIA low-resolution author PDF, 14 pages, ACM TOG/SIGGRAPH 2023 version; official project and code metadata from Batch 003/004

## 一句话概括

3DGS 用可优化的各向异性 3D Gaussians 表示 radiance field，并用 visibility-aware tile-based splatting rasterizer 实现 1080p 级实时新视角渲染，在接近最快 NeRF 方法训练时间的同时达到接近或优于 Mip-NeRF360 的视觉质量。

## 研究问题

NeRF 类方法可以产生高质量 novel view synthesis，但训练和渲染通常依赖昂贵的 volumetric ray marching。Plenoxels、Instant-NGP 等快速方法缩短训练时间，但在真实无界场景、高分辨率和最高质量上仍与 Mip-NeRF360 有差距；Mip-NeRF360 质量强但训练可达数十小时、渲染远非实时。

论文要解决的问题是：能否从多视角图像和 SfM camera calibration 出发，学习一种显式、可微、可快速 rasterize 的 scene representation，同时兼顾训练速度、视觉质量和实时渲染。

## 方法概述

1. 输入：静态场景多视角图像、相机参数，以及 SfM 过程附带产生的 sparse point cloud。
2. 表示：每个点初始化为 3D Gaussian，参数包括中心 $\mu$、opacity $\alpha$、各向异性 covariance、以及用 spherical harmonics 表示的 view-dependent color。
3. covariance 参数化：不直接优化 covariance matrix，而是用 scale vector $s$ 和 quaternion $q$ 表达旋转/尺度，保证 covariance 有效。
4. 优化：重复 render 当前 Gaussians、与训练视图比较、反向传播更新 position、opacity、scale/rotation 和 SH coefficients。
5. adaptive density control：每 100 iterations 根据 view-space position gradient densify；对小 Gaussian clone，对大 Gaussian split；删除近似透明的 Gaussians。
6. 渲染：把 3D Gaussian 投影为 2D splat，按 screen tile 分桶、排序，再做 front-to-back alpha blending。backward pass 只存 final accumulated opacity，并反向遍历恢复中间系数以计算梯度。
7. 实现：PyTorch 优化框架 + 自定义 CUDA rasterization kernels + NVIDIA CUB radix sort + SIBR interactive viewer。

## 关键公式

1. 3D Gaussian：

   $$
   G(x)=\exp\left(-\frac{1}{2}(x-\mu)^T\Sigma^{-1}(x-\mu)\right).
   $$

2. 投影到 camera/image space 的 covariance：

   $$
   \Sigma'=J W \Sigma W^T J^T.
   $$

   $W$ 是 viewing transform，$J$ 是 projective transform 的 Jacobian affine approximation。

3. 优化中使用 scale/rotation 构造 covariance：

   $$
   \Sigma=RSS^TR^T.
   $$

4. photometric loss：

   $$
   L=(1-\lambda)L_1+\lambda L_{\mathrm{D\text{-}SSIM}},
   \quad \lambda=0.2.
   $$

5. alpha blending 的思想与 NeRF image formation 类似，但实现为排序 splats 的 rasterization，而不是沿 ray 采样 MLP。

## 实验设置

1. 数据集：Mip-NeRF360 全部真实场景，Tanks and Temples 两个场景，Deep Blending 两个场景，以及 Synthetic NeRF/Blender。
2. 对比方法：Mip-NeRF360 作为质量基准；InstantNGP Base/Big 和 Plenoxels 作为快速方法基准。
3. 训练/测试划分：真实场景按 Mip-NeRF360 建议每 8 张取 1 张为 test view。
4. 指标：SSIM、PSNR、LPIPS、训练时间、FPS、模型内存。
5. 硬件：真实场景结果在 A6000 GPU 上报告；Mip-NeRF360 训练时间按作者自己的 A100 运行折算为 48 小时单 GPU。
6. 配置：报告 Ours-7K 和 Ours-30K 两种迭代数；Synthetic NeRF 从 100K 随机点初始化。

## 主要结果

1. Table 1 真实场景：Ours-30K 在 Mip-NeRF360 上 SSIM=0.815、PSNR=27.21、LPIPS=0.214、训练 41m33s、134 FPS、734MB；Mip-NeRF360 为 SSIM=0.792、PSNR=27.69、LPIPS=0.237、训练 48h、0.06 FPS、8.6MB。
2. Table 1 Tanks and Temples：Ours-30K 为 SSIM=0.841、PSNR=23.14、LPIPS=0.183、训练 26m54s、154 FPS、411MB。
3. Table 1 Deep Blending：Ours-30K 为 SSIM=0.903、PSNR=29.41、LPIPS=0.243、训练 36m2s、137 FPS、676MB。
4. Ours-7K：在 4-7 分钟训练内已经达到有竞争力的质量，例如 Mip-NeRF360 上 PSNR=25.60、160 FPS，说明早期迭代即可得到可用表示。
5. Synthetic NeRF，Table 2：Ours-30K 平均 PSNR=33.32，略高于 Mip-NeRF 的 33.09，并在 synthetic scenes 上达到 180-300 FPS。
6. Ablation，Table 3：Full 在 5K/30K 平均 PSNR 为 23.90/26.05；去掉 split、clone、SH、anisotropy 或限制 backward depth complexity 都会下降。Limited-BW 在 Truck 上可掉约 11dB，说明无限制深度复杂度的梯度传播很重要。

## 论文贡献

1. 提出 anisotropic 3D Gaussians 作为 unstructured explicit radiance field representation。
2. 提出交替优化和 adaptive density control，从 SfM sparse points 生成密集、高质量 Gaussian representation。
3. 实现 visibility-aware, differentiable, tile-based Gaussian rasterizer，使训练和 1080p 新视角渲染都显著加速。
4. 证明显式 splatting representation 可以达到或接近最强 NeRF 质量，同时提供真正实时渲染。

## 局限性

作者明确指出：未充分观测区域仍会产生 artifacts；各向异性 Gaussians 有时出现 elongated 或 splotchy artifacts；large Gaussians 可能造成 popping；简单 guard band culling 和 visibility ordering 会导致 artifacts；当前没有 regularization，未来可用于缓解 unseen regions 和 popping；超大城市级场景可能需要调低 position learning rate。

个人分析：3DGS 的强项是 fast optimization + real-time rendering，不是表面几何准确性、压缩率或动态建模。后续 2DGS/SuGaR 等方法正是针对表面重建和 mesh extraction 补足这一点。

## 与其他论文的关系

- 前置工作：NeRF、Mip-NeRF360、Plenoxels、Instant-NGP、point-based rendering、Pulsar、ADOP。
- 后续工作：2DGS、SuGaR、dynamic 3DGS/4DGS、Gaussian editing、Gaussian compression、text-to-3D/4D Gaussian generation。
- 相似方法：Plenoxels/Instant-NGP 也是加速 radiance field；NeRF/Mip-NeRF360 用隐式连续场和 ray marching。
- 主要区别：3DGS 是显式 primitive + rasterization route，不再以 MLP ray marching 为核心渲染路径。

## 与当前研究方向的关系

3DGS 是本库 3D Vision 与 3D/4D generation 的关键基础论文。它提供了后续大量 scene generation、dynamic scene rendering、4D generation、Gaussian editing 方法的底层表示和渲染范式；2DGS 则是在它的基础上加入 surface-aware geometry bias。

## 待确认问题

- [ ] 本次完整阅读的是 INRIA low-resolution author PDF；该 PDF 的 front matter 中 DOI 字段仍是占位符，正式 DOI 后续需从 ACM DL 或官方出版页核对。
- [ ] 官方 GitHub 代码只沿用 Batch 003/004 已确认链接，本次未逐行审查实现。

## 阅读记录

### 摘要总结

论文针对 NeRF 类 radiance field 方法训练和渲染成本高、实时性不足的问题，提出以 3D Gaussians 作为显式场景表示。三个关键点是：从相机标定得到的稀疏点初始化并优化 3D Gaussians；通过交替优化和密度控制，尤其是各向异性协方差优化，提高场景表示精度；使用 visibility-aware 渲染算法加速训练和实时渲染。

### 粗读记录

粗读确认 3DGS 的核心不是“高斯”单独一个选择，而是表示、densification 和 rasterizer 的组合。anisotropic covariance 保证 compactness，clone/split 保证 coverage，tile sorting + alpha blending 保证速度和可微性。

### 完整总结

完整阅读 INRIA author PDF 后，3DGS 的主要贡献可以概括为把 radiance field 从 neural ray marching 转为 explicit differentiable rasterization。它保留连续体积表示的优化便利性，又利用 GPU rasterization 达到 100+ FPS。实验显示 Ours-30K 在多个真实数据集上接近或超过 Mip-NeRF360 质量，但训练从 48 小时级缩短到 30-40 分钟级，渲染从约 0.1 FPS 提升到百 FPS 级。它的限制也很清楚：几何不是 surface-first，未观测区域和 depth/blending ordering 仍可能出问题。

## 用户原始备注

> 暂无。

## 引用信息

```bibtex
@article{kerbl2023gaussian,
  title={3D Gaussian Splatting for Real-Time Radiance Field Rendering},
  author={Kerbl, Bernhard and Kopanas, Georgios and Leimkuehler, Thomas and Drettakis, George},
  journal={ACM Transactions on Graphics},
  volume={42},
  number={4},
  year={2023}
}
```
