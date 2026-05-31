---
title: "Denoising Diffusion Probabilistic Models"
short_name: "DDPM"
authors: 
  - "Jonathan Ho"
  - "Ajay Jain"
  - "Pieter Abbeel"
year: 2020
venue: "NeurIPS 2020"
url: "https://arxiv.org/abs/2006.11239"
doi: "10.48550/arXiv.2006.11239"
arxiv: "2006.11239"
code_url: "https://github.com/hojonathanho/diffusion"
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
  - "https://github.com/hojonathanho/diffusion"
pdf_path: "Literature/PDFs/DDPM.pdf"
retrieved_from: 
  - "https://arxiv.org/abs/2006.11239"
  - "https://arxiv.org/pdf/2006.11239"
  - "https://github.com/hojonathanho/diffusion"
retrieval_date: "2026-05-31"
read_version: "arXiv v2, NeurIPS 2020 version, last revised 2020-12-16"
---

# Denoising Diffusion Probabilistic Models

## 基本信息

- 简称：DDPM
- 作者：Jonathan Ho; Ajay Jain; Pieter Abbeel
- 发表时间：2020
- 会议或期刊：NeurIPS 2020
- 原文链接：https://arxiv.org/abs/2006.11239
- arXiv 链接：https://arxiv.org/abs/2006.11239
- DOI：10.48550/arXiv.2006.11239
- 代码仓库：https://github.com/hojonathanho/diffusion
- 本地 PDF 路径：Literature/PDFs/DDPM.pdf
- 所属方向：diffusion-model, text-to-image
- 当前状态：fully-summarized
- 实际阅读版本：arXiv v2, NeurIPS 2020 version, last revised 2020-12-16

## 一句话概括

DDPM 将扩散概率模型做成可训练、可采样的高质量图像生成模型：前向过程逐步加高斯噪声，反向过程学习去噪。论文用预测噪声的参数化和简化加权 variational bound 建立了与 denoising score matching / Langevin dynamics 的联系，并在 CIFAR10 与 LSUN 上展示强样本质量。

## 研究问题

1. 早期 diffusion probabilistic models 理论明确，但尚未充分证明能生成与 GAN 竞争的高质量图像。
2. 直接优化标准 variational bound 时，各时间步目标权重不一定对应最好的感知样本质量。
3. 作者要解决：如何选择前向噪声 schedule、反向过程参数化和训练目标，使 diffusion model 成为实用的图像生成基线。

## 方法概述

1. 输入与输出：训练输入为数据图像 $x_0$，随机时间步 $t$ 和噪声 $\epsilon$；模型学习从带噪 $x_t$ 预测噪声，采样时从 $x_T\sim \mathcal{N}(0,I)$ 逐步去噪得到 $x_0$。
2. 前向过程：固定 Markov chain，按预设 $\beta_t$ 逐步把数据加噪到近似标准高斯。
3. 反向过程：学习 Gaussian transition 的均值，方差用固定时间相关常数；均值通过噪声预测网络 $\epsilon_\theta(x_t,t)$ 表达。
4. 训练目标：从 variational bound 推导出预测噪声的 MSE，并使用简化目标 $L_{\mathrm{simple}}$，它更强调高噪声、困难去噪任务。
5. 网络结构：U-Net / Wide ResNet 风格 backbone，group normalization，16x16 分辨率 self-attention，时间步用 Transformer sinusoidal embedding 注入残差块。

## 关键公式

1. 前向加噪：

   $$
   q(x_t \mid x_{t-1})
   =
   \mathcal{N}
   \left(
   x_t;
   \sqrt{1-\beta_t}x_{t-1},
   \beta_t I
   \right).
   $$

   这定义了逐步破坏数据的 diffusion process。

2. 任意时间步闭式采样：

   $$
   q(x_t \mid x_0)
   =
   \mathcal{N}
   \left(
   x_t;
   \sqrt{\bar{\alpha}_t}x_0,
   (1-\bar{\alpha}_t) I
   \right).
   $$

   它使训练可随机采样单个 $t$，不用展开整条链。

3. 简化训练目标（论文 Eq. 14 / Algorithm 1）：

   $$
   L_{\mathrm{simple}}
   =
   \mathbb{E}_{t,x_0,\epsilon}
   \left[
   \left\|
   \epsilon
   -
   \epsilon_\theta
   \left(
   \sqrt{\bar{\alpha}_t}x_0
   +
   \sqrt{1-\bar{\alpha}_t}\epsilon,
   t
   \right)
   \right\|^2
   \right].
   $$

   模型直接预测加入的噪声；这是 DDPM 后续大量工作采用的核心训练形式。

4. 采样更新（Algorithm 2）：

   $$
   x_{t-1}
   =
   \frac{1}{\sqrt{\alpha_t}}
   \left(
   x_t
   -
   \frac{1-\alpha_t}{\sqrt{1-\bar{\alpha}_t}}
   \epsilon_\theta(x_t,t)
   \right)
   +
   \sigma_t z.
   $$

   该式用网络噪声预测构造反向均值，再加可控方差噪声。

## 实验设置

1. 数据集：CIFAR10 32x32、CelebA-HQ 256x256、LSUN Church/Bedroom/Cat 256x256。
2. 主要指标：CIFAR10 使用 Inception Score、FID、NLL bits/dim；LSUN 使用 FID。
3. 训练设置：$T=1000$，$\beta_t$ 线性从 $10^{-4}$ 到 0.02；CIFAR10 batch size 128，大图 batch size 64；CIFAR10 dropout 0.1；EMA decay 0.9999。
4. 计算设置：CIFAR 模型 35.7M 参数；LSUN/CelebA-HQ 模型 114M 参数；TPU v3-8；CIFAR10 800k steps，CelebA-HQ 0.5M steps，LSUN Bedroom 2.4M steps，LSUN Church 1.2M steps。
5. 对比方法：BigGAN、StyleGAN2+ADA、NCSN/NCSNv2、SNGAN、PixelCNN/Transformer 等。

## 主要结果

1. CIFAR10，Table 1：使用 $L_{\mathrm{simple}}$ 的 DDPM 得到 IS=9.46±0.11、FID=3.17、test NLL ≤3.75 bits/dim；这是论文最核心的样本质量结果。
2. CIFAR10，Table 2：$\epsilon$-prediction + $L_{\mathrm{simple}}$ 的 FID=3.17，而同为 fixed isotropic variance 但训练 true variational bound 的 $\epsilon$-prediction FID=13.51；说明简化目标显著改善样本质量。
3. LSUN，Figures 3/4 与 Appendix Table 3：LSUN Church FID=7.89，LSUN Bedroom large model FID=4.90；作者据此认为 256x256 LSUN 样本质量接近 ProgressiveGAN。
4. Progressive coding，Section 4.3：CIFAR10 高质量模型的 rate=1.78 bits/dim、distortion=1.97 bits/dim，RMSE=0.95/255；作者用这说明许多 lossless codelength 用在感知上不重要的细节。

## 论文贡献

1. 将扩散概率模型整理成高质量图像生成框架，证明其样本质量可与 GAN 系列竞争。
2. 推导并实证验证噪声预测参数化 $\epsilon_\theta$ 与简化 denoising 目标。
3. 建立 diffusion training 与 denoising score matching、多噪声 Langevin dynamics 之间的联系。
4. 展示 diffusion sampling 可解释为 progressive lossy decompression。
5. 发布官方 TensorFlow 实现。

## 局限性

作者明确指出的局限性：
- 与其他 likelihood-based 模型相比，DDPM 的 log likelihood / lossless codelength 不具竞争力。
- 采样需要 1000 次网络评估，速度慢；官方实验中 CIFAR batch sampling 和 256x256 sampling 都显著重。
- 论文保留了改用更强 decoder 等方向作为未来工作。

个人分析：
- DDPM 的基础版本没有条件控制、没有快速采样，也没有后续 learned variance / classifier guidance / CFG 的增强；很多实际系统依赖后续论文补齐这些问题。

## 与其他论文的关系

- 前置工作：Sohl-Dickstein et al. 的 nonequilibrium thermodynamics diffusion；NCSN / score matching；VAE variational bound。
- 后续工作：DDIM、Improved DDPM、classifier guidance、CFG、LDM、DreamFusion 等。
- 相似方法：score-based generative models、annealed Langevin dynamics。
- 主要区别：DDPM 用固定前向加噪链和学习反向去噪链，以噪声预测 MSE 训练，使 diffusion model 成为强图像生成基线。

## 与当前研究方向的关系

这是当前文献库扩散模型方向的核心基础论文。后续 DDIM 的采样加速、CFG 的条件控制、LDM 的 latent-space diffusion、DreamFusion 的 SDS 都直接建立在 DDPM 的噪声预测和反向去噪思想上。

## 待确认问题

- [ ] 需要继续读 Improved DDPM 来补充 learned variance、cosine schedule 和 likelihood/sample-quality trade-off。
- [ ] DDPM 与 SDE 统一框架的关系需要在 Score-Based Generative Modeling through SDEs 中进一步梳理。

## 阅读记录

### 摘要总结

DDPM 提出用固定前向高斯加噪和学习反向去噪链进行图像生成。其关键实现是预测噪声 $\epsilon$ 并用简化 MSE 目标训练，论文在 CIFAR10 上报告 IS=9.46、FID=3.17，并展示 256x256 LSUN 质量。

### 粗读记录

要把 DDPM 记成三件事：前向闭式加噪、反向预测噪声、1000 步采样。它的最大影响是让 diffusion model 从理论模型变成强生成基线，但采样慢和后续控制问题仍留给 DDIM/CFG 等论文。

### 完整总结

完整阅读 arXiv v2 / NeurIPS 2020 版本。论文从 variational diffusion chain 出发，把反向均值改写为噪声预测问题，最终采用 $L_{\mathrm{simple}}$ 训练目标。实验显示，虽然 true variational bound 给出更好 codelength，但 $L_{\mathrm{simple}}$ 给出更好的感知样本质量；CIFAR10 FID=3.17 是关键结果。论文还把采样过程解释为 progressive decoding / lossy compression，指出大量比特用于不可感知细节。官方 GitHub README 确认了论文、项目网页、训练/评估脚本和 citation。

## 用户原始备注

> 暂无。

## 引用信息

```bibtex
@article{ho2020denoising,
  title={Denoising Diffusion Probabilistic Models},
  author={Jonathan Ho and Ajay Jain and Pieter Abbeel},
  year={2020},
  journal={arXiv preprint arxiv:2006.11239}
}
```
