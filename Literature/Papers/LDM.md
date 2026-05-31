---
title: "High-Resolution Image Synthesis with Latent Diffusion Models"
short_name: "LDM"
authors:
  - "Robin Rombach"
  - "Andreas Blattmann"
  - "Dominik Lorenz"
  - "Patrick Esser"
  - "Björn Ommer"
year: 2022
venue: "CVPR 2022"
url: "https://arxiv.org/abs/2112.10752"
doi: "10.48550/arXiv.2112.10752"
arxiv: "2112.10752"
code_url: "https://github.com/CompVis/latent-diffusion"
topics:
  - "diffusion-model"
  - "text-to-image"
paper_type: "foundation"
task: "text-to-image"
tags:
  - literature
status: fully-summarized
summary_level: full
source_level: fulltext-and-code
priority: medium
read_date: "2026-05-31"
review_date: "2026-05-31"
related_projects:
  - "https://github.com/CompVis/latent-diffusion"
  - "https://ommer-lab.com/research/latent-diffusion-models/"
pdf_path: "Literature/PDFs/LDM.pdf"
retrieved_from:
  - "https://arxiv.org/abs/2112.10752"
  - "https://arxiv.org/pdf/2112.10752"
  - "https://github.com/CompVis/latent-diffusion"
  - "https://ommer-lab.com/research/latent-diffusion-models/"
retrieval_date: "2026-05-31"
read_version: "arXiv 2112.10752, CVPR 2022 version, PDF retrieved 2026-05-31"
---

# High-Resolution Image Synthesis with Latent Diffusion Models

## 基本信息

- 简称：LDM
- 作者：Robin Rombach; Andreas Blattmann; Dominik Lorenz; Patrick Esser; Björn Ommer
- 发表时间：2022
- 会议或期刊：CVPR 2022
- 原文链接：https://arxiv.org/abs/2112.10752
- arXiv 链接：https://arxiv.org/abs/2112.10752
- DOI：10.48550/arXiv.2112.10752
- 代码仓库：https://github.com/CompVis/latent-diffusion
- 项目页：https://ommer-lab.com/research/latent-diffusion-models/
- 本地 PDF 路径：Literature/PDFs/LDM.pdf
- 所属方向：diffusion-model, text-to-image
- 当前状态：fully-summarized
- 实际阅读版本：arXiv 2112.10752, CVPR 2022 version, PDF retrieved 2026-05-31

## 一句话概括

LDM 将扩散模型从像素空间搬到学习到的感知压缩 latent space 中训练，再用 cross-attention 接入文本、语义图、超分辨率和 inpainting 等条件，从而在保持生成质量的同时显著降低训练和采样成本，是 Stable Diffusion 一类 latent text-to-image 系统的直接基础。

## 研究问题

1. 像素空间 diffusion model 质量高，但对高分辨率图像训练和采样都很昂贵。
2. 过强的压缩会损坏细节，过弱的压缩又不能充分降低 diffusion 的计算量；论文要找到感知质量、压缩率和生成先验学习之间的平衡。
3. 条件生成需要统一地支持文本、类别、语义布局、低分辨率图像、inpainting mask 等多种条件，而不为每个任务重做一套生成框架。

## 方法概述

1. 先训练感知压缩 autoencoder，把图像 $x$ 编码为 latent $z=\mathcal{E}(x)$，再用 decoder $\mathcal{D}(z)$ 还原图像。autoencoder 使用 perceptual loss、patch discriminator 和 KL 或 VQ 正则化，让 latent 保留语义和外观细节。
2. 在 latent space 上训练 diffusion prior。扩散模型不再直接预测像素噪声，而是在 $z_t$ 上学习 denoising score / noise prediction，这让 U-Net 的空间分辨率按压缩因子 $f$ 缩小。
3. 对条件生成，论文把条件编码器 $\tau_\theta(y)$ 的输出通过 cross-attention 注入 U-Net。这样文本 token、semantic layout、类别 embedding 等都能作为条件进入同一个 latent diffusion backbone。
4. 对高分辨率生成，LDM 可以以卷积方式在 latent feature map 上采样，避免自回归像素模型或像素 diffusion 的高成本。
5. 任务覆盖无条件图像合成、class-conditional ImageNet、text-to-image、layout-to-image、super-resolution 和 inpainting。

## 关键公式

1. Autoencoder 编码和解码：

   $$
   z = \mathcal{E}(x), \qquad \tilde{x} = \mathcal{D}(z).
   $$

   这一阶段把像素空间压缩到感知等价但计算更便宜的 latent space。

2. Latent diffusion 训练目标：

   $$
   L_{\mathrm{LDM}}
   =
   \mathbb{E}_{\mathcal{E}(x), \epsilon, t}
   \left[
   \left\|
   \epsilon -
   \epsilon_\theta(z_t,t)
   \right\|_2^2
   \right].
   $$

   与 DDPM 的噪声预测目标类似，只是 $x_t$ 被替换为 latent $z_t$。

3. 条件 LDM：

   $$
   L_{\mathrm{LDM}}
   =
   \mathbb{E}_{\mathcal{E}(x), y, \epsilon, t}
   \left[
   \left\|
   \epsilon -
   \epsilon_\theta(z_t,t,\tau_\theta(y))
   \right\|_2^2
   \right].
   $$

   其中 $\tau_\theta(y)$ 可以是文本、类别、布局或其他模态的编码。

4. Cross-attention 注入条件：

   $$
   \mathrm{Attention}(Q,K,V)
   =
   \mathrm{softmax}
   \left(
   \frac{QK^\top}{\sqrt{d}}
   \right)V,
   $$

   U-Net feature 产生 query，条件 token 产生 key/value，使生成过程在不同空间位置选择性读取条件信息。

## 实验设置

1. Autoencoder 在 OpenImages 等大规模图像上训练，论文比较不同压缩因子 $f$、KL/VQ 正则和 attention 配置，并用 ImageNet validation 的 reconstruction FID / PSNR 等指标评估压缩损失。
2. 无条件生成覆盖 CelebA-HQ、FFHQ、LSUN-Churches、LSUN-Bedrooms、ImageNet 等数据集。
3. 文本条件生成使用 LAION-400M 训练文本图像模型，并在 MS-COCO 256x256 validation 上与 CogView、LAFITE、GLIDE、Make-A-Scene 等比较。
4. Class-conditional ImageNet 使用 250 DDIM steps，并比较 ADM / ADM-G 等 diffusion baseline。
5. 下游任务还包括 ImageNet 4x super-resolution、COCO/OpenImages layout-to-image 和 Places inpainting。

## 主要结果

1. 无条件图像生成中，LDM 在多个数据集接近或超过同时期强基线：CelebA-HQ FID 5.11，FFHQ FID 4.98，LSUN-Churches FID 4.02，LSUN-Bedrooms FID 2.95。
2. MS-COCO text-to-image 上，LDM-KL-8-G 取得 FID 12.63、IS 30.29，在参数量显著少于 GLIDE / Make-A-Scene 的情况下达到接近的质量。
3. Class-conditional ImageNet 上，LDM-4-G 取得 FID 3.60、IS 247.67，优于 ADM-G 报告的 FID 4.59。
4. Autoencoder 压缩因子分析显示，$f=4$ 到 $f=8$ 通常是较好的质量和效率折中；过小压缩省不了太多算力，过大压缩会损坏细粒度重建。
5. Inpainting 和 super-resolution 结果说明 latent diffusion 不只是文生图骨干，也可以作为通用 image-to-image conditional generator。

## 论文贡献

1. 提出在 perceptual latent space 中训练 diffusion model 的系统方案，显著降低高分辨率图像生成成本。
2. 通过 cross-attention 给 latent diffusion 提供通用条件接口，奠定后续 Stable Diffusion 风格文本条件生成的架构基础。
3. 系统分析 autoencoder 压缩率与 diffusion 先验学习之间的权衡，给出 $f=4/8$ 的实用经验。
4. 在无条件、文本条件、类别条件、layout-to-image、super-resolution 和 inpainting 中展示统一框架的广泛适用性。

## 局限性

1. 采样仍是序列式去噪，虽然比像素 diffusion 便宜，但仍慢于 GAN。
2. Autoencoder 是潜在瓶颈：当任务需要像素级精确性时，latent 压缩带来的重建误差会限制上限。
3. 大规模文本图像模型仍继承训练数据中的偏见、版权和误用风险。
4. Cross-attention 能接入多种条件，但条件对齐和可控性仍不保证，复杂组合提示可能失败。

## 与其他论文的关系

- 前置工作：[[DDPM]] 提供 diffusion 训练和采样范式；[[DDIM]] 提供更快的确定性采样；[[CFG]] 强化条件生成质量。
- 后续工作：Stable Diffusion 直接继承 LDM 的 latent autoencoder + U-Net + cross-attention 设计；[[Null-text Inversion for Editing Real Images using Guided Diffusion Models]] 利用 Stable Diffusion 和 Prompt-to-Prompt 做真实图像编辑。
- 相似方法：GLIDE、Imagen、DALL-E 2 等也做文本条件图像生成，但很多工作在像素空间或使用更重的模型。
- 主要区别：LDM 的核心不是新的 diffusion 目标，而是把 diffusion prior 放到感知 latent space 中，并证明这个工程选择能系统改善成本和质量平衡。

## 与当前研究方向的关系

对文生图、文生 3D 和后续 SDS / Score Distillation 系列非常关键。许多 3D 生成论文实际依赖的 2D prior 都来自 Stable Diffusion 或 LDM 类模型，因此理解 latent 压缩、cross-attention、CFG 与采样成本，是理解 DreamFusion、Magic3D、3D editing 等工作的前提。

## 待确认问题

- 无。

## 阅读记录

### 摘要总结

LDM 的中心思想是：不要在像素空间里浪费 diffusion 模型的大部分计算去学习人眼不敏感的细节，而是先用 autoencoder 得到感知压缩 latent，再在 latent 上做扩散生成。论文证明这种做法在质量、速度和显存之间取得好折中，并用 cross-attention 让同一个框架支持文本、布局、类别和图像条件。

### 粗读记录

重点关注了三个层面：第一，autoencoder 压缩率决定生成上限；第二，latent U-Net 的训练目标基本沿用 DDPM 噪声预测；第三，cross-attention 是 LDM 成为通用条件生成器的关键。实验表明 LDM 不只是省算力的 trick，而是在多个基准上达到或接近 SOTA。

### 完整总结

这篇论文把高质量扩散生成的主要计算负担从像素空间移到语义更紧凑的 latent space。作者先训练 autoencoder，使图像经过 $\mathcal{E}$ 和 $\mathcal{D}$ 后仍保留感知质量；再在 latent $z$ 上训练 diffusion 模型。由于 latent feature map 的空间尺寸被压缩，U-Net 的训练和采样成本下降，同时模型仍可通过 decoder 回到高分辨率图像。

论文最重要的工程选择是不要把 latent 压得太狠。若压缩因子过大，autoencoder 的 reconstruction bottleneck 会限制所有后续生成任务；若压缩因子过小，扩散模型仍然昂贵。实验显示 $f=4$ 和 $f=8$ 是常用且有效的区间。第二个关键选择是把条件作为 token 序列通过 cross-attention 注入 U-Net，这比简单拼接更适合文本等非空间条件，也让 LDM 成为后续 Stable Diffusion 的基础架构。

实验覆盖面很宽。无条件图像生成说明 latent diffusion 不牺牲太多质量；MS-COCO text-to-image 和 ImageNet class-conditional 说明它能和更大模型竞争；super-resolution、inpainting、layout-to-image 则说明该框架能作为通用 image-to-image prior。局限主要在两处：采样仍是慢的多步去噪；autoencoder 的重建能力对像素级任务形成硬上限。

## 用户原始备注

> 暂无。

## 引用信息

~~~bibtex
@inproceedings{rombach2022highresolution,
  title = {High-Resolution Image Synthesis with Latent Diffusion Models},
  author = {Rombach, Robin and Blattmann, Andreas and Lorenz, Dominik and Esser, Patrick and Ommer, Björn},
  booktitle = {Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition},
  year = {2022}
}
~~~

