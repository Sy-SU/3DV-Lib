---
title: "Denoising Diffusion Implicit Models"
short_name: "DDIM"
authors: 
  - "Jiaming Song"
  - "Chenlin Meng"
  - "Stefano Ermon"
year: 2021
venue: "ICLR 2021"
url: "https://arxiv.org/abs/2010.02502"
doi: "10.48550/arXiv.2010.02502"
arxiv: "2010.02502"
code_url: "https://github.com/ermongroup/ddim"
topics: 
  - "diffusion-model"
  - "text-to-image"
paper_type: "method"
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
  - "https://github.com/ermongroup/ddim"
pdf_path: "Literature/PDFs/DDIM.pdf"
retrieved_from: 
  - "https://arxiv.org/abs/2010.02502"
  - "https://arxiv.org/pdf/2010.02502"
  - "https://github.com/ermongroup/ddim"
retrieval_date: "2026-05-31"
read_version: "arXiv v4, ICLR 2021 version, last revised 2022-10-05"
---

# Denoising Diffusion Implicit Models

## 基本信息

- 简称：DDIM
- 作者：Jiaming Song; Chenlin Meng; Stefano Ermon
- 发表时间：2021
- 会议或期刊：ICLR 2021
- 原文链接：https://arxiv.org/abs/2010.02502
- arXiv 链接：https://arxiv.org/abs/2010.02502
- DOI：10.48550/arXiv.2010.02502
- 代码仓库：https://github.com/ermongroup/ddim
- 本地 PDF 路径：Literature/PDFs/DDIM.pdf
- 所属方向：diffusion-model, text-to-image
- 当前状态：fully-summarized
- 实际阅读版本：arXiv v4, ICLR 2021 version, last revised 2022-10-05

## 一句话概括

DDIM 解决 DDPM 采样慢的问题：它构造一族与 DDPM 共享训练目标的非 Markovian forward process，并在采样时选择更短的时间子序列。特殊的确定性采样情形 $\eta=0$ 形成 implicit model，可在不重新训练网络的情况下用 10 到 50 倍更少的 wall-clock 时间生成高质量样本，并提供更稳定的 latent interpolation 与 reconstruction。

## 研究问题

1. DDPM 能生成高质量图像，但通常需要 1000 个反向步骤，采样远慢于 GAN。
2. 直接减少 DDPM 反向链步数会显著损伤样本质量，特别是在短 trajectory 下。
3. 作者要解决的是：是否能在不改训练目标、不重训模型的前提下，设计一个更快的 generative process，并保留可接受的样本质量。

## 方法概述

1. 输入与输出：输入是已训练好的 DDPM 噪声预测模型 $\epsilon_\theta$，输出是通过 DDIM / generalized process 采样出的图像。
2. 理论核心：把 DDPM 的 Markovian forward process 推广为一族 non-Markovian process，同时保持 $q(x_t \mid x_0)$ marginal 不变，因此可共享 DDPM 的 denoising 训练目标。
3. 采样控制：通过选择 $\sigma_t$ 控制随机性；$\sigma_t=0$ 时没有额外随机噪声，得到确定性 DDIM；$\eta=1$ 对应一种 DDPM-like 随机采样。
4. 加速方式：只在时间子序列 $\tau$ 上采样，令 $S= \mid \tau \mid \ll T$，从而把 1000 步缩短到 10/20/50/100 步。
5. ODE 关系：DDIM 的 deterministic iterate 可以解释为某个 ODE 的 Euler 积分，因此同一初始 latent 通过不同步数采样时会保留相似高级语义。

## 关键公式

1. DDIM/generalized reverse update（论文 Eq. 12）：

   $$
   x_{t-1}=\sqrt{\alpha_{t-1}}\left(\frac{x_t-\sqrt{1-\alpha_t}\epsilon^{(t)}_\theta(x_t)}{\sqrt{\alpha_t}}\right)+\sqrt{1-\alpha_{t-1}-\sigma_t^2}\epsilon^{(t)}_\theta(x_t)+\sigma_t\epsilon_t.
   $$

   第一项是 predicted $x_0$，第二项是指向 $x_t$ 的方向，第三项是随机噪声。控制 $\sigma_t$ 就是在控制采样随机性。

2. DDIM 特例：$\sigma_t=0$。此时反向过程给定 $x_T$ 后确定，模型成为 implicit probabilistic model，因此同一 latent 生成结果更一致。

3. ODE 形式（论文 Eq. 14）：

   $$
   d\bar{x}(t)=\epsilon^{(t)}_\theta(\bar{x}(t)\sqrt{\sigma^2+1})d\sigma(t).
   $$

   该式说明 deterministic DDIM 可以看作沿噪声尺度变量的 ODE 积分，解释了其 latent encoding 和 reconstruction 能力。

## 实验设置

1. 数据集：CIFAR10 32x32、CelebA 64x64、LSUN Bedroom 256x256、LSUN Church 256x256。
2. 模型：CIFAR10、LSUN Bedroom、LSUN Church 使用原 DDPM pretrained checkpoints；CelebA 模型由作者训练。
3. 指标：主要使用 FID；还评估 latent interpolation、reconstruction error 和 sample consistency。
4. 变量：采样步数 $S= \mid \tau \mid$、随机性参数 $\eta$、子序列选择方式（CIFAR10 用 quadratic，其余用 linear）。
5. 代码：官方 GitHub README 说明可用 --timesteps 控制采样步数，用 --eta 控制随机性，eta=0 为 DDIM，eta=1 为 DDPM-like。

## 主要结果

1. CIFAR10/CelebA，Table 1：CIFAR10 上 DDIM $\eta=0$ 在 20 步 FID=6.84、50 步 FID=4.67、100 步 FID=4.16；对应 DDPM-like $\eta=1$ 为 18.36、8.01、5.78。该表说明短 trajectory 下确定性 DDIM 明显优于随机 DDPM-like 采样。
2. CelebA，Table 1：20 步 DDIM FID=13.73，100 步 DDIM FID=6.53；100 步 DDPM-like FID=13.93。作者据此指出 CelebA 上 20 步 DDIM 质量接近 100 步 DDPM。
3. LSUN Bedroom/Church，Appendix Table 3：Bedroom 20 步 DDIM FID=8.89，而 20 步 DDPM FID=22.77；Church 20 步 DDIM FID=12.47，而 20 步 DDPM FID=23.37。
4. 论文正文 Fig. 4 讨论：采样时间近似随 trajectory 长度线性缩放；DDIM 可在 20 到 100 步达到接近 1000 步模型质量，即大约 10x 到 50x 加速。

## 论文贡献

1. 将 DDPM 扩展到共享训练目标的 non-Markovian diffusion process。
2. 提出确定性 DDIM 采样，允许不重训模型就显著减少采样步数。
3. 给出 DDIM 与 ODE / probability flow 的联系。
4. 展示 DDIM 在短采样轨迹下的更好 FID、latent consistency、image interpolation 和 reconstruction 能力。
5. 提供官方代码仓库和采样参数接口。

## 局限性

作者明确指出或实验中可见的局限性：
- DDIM 在步数足够多时不一定优于完整 DDPM；例如 CIFAR10 1000 步下 DDIM FID=4.04，而表中 DDPM noisy setting 可到 FID=3.17。
- 加速采样存在质量-计算量 trade-off，步数过少仍会损伤质量。
- 文中提到 ODE 数值误差可能影响 fewer sampling steps；多步 ODE solver 等降低离散误差的方法留作未来工作。

个人分析：
- DDIM 解决的是采样效率与可逆/一致性问题，不直接提升训练目标或模型表达能力；后续改进仍依赖更好的模型、噪声 schedule 和 sampler。

## 与其他论文的关系

- 前置工作：DDPM；NCSN / score-based generative modeling；probability flow ODE。
- 后续工作：Stable Diffusion 等系统广泛使用 DDIM/DDIM-like sampler；CFG 也可与 DDIM sampler 组合。
- 相似方法：DDPM 采样、SDE/ODE sampler。
- 主要区别：DDIM 不改变训练目标，而是在采样过程和 forward process 解释上做推广。

## 与当前研究方向的关系

DDIM 是扩散模型采样效率和确定性采样的关键论文。它对当前文献库中 Null-text Inversion、Stable Diffusion 相关编辑方法、以及 DreamFusion 这类需要大量迭代优化的系统都有基础作用。

## 待确认问题

- [ ] 后续需要结合 SDE/ODE 论文比较 DDIM ODE 与 probability flow ODE 的精确关系。
- [ ] DDIM 在不同现代 latent diffusion 模型上的最佳步数、$\eta$ 与质量/多样性关系需要单独整理。

## 阅读记录

### 摘要总结

DDIM 提出一族与 DDPM 共享训练目标的 non-Markovian process，并用确定性采样显著减少反向步骤。论文显示在 CIFAR10、CelebA、LSUN 上，短步数 DDIM 通常比短步数 DDPM-like 采样质量更高，并支持 latent interpolation 与 reconstruction。

### 粗读记录

核心记忆点：DDIM 不是重新训练的模型，而是复用 DDPM 噪声预测网络，改变采样轨迹和随机性。把 $\eta=0$ 理解为 deterministic sampler，对后续图像编辑和反演任务很重要。

### 完整总结

完整阅读 arXiv v4 / ICLR 2021 版本。论文先证明 non-Markovian forward process 仍可导向与 DDPM 相同的 surrogate objective，再提出用 $\sigma_t$ 控制采样随机性、用子序列 $\tau$ 控制步数。实验结果集中说明：当从 1000 步缩到 10/20/50/100 步时，DDIM 比 DDPM-like 采样更稳；同时 deterministic process 让同一 latent 对应相似高级内容，因此可做语义插值和重建。官方代码 README 确认了 --timesteps 和 --eta 作为核心采样控制接口。

## 用户原始备注

> 暂无。

## 引用信息

```bibtex
@article{song2020denoising,
  title={Denoising Diffusion Implicit Models},
  author={Song, Jiaming and Meng, Chenlin and Ermon, Stefano},
  journal={arXiv:2010.02502},
  year={2020},
  month={October},
  url={https://arxiv.org/abs/2010.02502}
}
```
