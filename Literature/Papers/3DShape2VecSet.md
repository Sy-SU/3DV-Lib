---
title: "A 3D Shape Representation for Neural Fields and Generative Diffusion Models"
short_name: "3DShape2VecSet"
authors:
  - "Biao Zhang"
  - "Jiapeng Tang"
  - "Matthias Niessner"
  - "Peter Wonka"
year: 2023
venue: "arXiv 2023"
url: "https://arxiv.org/abs/2301.11445"
doi: ""
arxiv: "2301.11445"
code_url: "https://1zb.github.io/3DShape2VecSet/"
topics: 
  - "mesh-generation"
paper_type: "method"
task: "mesh-generation"
tags:
  - literature
status: fully-summarized
summary_level: full
source_level: fulltext
priority: medium
read_date: "2026-06-01"
review_date:
related_projects:
  - "https://1zb.github.io/3DShape2VecSet/"
pdf_path: "Literature/PDFs/3DShape2VecSet.pdf"
retrieved_from:
  - "https://arxiv.org/abs/2301.11445"
  - "https://arxiv.org/pdf/2301.11445.pdf"
retrieval_date: "2026-06-01"
read_version: "arXiv v3 PDF, 16 pages, dated 2023-05-01; project/code URL observed inside PDF"
---


# A 3D Shape Representation for Neural Fields and Generative Diffusion Models

## 基本信息

- 简称：3DShape2VecSet
- 作者：Biao Zhang; Jiapeng Tang; Matthias Niessner; Peter Wonka
- 发表时间：2023
- 会议或期刊：arXiv 2023
- 原文链接：https://arxiv.org/abs/2301.11445
- arXiv：2301.11445
- DOI：
- 代码/项目页：https://1zb.github.io/3DShape2VecSet/
- 本地 PDF 路径：Literature/PDFs/3DShape2VecSet.pdf
- 所属方向：mesh-generation
- 当前状态：fully-summarized
- 实际阅读版本：arXiv v3 PDF, 16 pages, dated 2023-05-01; project/code URL observed inside PDF

## 一句话概括

3DShape2VecSet 为 3D neural fields 设计了一种固定长度 latent set 表示，用 cross-attention 从点云/表面编码形状，用 self-attention 和 cross-attention 解码 occupancy，并在压缩 latent set 上训练 diffusion model，从而支持无条件、类别条件、文本条件、点云补全和单图条件的 3D shape generation。

## 研究问题

2D diffusion model 的成功很大程度依赖规则图像网格或压缩 latent space；3D 形状则有 voxel、point cloud、mesh、neural field 等多种表示。Voxel/grid latent 对生成模型太大，单个 global latent 又难保留局部细节，irregular grid latent 仍显式依赖坐标和手工空间结构。作者关注的问题是：如何为 3D neural fields 学到一种紧凑、固定大小、适合 transformer 和 diffusion model 的 shape latent representation。

## 方法概述

1. 总体路线：两阶段 latent diffusion。第一阶段训练 3D shape autoencoder，把表面/点云编码为 latent set；第二阶段在压缩 latent set 上训练 diffusion model。
2. 表示设计：从 RBF 和 3DILG 的“value + similarity”插值思想出发，去掉显式 anchor coordinate，把 shape 表示为固定长度 latent vectors $\{f_i\}_{i=1}^{M}$。
3. 编码：从 surface mesh 采样点云，经过 positional embedding 后，用 cross-attention 把大点云聚合成较小 latent set。作者讨论 learnable query set 和 point query 两种方案。
4. KL regularization：把每个 latent 线性投影到 mean/variance 并采样压缩 latent $z_i$，形成 VAE-style bottleneck；该步骤对第二阶段 diffusion training 必要。
5. 解码：latent set 经过 self-attention blocks 交换信息；对任意 query point $x$，用 cross-attention 计算 interpolated feature，再用 fully connected layer 预测 occupancy。
6. 形状生成：在 compressed latent set $\{z_i\}$ 上训练 EDM 风格 denoiser。无条件生成用 self-attention set denoiser；条件生成在 denoising layer 中加入 cross-attention 注入 category、text、image 或 partial point cloud condition。
7. 表面重建：在 $128^3$ query grid 上预测 occupancy，并用 Marching Cubes 提取 surface。

## 关键公式

1. RBF 表示起点：

   $$
   \hat{O}_{\mathrm{RBF}}(x)=
   \sum_{i=1}^{M}\lambda_i\phi(x,x_i).
   $$

2. 3DILG 风格 coordinate-dependent latent：

   $$
   f_x=
   \sum_{i=1}^{M}
   f_i
   \frac{\phi(x,x_i)}
   {Z(x,\{x_i\}_{i=1}^{M})}.
   $$

3. 3DShape2VecSet 去掉显式 coordinate anchor，用 attention similarity：

   $$
   \hat{F}(x)=
   \sum_{i=1}^{M}
   v(f_i)
   \frac{
   \exp(q(x)^T k(f_i)/\sqrt{d})
   }
   {Z(x,\{f_i\}_{i=1}^{M})}.
   $$

4. occupancy prediction：

   $$
   \hat{O}(x)=\mathrm{FC}(\hat{F}(x)).
   $$

5. VAE bottleneck：

   $$
   z_{i,j}=\mu_{i,j}+\sigma_{i,j}\epsilon,
   \quad
   \epsilon \sim \mathcal{N}(0,1).
   $$

6. reconstruction loss：

   $$
   L_{\mathrm{recon}}
   =
   \mathbb{E}_{x\in \mathbb{R}^3}
   [\mathrm{BCE}(\hat{O}(x), O(x))].
   $$

7. latent set denoising objective，按论文 EDM 风格表述：

   $$
   \mathbb{E}_{n_i\sim \mathcal{N}(0,\sigma^2 I)}
   \frac{1}{M}
   \sum_{i=1}^{M}
   \left\|
   \mathrm{Denoiser}(\{z_i+n_i\}_{i=1}^{M},\sigma,C)_i-z_i
   \right\|_2^2.
   $$

## 实验设置

1. 数据集：ShapeNet，重点报告 ShapeNet-55 和常见类别条件生成；形状输入可以来自 triangle mesh 或 point cloud。
2. autoencoder 输入：2048 点 point cloud；每次迭代采样 1024 个 bounding volume query points 和 1024 个 near-surface query points。
3. autoencoder 训练：8 张 A100，batch size 512，1600 epochs；learning rate 前 80 epochs 线性升到 $5\times 10^{-5}$，随后 cosine decay 到 $1\times 10^{-6}$。
4. diffusion 训练：4 张 A100，batch size 256，8000 epochs。
5. 默认表示：latent 数 $M=512$，channel $C=512$，推荐压缩 channel $C_0=32$。
6. 评价指标：autoencoding 用 IoU、Chamfer、F-Score；生成用 Surface-FPD/KPD、Rendering-FID/KID、Precision/Recall、MMD-CD/EMD、COV-CD/EMD。

## 主要结果

1. autoencoding ablation，Table 4：$M=512$ 时 IoU=0.965、Chamfer=0.038、F-Score=0.970；随着 $M$ 降到 64，IoU 降到 0.916、Chamfer 升到 0.049。
2. KL compression ablation，Table 5：$C_0=32$ 时 IoU=0.963、Chamfer=0.038、F-Score=0.969；$C_0=8,16,32,64$ 的重建性能接近，说明较强压缩对重建影响不大。
3. unconditional generation，Table 6：$C_0=32$ 的 Ours 达到 Surface-FPD=0.76、Surface-KPD=0.66、Rendering-FID=17.08、Rendering-KID=6.75，优于 Grid-83 和 3DILG。
4. 与 PVD 比较，Table 7：Ours 的 Surface-FPD=0.63、Surface-KPD=0.53、Rendering-FID=17.08、Rendering-KID=6.75；PVD 分别为 2.33、2.65、270.64、281.54，差距很大。
5. category-conditioned generation，Table 9：chair 上 Ours 的 recall=0.86，高于 NeuralWavelet 的 0.57；table 上 Ours recall=0.89，高于 NeuralWavelet 的 0.68，作者据此强调覆盖训练分布的能力更强。
6. qualitative applications：展示 text-conditioned generation、partial point cloud completion、single-view image-conditioned generation 和 novelty retrieval，说明模型不是简单记忆训练集形状。

## 论文贡献

1. 提出固定长度 latent set 的 3D neural field representation，避免 global latent 细节不足和 coordinate grid 太大的问题。
2. 设计面向点云/表面输入的 cross-attention shape encoder，以及结合 KL bottleneck/self-attention/cross-attention 的 decoder。
3. 在 3D shape autoencoding 上提高局部细节重建质量。
4. 在 compressed latent set 上训练 diffusion model，并在多种条件生成任务中展示可扩展性。
5. 把 transformer-friendly representation 引入 3D shape diffusion，提供 mesh/point-cloud 输入到 neural field 输出的统一路线。

## 局限性

作者明确指出：方法需要两阶段训练。第一阶段 autoencoder 比手工 wavelet 等特征更耗时；如果 shape data 分布改变，第一阶段可能需要重训；第二阶段 diffusion 训练本身也很长，训练加速仍是未来方向。

个人分析：3DShape2VecSet 适合 shape-level neural field generation，不是 scene-level generation；它的重建输出仍依赖 occupancy sampling 和 Marching Cubes，处理极大规模场景或材质/纹理不是本文重点。

## 与其他论文的关系

- 前置工作：Occupancy Networks、Convolutional Occupancy Networks、3DILG、AutoSDF、NeuralWavelet、PVD、EDM、Latent Diffusion。
- 后续工作：mesh/shape generation 的 transformer latent 表示、text-to-3D shape diffusion、partial point cloud completion。
- 相似方法：AutoSDF 和 NeuralWavelet 也是 latent-space 3D generation；PVD 直接生成 point cloud。
- 主要区别：本文用 learnable latent set + attention 来表示 neural field，而不是固定 grid、wavelet coefficient 或直接 point cloud。

## 与当前研究方向的关系

在本库 Mesh Generation 方向中，3DShape2VecSet 是“3D shape latent representation + diffusion”的核心节点。它比纯 mesh tokenization 更接近 neural field/occupancy route，也为后续把 diffusion model 用到 shape-level 3D content generation 提供表示层参考。

## 待确认问题

- [ ] 本次完整阅读的是 arXiv v3 PDF；正式发表 venue/DOI 未在 PDF 中确认。
- [ ] PDF 中给出项目/代码 URL，但本次没有额外访问项目页或逐行审查代码。

## 阅读记录

### 摘要总结

论文提出 3DShape2VecSet，一种面向 neural fields 和 generative diffusion models 的 3D shape representation。它把 3D shapes 编码为 latent vectors set，并用 attention 结构进行编码、解码和 diffusion denoising，支持无条件生成、类别条件生成、文本条件生成、点云补全和单图条件生成。

### 粗读记录

粗读确认论文重点是 representation，而不是某个单一条件生成任务。作者先证明 latent set autoencoder 能保留 shape details，再证明压缩 latent set 可以支持 diffusion training，最后把同一个框架接到 category/text/image/partial point cloud 条件上。

### 完整总结

完整阅读 arXiv v3 PDF 后，3DShape2VecSet 的价值在于把 3D neural field 表示做成适合 transformer 和 diffusion 的固定长度 latent set。它用 cross-attention 替代显式坐标 anchor 的 kernel interpolation，用 self-attention 处理 latent set 内部关系，再用 VAE bottleneck 压缩到可训练 diffusion 的空间。实验上，$M=512,C_0=32$ 是重要配置；在 unconditional generation 上，相比 Grid-83/3DILG/PVD，Surface-FPD、Surface-KPD 和 Rendering-FID/KID 都显著更好。

## 用户原始备注

> 暂无。

## 引用信息

```bibtex
@article{zhang2023shape2vecset,
  title={3DShape2VecSet: A 3D Shape Representation for Neural Fields and Generative Diffusion Models},
  author={Zhang, Biao and Tang, Jiapeng and Niessner, Matthias and Wonka, Peter},
  journal={arXiv preprint arXiv:2301.11445},
  year={2023}
}
```
