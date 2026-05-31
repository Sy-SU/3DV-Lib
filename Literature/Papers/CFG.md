---
title: "Classifier-Free Diffusion Guidance"
short_name: "CFG"
authors: 
  - "Jonathan Ho"
  - "Tim Salimans"
year: 2022
venue: "arXiv; short version in NeurIPS 2021 Workshop on Deep Generative Models and Downstream Applications"
url: "https://arxiv.org/abs/2207.12598"
doi: "10.48550/arXiv.2207.12598"
arxiv: "2207.12598"
code_url: ""
topics: 
  - "diffusion-model"
  - "text-to-image"
paper_type: "method"
task: "text-to-image"
tags:
  - literature
status: fully-summarized
summary_level: full
source_level: fulltext
priority: high
read_date: "2026-05-31"
review_date: "2026-05-31"
related_projects: []
pdf_path: "Literature/PDFs/CFG.pdf"
retrieved_from: 
  - "https://arxiv.org/abs/2207.12598"
  - "https://arxiv.org/pdf/2207.12598"
retrieval_date: "2026-05-31"
read_version: "arXiv v1, submitted 2022-07-26"
---

# Classifier-Free Diffusion Guidance

## 基本信息

- 简称：CFG
- 作者：Jonathan Ho; Tim Salimans
- 发表时间：2022
- 会议或期刊：arXiv；短版发表于 NeurIPS 2021 Workshop on Deep Generative Models and Downstream Applications
- 原文链接：https://arxiv.org/abs/2207.12598
- arXiv 链接：https://arxiv.org/abs/2207.12598
- DOI：10.48550/arXiv.2207.12598
- 代码仓库：
- 本地 PDF 路径：Literature/PDFs/CFG.pdf
- 所属方向：diffusion-model, text-to-image
- 当前状态：fully-summarized
- 实际阅读版本：arXiv v1, submitted 2022-07-26

## 一句话概括

Classifier-Free Guidance 解决的是条件扩散模型如何在不额外训练噪声分类器的情况下做“低温/截断式”采样控制的问题。它通过在训练时随机丢弃条件，联合得到条件与无条件噪声预测，再在采样时线性组合二者，从而在样本质量和多样性之间调节。论文在 ImageNet 类条件生成上证明，该方法能得到类似 classifier guidance 的 FID/IS 权衡，并成为后续文生图和文生 3D 系统常用的基础组件。

## 研究问题

1. 扩散模型不能像 GAN truncation 或 flow low-temperature sampling 那样直接通过缩放噪声得到稳定的质量-多样性权衡；已有的 classifier guidance 需要额外训练一个可处理噪声图像的分类器。
2. 额外分类器会让训练管线复杂化，且不能简单使用普通预训练分类器，因为分类器需要在带噪 latent/input 上工作。
3. 作者要回答的问题是：是否可以只用生成模型本身，在没有分类器梯度的情况下完成 guidance，并得到与 classifier guidance 类似的 FID/IS trade-off。

## 方法概述

1. 输入与输出：输入是带条件 $c$ 的扩散模型采样过程；输出是在同一条件下经过 guidance 调节的样本。
2. 训练框架：训练单个条件扩散网络，但以概率 $p_{uncond}$ 将条件 $c$ 替换为空条件 $\emptyset$，使同一个网络同时学习 $\epsilon_\theta(z_\lambda,c)$ 和 $\epsilon_\theta(z_\lambda)$。
3. 采样框架：每一步同时计算条件噪声预测和无条件噪声预测，用 guidance strength $w$ 做线性组合，再把组合后的 $\tilde{\epsilon}$ 放入原扩散采样器；论文指出采样器本身可以替换为 DDIM 等其他 sampler。
4. 与 classifier guidance 的差异：classifier guidance 把扩散 score 与外部分类器梯度混合；CFG 用条件/无条件生成 score 的差值构造隐式 guidance，不依赖外部分类器，也不需要训练 noisy classifier。
5. 训练超参数：论文主要研究 $p_{uncond}$ 和采样步数 $T$，并在 ImageNet 64x64/128x128 上扫 guidance strength $w$。

## 关键公式

1. Classifier guidance 的参考形式：

   $$
   \tilde{\epsilon}_\theta(z_\lambda,c)=\epsilon_\theta(z_\lambda,c)-w\sigma_\lambda\nabla_{z_\lambda}\log p_\theta(c \mid z_\lambda).
   $$

   这里 $w$ 控制 guidance 强度，分类器梯度提高分类器认为属于目标类的区域概率；该式说明传统 guidance 依赖外部分类器。

2. CFG 的采样公式（论文 Eq. 6）：

   $$
   \tilde{\epsilon}_\theta(z_\lambda,c)=(1+w)\epsilon_\theta(z_\lambda,c)-w\epsilon_\theta(z_\lambda).
   $$

   $\epsilon_\theta(z_\lambda,c)$ 是条件噪声预测，$\epsilon_\theta(z_\lambda)$ 是空条件/无条件噪声预测。$w=0$ 退化为普通条件采样，增大 $w$ 通常提升单样本保真度但降低多样性。

3. 训练目标核心：随机丢弃条件后仍使用普通扩散 denoising MSE，$\|\epsilon_\theta(z_\lambda,c)-\epsilon\|^2$。这说明 CFG 的实现主要是训练时条件 dropout 与采样时线性组合。

## 实验设置

1. 数据集：area-downsampled class-conditional ImageNet，分辨率 64x64 和 128x128。
2. 指标：FID 和 Inception Score，均用 50,000 个样本计算。
3. 主要变量：guidance strength $w \in \{0,0.1,0.2,\ldots,4\}$，无条件训练概率 $p_{uncond}\in\{0.1,0.2,0.5\}$，以及 128x128 上采样步数 $T\in\{128,256,1024\}$。
4. 训练设置：64x64 模型训练 400k steps；128x128 模型训练 2.7M steps；模型架构和多数超参数沿用 Dhariwal & Nichol 的 guided diffusion 设置。
5. 对比方法：BigGAN-deep、CDM、ADM/ADM-G、LOGAN 等。

## 主要结果

1. ImageNet 64x64，Table 1：在 $p_{uncond}=0.1$ 下，非 guidance $w=0$ 时 FID=1.80、IS=53.71；$w=0.1$ 时 FID=1.55、IS=66.11；$w=4.0$ 时 FID=26.22、IS=260.2。该表显示 guidance 强度增大时 IS 提升但 FID 在强 guidance 下变差，体现质量/多样性权衡。
2. ImageNet 64x64，Table 1：$p_{uncond}=0.5$ 在几乎整个曲线上弱于 $p_{uncond}=0.1/0.2$，作者据此认为只需较小比例模型容量用于无条件任务。
3. ImageNet 128x128，Table 2：$T=256, w=0.3$ 时 FID=2.43、IS=158.47；同表中 ADM-G 的 FID=2.97。这是作者用来说明其 128x128 FID 优于 classifier-guided ADM-G 的关键结果。
4. ImageNet 128x128，Table 2：$T=256, w=4.0$ 时 FID=21.53、IS=421.03；作者将其与 BigGAN-deep max-IS 设置 FID=25、IS=253 对比，说明强 guidance 下 IS 很高但多样性下降明显。

## 论文贡献

1. 提出不依赖外部分类器的 classifier-free guidance。
2. 给出训练时随机丢弃条件、采样时混合条件/无条件 score 的极简实现方式。
3. 从隐式分类器 $p(c \mid z)\propto p(z \mid c)/p(z)$ 角度解释条件与无条件 score 差值为何可作为 guidance 信号。
4. 在 ImageNet 64x64/128x128 上系统展示 guidance strength、无条件训练概率和采样步数对 FID/IS 曲线的影响。

## 局限性

作者明确指出的局限性：
- CFG 采样通常需要对扩散模型做两次前向传播：一次条件、一次无条件；如果分类器比生成模型小，classifier guidance 可能更快。
- 任何提高保真度同时牺牲多样性的 guidance 都可能在部署场景中带来问题，尤其当数据中某些群体或模式本来就欠代表时。
- 当前形式依赖训练出无条件模型；如果条件空间很大，用枚举条件来恢复无条件 score 并不实际。

个人分析：
- 论文主要在类条件 ImageNet 上验证，后续文本条件、跨模态和高分辨率场景中的最佳 $w$ 与副作用需要重新校准。
- FID/IS 本身是分类器相关指标，虽然论文证明不需要分类器梯度，也并不等于完全解决评价指标偏差。

## 与其他论文的关系

- 前置工作：DDPM；classifier guidance / ADM-G；BigGAN truncation；Glow low-temperature sampling。
- 后续工作：Stable Diffusion、Imagen 类文生图系统和 DreamFusion 等文生 3D 方法大量使用 CFG 思想。
- 相似方法：classifier guidance。
- 主要区别：CFG 不训练额外 noisy classifier，而是在同一扩散网络内联合条件和无条件预测，再通过线性组合实现 guidance。

## 与当前研究方向的关系

这篇论文是扩散模型条件控制的基础方法之一。对当前文献库而言，它直接连接 DDPM/DDIM 等采样理论与 DreamFusion 这类使用大 guidance weight 的文生 3D 方法，也解释了文生图系统里常见的“guidance scale”为什么会提高保真度但压缩多样性。

## 待确认问题

- [ ] 文本条件扩散中 $w$ 的最佳范围与 ImageNet 类条件实验的关系需要结合具体模型重新检查。
- [ ] CFG 与后续 negative prompt、null-text inversion 等方法的联系可以在读相关论文时继续补充。

## 阅读记录

### 摘要总结

本文提出 classifier-free guidance：训练时随机把条件替换为空条件，使一个扩散模型同时具备条件和无条件噪声预测；采样时用 $(1+w)\epsilon_\theta(z,c)-w\epsilon_\theta(z)$ 调节样本。论文证明它能在无需外部分类器的情况下实现类似 classifier guidance 的质量-多样性权衡。

### 粗读记录

重点理解两点：第一，CFG 不是新训练目标，而是条件 dropout 加采样时 score 组合；第二，$w$ 增大并不是单向“变好”，而是在保真度、分类器指标和多样性之间移动。

### 完整总结

完整阅读 arXiv v1。本文从 classifier guidance 的复杂性出发，提出用生成模型自身替代 noisy classifier。训练阶段以概率 $p_{uncond}$ 丢弃条件，采样阶段混合条件和无条件噪声预测，形成 $\tilde{\epsilon}$。实验在 ImageNet 64x64 与 128x128 上扫 $w$、$p_{uncond}$ 和步数 $T$，展示小 guidance 可改善 FID，强 guidance 可提高 IS 但牺牲多样性。作者还指出 CFG 的代价是采样可能需要两次 U-Net 前向，且所有 guidance 都要面对多样性下降的问题。

## 用户原始备注

> 第一遍读的时候这部分过的比较快，再读这个系列的时候需要深究一下里面的原理；CFG 似乎也是文生图、文生 3D 中的基础之一。

## 引用信息

```bibtex

```
