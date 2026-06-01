---
title: "BlockFusion: Expandable 3D Scene Generation using Latent Tri-plane Extrapolation"
short_name: "BlockFusion"
authors:
  - "Zhennan Wu"
  - "Yang Li"
  - "Han Yan"
  - "Taizhang Shang"
  - "Weixuan Sun"
  - "Senbo Wang"
  - "Ruikai Cui"
  - "Weizhe Liu"
  - "Hiroyuki Sato"
  - "Hongdong Li"
  - "Pan Ji"
year: 2024
venue: "ACM Transactions on Graphics (SIGGRAPH 2024)"
url: "https://arxiv.org/abs/2401.17053"
doi: ""
arxiv: "2401.17053"
code_url: "https://github.com/Tencent/BlockFusion"
topics: 
  - "3d-scene-generation"
  - "neural-3d-based-generation"
paper_type: "method"
task: "large-scale-3d-scene-generation"
tags:
  - literature
status: fully-summarized
summary_level: full
source_level: fulltext-and-code
priority: medium
read_date: "2026-06-01"
review_date:
related_projects:
  - "https://yang-l1.github.io/blockfusion"
pdf_path: ""
retrieved_from:
  - "https://arxiv.org/abs/2401.17053"
  - "https://arxiv.org/html/2401.17053"
  - "https://yang-l1.github.io/blockfusion"
  - "https://github.com/Tencent/BlockFusion"
retrieval_date: "2026-06-01"
read_version: "arXiv v4 HTML, updated 2024-05-24; official project page and code page"
---


# BlockFusion: Expandable 3D Scene Generation using Latent Tri-plane Extrapolation

## 基本信息

- 简称：BlockFusion
- 作者：Zhennan Wu; Yang Li; Han Yan; Taizhang Shang; Weixuan Sun; Senbo Wang; Ruikai Cui; Weizhe Liu; Hiroyuki Sato; Hongdong Li; Pan Ji
- 发表时间：2024
- 会议或期刊：ACM Transactions on Graphics (SIGGRAPH 2024)
- 原文链接：https://arxiv.org/abs/2401.17053
- arXiv：2401.17053
- DOI：
- 代码仓库：https://github.com/Tencent/BlockFusion
- 项目页：https://yang-l1.github.io/blockfusion
- 本地 PDF 路径：
- 所属方向：3d-scene-generation, neural-3d-based-generation
- 当前状态：fully-summarized
- 实际阅读版本：arXiv v4 HTML, updated 2024-05-24; official project page and code page

## 一句话概括

BlockFusion 把 3D scene 切成可扩展的 blocks，用 latent tri-plane diffusion 生成单个块，再通过 overlapping latent tri-plane extrapolation 拼接新块，从而生成可无限扩展的 indoor/outdoor 3D scenes。

## 研究问题

现有 3D generation 多生成固定空间范围内的单物体或小场景；open-world 游戏、AR/VR 和虚拟制作需要可持续扩展的大场景。直接生成大尺度 3D scene 有两个核心难点：场景几何复杂且分布多样，直接训练 3D diffusion 容易不稳定；扩展已有场景时，新旧区域需要在语义和几何上自然过渡。BlockFusion 试图用 block-wise generation 和 overlap-conditioned extrapolation 解决“高质量 3D scene generation”与“无限扩展”之间的矛盾。

## 方法概述

1. block representation：从完整 scene mesh 随机裁剪固定大小 3D blocks，训练模型先学会单块生成。
2. raw tri-plane fitting：每个 block 拟合为 hybrid neural field，由 tri-plane geometry features 和 MLP decoder 输出 SDF。
3. latent tri-plane autoencoder：直接对 raw tri-plane 训练 diffusion 会 collapse；作者用 VAE 把 raw tri-plane 压缩到 latent tri-plane space，再训练 DDPM。
4. latent tri-plane diffusion：在 latent tri-plane 上进行 denoising diffusion，可加入 2D layout conditioning 控制物体布置。
5. latent tri-plane extrapolation：扩展场景时，新 block 与旧 block 有重叠区域；在 reverse diffusion 中用旧 block overlap features 同步新 block，对非重叠区域生成新内容。
6. resampling：借鉴 RePaint，在若干 denoising steps 中回滚加噪并重复采样，提升 overlap 区域的一致性。
7. surface refinement：latent extrapolation 仍可能留下微小 seam，使用 non-rigid registration 对 overlap mesh 做后处理。
8. large scene generation：并行生成不重叠 seed blocks，再并行外推其余 blocks，避免完全串行扩展。

## 关键公式

1. 扩散前向过程：

   $$
   q(z_t\mid z_{t-1})
   =
   \mathcal{N}(\sqrt{1-\beta_t}z_{t-1},\beta_t I).
   $$

2. reverse diffusion：

   $$
   p_{\psi}(z_{t-1}\mid z_t)
   =
   \mathcal{N}(\mu_{\psi}(z_t,t),\Sigma_{\psi}(z_t,t)).
   $$

3. latent tri-plane extrapolation 中旧 block overlap 的加噪：

   $$
   z^{P}_{t-1}(i)
   \sim
   \mathcal{N}
   \left(
   \sqrt{\bar{\alpha}_t}z^P_0(i),
   (1-\bar{\alpha}_t)I
   \right).
   $$

4. 新 block denoising：

   $$
   z^{Q}_{t-1}(i)
   \sim
   \mathcal{N}
   \left(
   \mu_{\psi}(z^Q_t(i),t),
   \Sigma_{\psi}(z^Q_t(i),t)
   \right).
   $$

5. overlap synchronization：

   $$
   z^Q_{t-1}(i)
   \leftarrow
   \mathrm{Cat}
   \left(
   z^P_{t-1}(i)\in O_i,\;
   z^Q_{t-1}(i)\notin O_i
   \right).
   $$

6. non-rigid registration cost：

   $$
   \mathcal{L}_{\mathrm{nrr}}
   =
   \mathcal{L}_{\mathrm{CD}}(\mathcal{W}(\Omega^{{ol}}_P),\Omega^{{ol}}_Q)
   +
   \mathcal{L}_{\mathrm{CD}}(\mathcal{W}(\Omega^{{new}}_Q),\Omega^{{new}}_Q).
   $$

## 实验设置

1. 场景类型：room、city、village。
2. 数据：room 来自 3D-FRONT 和 3D-FUTURE，含 18,968 个 indoor scenes；作者得到 57K random crops，过滤后保留 9,123 rooms。city 和 village 由艺术家设计，各得到 10K blocks。
3. block size：room 为 3.2 米立方，city 为 12 米立方，village 为 15 米立方。
4. 训练成本：3D-FRONT 上 raw tri-plane fitting、autoencoder training、diffusion training 分别约 4,750、768、384 GPU hours；VAE 和 diffusion 用 8 张 GPU。
5. baseline：单块生成与 NFD 比较；室内场景生成与 Text2Room 比较。
6. 指标：unconditional block generation 用 MMD、COV、1-NNA；室内 scene generation 用 48 人 user study 的 Perceptual Quality 和 Structure Completeness，分 textured / geometry-only 两种模式。

## 主要结果

1. Table 3：latent tri-plane diffusion 显著优于 NFD。以 CD 为距离时，NFD 的 COV 为 22.66，latent tri-plane diffusion 的 COV 为 51.83；以 EMD 为距离时，NFD 为 29.66，latent tri-plane diffusion 为 53.33。
2. 单块生成：相对 NFD，作者报告 COV 在 CD / EMD 下分别提升 29.17% 和 23.67%。
3. 室内场景 user study：geometry-only 下 Text2Room 的 G-PQ/G-SC 为 1.92/1.92，BlockFusion 为 4.44/4.58；textured 下 Text2Room 为 2.27/2.29，BlockFusion + Meshy 为 4.08/4.17。
4. Table 4：latent tri-plane 在 99.60% compression rate 下仍有较好 reconstruction（CD 4.097），显著优于相近压缩率的 latent vector（CD 5.431）。
5. Ablation：latent tri-plane 比 raw tri-plane 更适合作为 diffusion target；layout conditioning 可控制大致物体摆放；resampling 能显著降低 overlap region Chamfer Distance；non-rigid registration 能消除 extrapolation seam。
6. 大场景展示：论文展示了 room、city、village 的 block-wise generation，声称可扩展到任意尺度。

## 论文贡献

1. 提出基于 latent tri-plane diffusion 的 high-quality 3D scene block generation。
2. 提出 overlap-conditioned latent tri-plane extrapolation，使新 block 与已有 block 语义和几何自然衔接。
3. 引入 layout conditioning 控制场景元素布置，并支持 room/city/village 多种场景。
4. 使用 surface non-rigid registration 减少 block seam。
5. 给出并行 seed block + extrapolated block 的 large scene generation 策略。

## 局限性

1. 当前 tri-plane 分辨率限制细节，可能无法生成椅子腿等很细的几何结构。
2. bounding box condition 只能控制物体大致位置，不能控制 orientation；作者建议未来加入 object orientation map。
3. 大场景全局一致 texture generation 仍然困难；论文主要展示小场景 texture，并依赖 Meshy 等 off-the-shelf texture tool。
4. 训练成本高，raw tri-plane fitting 与模型训练需要大量 GPU hours。
5. 本次未获得可读本地 PDF，全文总结基于 arXiv HTML 和官方项目页。

## 与其他论文的关系

- 前置工作：NFD、Rodin、EG3D tri-plane representation、latent diffusion、Text2Room、RePaint、3D-FRONT/3D-FUTURE。
- 后续工作：large-scale 3D world generation、layout-conditioned scene diffusion、open-world procedural replacement。
- 相似方法：Text2Room 用 2D diffusion + depth lifting 扩展 room-scale scene；BlockFusion 直接在 3D latent tri-plane blocks 上生成和外推。
- 主要区别：BlockFusion 的“可扩展性”来自 block-wise latent extrapolation，而不是移动相机生成图像再融合。

## 与当前研究方向的关系

在 Neural 3D-based Scene Generation 中，BlockFusion 是从 fixed-size room layout generation 走向 expandable world generation 的代表。它与 ATISS 的关系是：ATISS 关注室内对象布局，BlockFusion 直接生成可拼接的 3D geometry blocks；与 Text2Room/3D Cinemagraphy 相比，它更强调直接 3D 表示和大尺度扩展。

## 待确认问题

- [ ] DOI 未在 arXiv 元数据或项目页中确认。
- [ ] 本地 PDF 下载超时，已删除 partial PDF；本次全文总结基于 arXiv HTML 和官方项目页。
- [ ] 代码仓库已确认存在，但未逐行审查实现。

## 阅读记录

### 摘要总结

BlockFusion 用 latent tri-plane diffusion 生成 3D scene blocks，并通过 overlap-conditioned extrapolation 持续扩展场景。它先把 mesh blocks 拟合成 raw tri-planes，再压缩到 latent tri-plane space 训练 diffusion；扩展时同步 overlap features，必要时 resampling 和 non-rigid registration 消除 seam。

### 粗读记录

论文试图替代手写 procedural generation 的一部分功能：不是人工写规则，而是从数据中学 block distribution，并能在 layout condition 下扩展。方法的关键不是单块生成，而是如何把新块和旧块自然拼起来。

### 完整总结

完整阅读 arXiv HTML 后，BlockFusion 的主要工程权衡很清楚：raw tri-plane 表示能力强但难以直接扩散；latent tri-plane 降低维度后可以稳定训练；overlap-conditioned denoising 让新 block 延续旧 block 的语义和几何。实验支持 latent representation、layout control、resampling 和 surface refinement 的作用。它仍受分辨率、训练成本和 texture consistency 限制，但对 large-scale 3D scene generation 是重要路线。

## 用户原始备注

> 暂无。

## 引用信息

```bibtex
@article{wu2024blockfusion,
  title={BlockFusion: Expandable 3D Scene Generation using Latent Tri-plane Extrapolation},
  author={Wu, Zhennan and Li, Yang and Yan, Han and Shang, Taizhang and Sun, Weixuan and Wang, Senbo and Cui, Ruikai and Liu, Weizhe and Sato, Hiroyuki and Li, Hongdong and Ji, Pan},
  journal={ACM Transactions on Graphics},
  year={2024}
}
```
