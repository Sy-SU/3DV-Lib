---
title: "Human Motion Diffusion Model"
short_name: "MDM"
authors:
  - "Guy Tevet"
  - "Sigal Raab"
  - "Brian Gordon"
  - "Yonatan Shafir"
  - "Daniel Cohen-Or"
  - "Amit H. Bermano"
year: 2023
venue: "ICLR 2023"
url: "https://arxiv.org/abs/2209.14916"
doi: "10.48550/arXiv.2209.14916"
arxiv: "2209.14916"
code_url: "https://github.com/GuyTevet/motion-diffusion-model"
topics:
  - "text-to-human-motion"
paper_type: "method"
task: "motion-generation"
tags:
  - literature
status: fully-summarized
summary_level: full
source_level: fulltext-and-code
priority: medium
read_date: "2026-05-31"
review_date: "2026-05-31"
related_projects:
  - "https://github.com/GuyTevet/motion-diffusion-model"
pdf_path: "Literature/PDFs/MDM.pdf"
retrieved_from:
  - "https://arxiv.org/abs/2209.14916"
  - "https://arxiv.org/pdf/2209.14916"
  - "https://github.com/GuyTevet/motion-diffusion-model"
retrieval_date: "2026-05-31"
read_version: "arXiv 2209.14916, ICLR 2023 version, PDF retrieved 2026-05-31"
---

# Human Motion Diffusion Model

## 基本信息

- 简称：MDM
- 作者：Guy Tevet; Sigal Raab; Brian Gordon; Yonatan Shafir; Daniel Cohen-Or; Amit H. Bermano
- 发表时间：2023
- 会议或期刊：ICLR 2023
- 原文链接：https://arxiv.org/abs/2209.14916
- arXiv 链接：https://arxiv.org/abs/2209.14916
- DOI：10.48550/arXiv.2209.14916
- 代码仓库：https://github.com/GuyTevet/motion-diffusion-model
- 本地 PDF 路径：Literature/PDFs/MDM.pdf
- 所属方向：text-to-human-motion
- 当前状态：fully-summarized
- 实际阅读版本：arXiv 2209.14916, ICLR 2023 version, PDF retrieved 2026-05-31

## 一句话概括

MDM 是面向人体动作生成的 classifier-free diffusion 模型：它用 Transformer encoder 在动作序列上直接预测 clean motion sample $x_0$，从而能加入位置、速度和 foot contact 等几何损失，在 text-to-motion、action-to-motion 和 motion in-betweening 上形成强基线。

## 研究问题

1. 文本到人体动作是多解问题，同一句描述可以对应多种合理动作，传统 one-to-one 回归或 VAE 方法容易过平滑。
2. 人体动作不仅要匹配语义，还要满足物理和运动学约束，例如脚接触地面时不应明显滑动。
3. 现有 diffusion 在图像中有效，但人体动作维度更低、时序结构更强，需要适合 motion representation 的网络和损失设计。

## 方法概述

1. MDM 把人体动作表示为长度为 $N$ 的序列 $x \in \mathbb{R}^{N \times J \times D}$，可包含关节位置、旋转、速度和 foot contact 标签。
2. 条件输入可以是文本、动作类别或空条件。文本条件使用冻结 CLIP ViT-B/32 文本编码器得到 embedding，并用 classifier-free 方式随机丢弃条件以支持 guidance。
3. Backbone 是 Transformer encoder。扩散时间步、条件 token 和 noisy motion token 共同进入模型。
4. 关键设计是预测 $x_0$ 而不是预测噪声 $\epsilon$。这使模型输出可直接用于几何空间约束，例如 joint position loss、velocity loss 和 foot contact loss。
5. 同一模型可以支持 text-to-motion、action-to-motion、unconstrained generation 和 motion in-betweening。

## 关键公式

1. 反向生成分布：

   $$
   p_\theta(x_{0:T} \mid c)
   =
   p(x_T)
   \prod_{t=1}^{T}
   p_\theta(x_{t-1} \mid x_t, c).
   $$

   条件 $c$ 可以是文本 embedding、动作类别或空条件。

2. MDM 的 sample prediction 简化目标：

   $$
   L_{\mathrm{simple}}
   =
   \mathbb{E}_{x_0,t,\epsilon}
   \left[
   \left\|
   x_0 -
   G_\theta(x_t,t,c)
   \right\|_2^2
   \right].
   $$

   与 DDPM 的噪声预测不同，模型直接输出 clean motion。

3. Foot contact loss：

   $$
   L_{\mathrm{foot}}
   =
   \frac{1}{N}
   \sum_i
   \left\|
   (p_i - p_{i-1}) \cdot f_i
   \right\|_2^2.
   $$

   其中 $f_i$ 是脚部接触 mask，用来抑制接触地面时的脚滑。

4. 总训练目标：

   $$
   L
   =
   L_{\mathrm{simple}}
   +
   \lambda_{\mathrm{pos}}L_{\mathrm{pos}}
   +
   \lambda_{\mathrm{vel}}L_{\mathrm{vel}}
   +
   \lambda_{\mathrm{foot}}L_{\mathrm{foot}}.
   $$

## 实验设置

1. Text-to-motion：HumanML3D 和 KIT，指标包括 R-Precision、FID、Multimodal Distance、Diversity、Multimodality。
2. Action-to-motion：HumanAct12 和 UESTC，指标包括 FID、动作识别准确率、Diversity、Multimodality。
3. Unconstrained generation：HumanAct12 无标签设置，使用 FID、KID、Precision/Recall 等。
4. Text-to-motion 文本编码器使用冻结 CLIP-ViT-B/32；HumanML3D 模型训练 500K steps，并以 FID 选择 checkpoint。
5. 论文还做了 backbone 比较和 guidance scale sweep，并通过用户研究评估 KIT 上的主观偏好。

## 主要结果

1. HumanML3D text-to-motion：MDM top-3 R-Precision 0.611，FID 0.544，Diversity 9.559，Multimodality 2.799。它在 FID 和多样性上强，但文本检索精度低于 T2M。
2. KIT text-to-motion：MDM FID 0.497，优于 T2M 的 2.770；但 R-Precision 0.396，低于 T2M 的 0.693，说明动作质量强于文本匹配。
3. KIT 用户研究中，MDM 相比 JL2P、TEMOS、T2M 分别被偏好 90.4%、59.4%、54.8%，甚至相对 ground truth 也有 42.3% 的偏好率。
4. HumanAct12 action-to-motion：MDM FID 0.100，Accuracy 0.990；去掉 foot contact 后 FID 0.080 但物理视觉质量可能变差。
5. UESTC action-to-motion：MDM test FID 12.81，优于 ACTOR 的 23.43 和 INR 的 15.00。

## 论文贡献

1. 把 diffusion 引入人体动作生成，并给出适合 motion domain 的 Transformer sample-prediction 架构。
2. 证明直接预测 $x_0$ 可以方便加入几何损失，比纯噪声预测更适合动作约束。
3. 用 classifier-free guidance 统一 text condition、action condition 和 unconditional generation。
4. 提供了后续 text-to-motion diffusion 方法常用的强基线和开源实现。

## 局限性

1. 推理慢：论文指出单个结果大约需要 1000 次 forward pass，虽然 motion 模型比图像模型小，但仍可能接近一分钟。
2. 文本对齐并非所有指标都领先，HumanML3D 和 KIT 上 R-Precision 不总是最好。
3. Foot contact loss 有助于视觉物理合理性，但表格中的数值指标并不总能体现它的贡献。
4. 控制粒度仍有限；作者也把更好的可控生成列为未来方向。

## 与其他论文的关系

- 前置工作：[[DDPM]]、[[DDIM]]、[[CFG]] 提供 diffusion 与 classifier-free guidance 基础；MotionCLIP、T2M、TEMOS 等提供动作文本表示和评估基线。
- 后续工作：[[MotionGPT]] 等把 motion tokenization 和语言模型引入统一多任务 motion-language 框架；MLD 等方法继续沿 diffusion route 改进 motion latent representation。
- 相似方法：T2M、TEMOS、MotionCLIP 更偏 latent / VAE / CLIP 对齐；MDM 则直接在动作序列上做 diffusion。
- 主要区别：MDM 的独特性在于 clean sample prediction 和几何损失，而不仅仅是把 DDPM 套到动作数据上。

## 与当前研究方向的关系

MDM 是 text-to-human-motion 方向的重要 diffusion baseline。若后续关注文生动作、动作编辑、动作到 3D avatar 或动作驱动场景生成，MDM 提供了动作分布建模、文本条件和物理约束之间如何折中的典型方案。

## 待确认问题

- 无。

## 阅读记录

### 摘要总结

MDM 把人体动作看成一个适合 diffusion 的时序生成对象。与图像 DDPM 常见的噪声预测不同，MDM 直接预测干净动作样本，因此能在训练中加入位置、速度和脚接触损失。它在 FID 和主观质量上表现强，尤其适合建立 motion diffusion 基线。

### 粗读记录

重点读了模型设计、几何损失和 HumanML3D/KIT/HumanAct12/UESTC 结果。最值得注意的是指标分裂：MDM 生成质量和多样性强，但文本检索精度不是全面领先。这说明 motion generation 不能只看 FID，也要看语义对齐和物理可用性。

### 完整总结

MDM 的贡献不在于提出新的 diffusion 数学框架，而在于把 diffusion 细致适配到人体动作。动作数据有明确的骨架结构、时间连续性和接触约束，因此模型直接预测 clean motion $x_0$，使输出可以被位置损失、速度损失和 foot contact loss 约束。这个设计让它比噪声预测更容易嵌入 motion-specific inductive bias。

实验上，MDM 覆盖文本生成动作、类别生成动作、无条件动作和补全任务。它在 FID 上通常表现非常强，说明生成动作的分布质量接近真实数据；但 R-Precision 不是所有数据集上最佳，说明文本语义对齐仍有提升空间。整体而言，MDM 是后续 motion diffusion 和 text-to-motion 论文必须对照的 baseline。

## 用户原始备注

> 暂无。

## 引用信息

~~~bibtex
@inproceedings{tevet2023human,
  title = {Human Motion Diffusion Model},
  author = {Tevet, Guy and Raab, Sigal and Gordon, Brian and Shafir, Yonatan and Cohen-Or, Daniel and Bermano, Amit H.},
  booktitle = {International Conference on Learning Representations},
  year = {2023}
}
~~~

