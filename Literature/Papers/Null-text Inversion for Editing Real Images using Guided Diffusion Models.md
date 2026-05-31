---
title: "Null-text Inversion for Editing Real Images using Guided Diffusion Models"
short_name: "Null-text Inversion"
authors:
  - "Ron Mokady"
  - "Amir Hertz"
  - "Kfir Aberman"
  - "Yael Pritch"
  - "Daniel Cohen-Or"
year: 2023
venue: "CVPR 2023"
url: "https://arxiv.org/abs/2211.09794"
doi: "10.48550/arXiv.2211.09794"
arxiv: "2211.09794"
code_url: "https://github.com/google/prompt-to-prompt"
topics:
  - "diffusion-model"
  - "text-to-image"
paper_type: "method"
task: "real-image-editing"
tags:
  - literature
status: fully-summarized
summary_level: full
source_level: fulltext-and-code
priority: medium
read_date: "2026-05-31"
review_date: "2026-05-31"
related_projects:
  - "https://null-text-inversion.github.io/"
  - "https://github.com/google/prompt-to-prompt"
pdf_path: "Literature/PDFs/Null-text Inversion for Editing Real Images using Guided Diffusion Models.pdf"
retrieved_from:
  - "https://arxiv.org/abs/2211.09794"
  - "https://arxiv.org/pdf/2211.09794"
  - "https://null-text-inversion.github.io/"
  - "https://github.com/google/prompt-to-prompt"
retrieval_date: "2026-05-31"
read_version: "arXiv 2211.09794, CVPR 2023 version, PDF retrieved 2026-05-31"
---

# Null-text Inversion for Editing Real Images using Guided Diffusion Models

## 基本信息

- 简称：Null-text Inversion
- 作者：Ron Mokady; Amir Hertz; Kfir Aberman; Yael Pritch; Daniel Cohen-Or
- 发表时间：2023
- 会议或期刊：CVPR 2023
- 原文链接：https://arxiv.org/abs/2211.09794
- arXiv 链接：https://arxiv.org/abs/2211.09794
- DOI：10.48550/arXiv.2211.09794
- 代码仓库：https://github.com/google/prompt-to-prompt
- 项目页：https://null-text-inversion.github.io/
- 本地 PDF 路径：Literature/PDFs/Null-text Inversion for Editing Real Images using Guided Diffusion Models.pdf
- 所属方向：diffusion-model, text-to-image
- 当前状态：fully-summarized
- 实际阅读版本：arXiv 2211.09794, CVPR 2023 version, PDF retrieved 2026-05-31

## 一句话概括

Null-text Inversion 通过 DDIM inversion 得到真实图像的 pivot trajectory，再只优化 classifier-free guidance 中的 unconditional null-text embeddings，使 Stable Diffusion 能高保真重建真实图像并继续使用 Prompt-to-Prompt 做文本编辑。

## 研究问题

1. Prompt-to-Prompt 等文本编辑方法主要适用于模型自己生成的图像，真实图像必须先被反演进扩散模型域。
2. 直接 DDIM inversion 在使用 classifier-free guidance 时误差会被放大，导致重建质量差。
3. 如果为每张图像调模型权重或条件文本 embedding，可能损坏预训练 prior 或破坏后续编辑能力；论文希望只优化最小必要变量。

## 方法概述

1. 输入是真实图像 $I$ 和源 caption $P$，目标是在保留图像细节的同时用编辑 caption $P^\*$ 改图。
2. 第一步做 DDIM inversion，得到一条近似 diffusion trajectory $\{z_t^\*\}_{t=0}^T$。这条轨迹本身重建不完美，但可作为优化 pivot。
3. 第二步做 pivotal null-text optimization：在每个时间步 $t$，只优化 unconditional embedding $\varnothing_t$，不改模型权重，也不改 conditional prompt embedding。
4. 优化目标是让当前反向一步预测的 $z_{t-1}$ 接近 DDIM inversion 的 pivot $z_{t-1}^\*$。
5. 得到优化后的 null-text embeddings 后，可以用 Prompt-to-Prompt 的 cross-attention 控制进行真实图像文本编辑。

## 关键公式

1. DDIM sampling step：

   $$
   z_{t-1}
   =
   \sqrt{\frac{\alpha_{t-1}}{\alpha_t}}z_t
   +
   \left(
   \sqrt{\frac{1}{\alpha_{t-1}}-1}
   -
   \sqrt{\frac{1}{\alpha_t}-1}
   \right)
   \epsilon_\theta(z_t,t,C).
   $$

   论文先用反向 DDIM inversion 得到 pivot trajectory。

2. Classifier-free guidance：

   $$
   \tilde{\epsilon}_\theta(z_t,t,C,\varnothing)
   =
   w\epsilon_\theta(z_t,t,C)
   +
   (1-w)\epsilon_\theta(z_t,t,\varnothing).
   $$

   论文观察到 unconditional branch 对最终结果影响很大，因此优化 $\varnothing$。

3. 单时间步 null-text optimization：

   $$
   \min_{\varnothing_t}
   \left\|
   z_{t-1}^\*
   -
   z_{t-1}(\bar{z}_t,\varnothing_t,C)
   \right\|_2^2.
   $$

   $\bar{z}_t$ 是优化过程中当前 trajectory 上的 latent，$z_{t-1}^\*$ 是 DDIM inversion pivot。

4. 逐步 warm start：

   $$
   \varnothing_t^{(0)}
   =
   \varnothing_{t+1}^{\mathrm{opt}}.
   $$

   每个时间步的优化从后一个时间步的结果初始化，提高效率和轨迹连续性。

## 实验设置

1. 基于公开 Stable Diffusion 模型和 Prompt-to-Prompt 编辑技术。
2. 评估包括 ablation、真实图像编辑可视化、用户研究，以及将 null-text inversion 与 SDEdit 等编辑方法组合。
3. 用户研究使用 100 组图像、源 caption 和编辑 caption，比较 VQGAN+CLIP、Text2Live、SDEdit 和本文方法。
4. 论文用 PSNR 分析 inversion reconstruction，使用 LPIPS 和 CLIP similarity 分析重建忠实度与文本编辑对齐。
5. 推理时间比较包括 VQGAN+CLIP、Text2Live、SDEdit、Imagic 和本文方法。

## 主要结果

1. 用户研究中，参与者选择本文方法的比例为 65.1%，高于 Text2Live 16.6%、SDEdit 14.5% 和 VQGAN+CLIP 3.8%。
2. Ablation 显示 DDIM inversion pivot 非常关键；随机 pivot、global null-text、random caption 和 textual inversion 都难以同时保证重建和可编辑性。
3. 论文报告单次 inversion 约 1 分钟，之后每次编辑约 10 秒；相比 Imagic 的每次编辑都要调优，本文一次 inversion 可支持多次编辑。
4. 与 SDEdit 组合时，null-text inversion 在相同 CLIP alignment 下显著降低 LPIPS，说明更好保留原图身份和背景细节。
5. 定性结果显示方法能做纹理替换、对象替换、表情/背景/风格编辑，并较好保留未编辑区域。

## 论文贡献

1. 提出 diffusion pivotal inversion：用 DDIM inversion trajectory 作为每个时间步的优化锚点。
2. 提出 null-text optimization：只优化 classifier-free guidance 的 unconditional embedding，而不是模型权重或条件文本。
3. 首次将 Prompt-to-Prompt 这类文本编辑方法可靠扩展到真实图像。
4. 在重建质量、编辑能力和重复编辑效率之间给出实用折中。

## 局限性

1. 推理仍不实时：单张图 inversion 约 1 分钟，编辑约 10 秒。
2. 依赖 Stable Diffusion 的 VQ autoencoder，面对人脸等细节时可能产生重建 artifacts。
3. Stable Diffusion 的 cross-attention map 不总是精确，词语可能无法对应正确区域。
4. Prompt-to-Prompt 本身难以处理复杂结构变化，例如把坐着的狗变成站着的狗。
5. 源 caption 必须覆盖要编辑的对象或属性，否则 attention map 不足以定位编辑区域。

## 与其他论文的关系

- 前置工作：[[LDM]] / Stable Diffusion 提供 latent diffusion backbone；[[DDIM]] 提供 deterministic inversion；[[CFG]] 是 null-text branch 的来源；Prompt-to-Prompt 提供 cross-attention 编辑机制。
- 后续工作：真实图像 diffusion editing、personalized editing 和 inversion-based editing 大量借鉴 null-text optimization。
- 相似方法：Textual Inversion、DreamBooth、Imagic、SDEdit 都能编辑或个性化图像，但优化对象、是否调模型权重和是否支持多次编辑不同。
- 主要区别：Null-text Inversion 保持 conditional prompt 和模型权重不变，只调 unconditional embedding，从而兼顾重建和 Prompt-to-Prompt 可编辑性。

## 与当前研究方向的关系

这篇论文对真实图像编辑、inversion-based editing 和 diffusion controllability 很关键。它展示了 CFG 里的 unconditional 分支不只是 guidance 技巧，还可以作为每张图像的可优化控制变量。对后续 2D editing、3D editing 和基于 Stable Diffusion 的内容保真编辑都有参考价值。

## 待确认问题

- 无。

## 阅读记录

### 摘要总结

Null-text Inversion 的核心是“只调空文本”。DDIM inversion 提供初始轨迹，但在高 CFG 下会积累误差；论文用这条轨迹作 pivot，并逐时间步优化 unconditional null-text embeddings，使真实图像可被 Stable Diffusion 高保真重建和文本编辑。

### 粗读记录

重点读了方法图、CFG 公式、ablation 和用户研究。最重要的理解是：它不是为了训练新模型，而是为了给单张真实图像找到一组不破坏编辑语义的 per-timestep null embeddings。

### 完整总结

真实图像进入 diffusion editing 的难点是 inversion。直接把图像反推到噪声再采样，常常无法在高 guidance 下重建原图；而如果调模型权重或调条件 prompt embedding，又会破坏编辑能力。Null-text Inversion 的解决办法很巧：条件文本保持原样，模型权重保持原样，只优化 CFG 中代表空文本的 unconditional embedding。

方法先用 DDIM inversion 生成一条粗略轨迹，再把每个时间步的 pivot latent 作为监督信号，逐步优化 $\varnothing_t$。这样得到的轨迹能重建原图，同时 conditional prompt 的 cross-attention 结构仍保留，Prompt-to-Prompt 可以继续用于局部或全局编辑。局限主要来自运行时间、Stable Diffusion autoencoder 和 Prompt-to-Prompt 的结构编辑能力。

## 用户原始备注

> 暂无。

## 引用信息

~~~bibtex
@inproceedings{mokady2023nulltext,
  title = {Null-text Inversion for Editing Real Images using Guided Diffusion Models},
  author = {Mokady, Ron and Hertz, Amir and Aberman, Kfir and Pritch, Yael and Cohen-Or, Daniel},
  booktitle = {Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition},
  year = {2023}
}
~~~

