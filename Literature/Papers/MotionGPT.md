---
title: "MotionGPT: Human Motion as a Foreign Language"
short_name: "MotionGPT"
authors:
  - "Biao Jiang"
  - "Xin Chen"
  - "Wen Liu"
  - "Jingyi Yu"
  - "Gang Yu"
  - "Tao Chen"
year: 2023
venue: "arXiv 2023"
url: "https://arxiv.org/abs/2306.14795"
doi: "10.48550/arXiv.2306.14795"
arxiv: "2306.14795"
code_url: "https://github.com/OpenMotionLab/MotionGPT"
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
  - "https://github.com/OpenMotionLab/MotionGPT"
pdf_path: "Literature/PDFs/MotionGPT.pdf"
retrieved_from:
  - "https://arxiv.org/abs/2306.14795"
  - "https://arxiv.org/pdf/2306.14795"
  - "https://github.com/OpenMotionLab/MotionGPT"
retrieval_date: "2026-05-31"
read_version: "arXiv 2306.14795, PDF retrieved 2026-05-31"
---

# MotionGPT: Human Motion as a Foreign Language

## 基本信息

- 简称：MotionGPT
- 作者：Biao Jiang; Xin Chen; Wen Liu; Jingyi Yu; Gang Yu; Tao Chen
- 发表时间：2023
- 会议或期刊：arXiv 2023
- 原文链接：https://arxiv.org/abs/2306.14795
- arXiv 链接：https://arxiv.org/abs/2306.14795
- DOI：10.48550/arXiv.2306.14795
- 代码仓库：https://github.com/OpenMotionLab/MotionGPT
- 本地 PDF 路径：Literature/PDFs/MotionGPT.pdf
- 所属方向：text-to-human-motion
- 当前状态：fully-summarized
- 实际阅读版本：arXiv 2306.14795, PDF retrieved 2026-05-31

## 一句话概括

MotionGPT 把人体动作离散成类似词表的 motion tokens，再用 T5/Flan-T5 风格语言模型统一处理 text-to-motion、motion-to-text、motion prediction 和 motion in-between，让人体动作像一门“外语”一样参与语言建模和指令调优。

## 研究问题

1. 传统 text-to-motion 方法多为单任务模型，生成、描述、预测、补全之间缺少统一接口。
2. 大语言模型已经证明 token 化序列可以承载复杂语义，但人体动作如何离散化并接入语言模型仍不清楚。
3. Motion-language 数据规模远小于语言和图像数据，模型既要利用预训练语言能力，又不能丢失动作的连续性和物理合理性。

## 方法概述

1. Motion tokenizer：先训练 VQ-VAE，把连续 3D 人体动作序列编码为离散 codebook index，也就是 motion vocabulary。
2. Motion-language vocabulary：把 motion tokens 和 text tokens 拼到统一序列中，使动作可以作为“外语句子”被语言模型读写。
3. Backbone：使用 T5 / Flan-T5 风格 encoder-decoder 模型，输入输出都可以是文本、动作 token 或两者混合。
4. 三阶段训练：先训练 motion tokenizer；再做 motion-language pre-training；最后用 instruction tuning 让模型根据不同 prompt 完成多任务。
5. 支持任务包括 text-to-motion、motion-to-text、motion prediction、motion in-between 和问答式 motion instruction following。

## 关键公式

1. VQ-VAE 量化：

   $$
   q_i = \arg\min_j \left\| h_i - e_j \right\|_2,
   \qquad
   z_i = e_{q_i}.
   $$

   连续动作 latent $h_i$ 被映射到 codebook entry $e_j$，得到离散 motion token。

2. Motion tokenizer 训练目标可概括为：

   $$
   L_{\mathrm{VQ}}
   =
   L_{\mathrm{rec}}
   +
   \left\|
   \mathrm{sg}[h] - e
   \right\|_2^2
   +
   \beta
   \left\|
   h - \mathrm{sg}[e]
   \right\|_2^2.
   $$

   其中 reconstruction loss 保证动作还原，codebook 和 commitment 项保证离散 token 可用。

3. 统一语言建模目标：

   $$
   L_{\mathrm{LM}}
   =
   -
   \sum_{t}
   \log p_\theta(s_t \mid s_{<t}, \mathrm{instruction}),
   $$

   序列 $s_t$ 可以是文本 token 或 motion token。

4. Motion tokens 解码：

   $$
   \hat{x}
   =
   \mathcal{D}_{\mathrm{motion}}
   (e_{q_1}, e_{q_2}, \ldots, e_{q_L}).
   $$

   语言模型生成的离散 motion token 最终由 motion decoder 还原为连续动作。

## 实验设置

1. HumanML3D：评估 text-to-motion、motion-to-text、motion prediction 和 motion in-between，使用 FID、Diversity、R-Precision、BLEU、CIDEr、ADE/FDE 等指标。
2. KIT：补充 text-driven motion generation 对比。
3. Motion tokenizer 采用 512 x 512 codebook，encoder 时间下采样率为 4。
4. 语言模型以 220M Flan-T5-Base 为主，论文还比较 60M、220M、770M 不同规模。
5. 训练资源：论文报告模型在 8 张 Tesla V100 上训练；补充材料给出推理速度，Base 约 222.69 FPS。

## 主要结果

1. HumanML3D 多任务表中，MotionGPT 在 text-to-motion 上 R Top1 0.492、FID 0.232、Diversity 9.528；在 motion-to-text 上 R Top3 0.827、BLEU@4 12.47、CIDEr 29.2；在 motion prediction 上 FID 0.905；在 in-between 上 FID 0.214。
2. HumanML3D text-to-motion 细表中，Fine-tuned MotionGPT Top1/Top2/Top3 R-Precision 为 0.492/0.681/0.778，FID 0.232。T2M-GPT 的 FID 0.116 更低，但 MotionGPT 提供统一多任务能力。
3. Motion captioning 中，MotionGPT Top1 0.543、Top3 0.827、BLEU@4 12.47、CIDEr 29.2，高于 TM2T 的 BLEU@4 7.00 和 CIDEr 16.8。
4. KIT text-to-motion 中，MotionGPT FID 0.510，与 T2M-GPT FID 0.514 接近，但 R-Precision 低于部分专用方法。
5. Ablation 显示 instruction tuning 提升多任务和动作生成能力，但可能牺牲纯文本描述任务表现。

## 论文贡献

1. 提出把人体动作视为一门“外语”的统一 motion-language 建模范式。
2. 用 VQ-VAE motion tokenizer 将连续动作转为离散 token，使语言模型可以生成和理解动作。
3. 将 text-to-motion、motion-to-text、prediction、in-between 等任务统一为 prompt-driven sequence-to-sequence。
4. 开源 OpenMotionLab/MotionGPT，为后续 motion LLM / embodied motion generation 提供基础代码。

## 局限性

1. 数据规模限制明显：HumanML3D 约 15k motion sequences，远小于语言和图像预训练语料。
2. 当前只建模关节人体动作，不包含脸、手、动物动作，也没有充分建模多人互动、人-物互动和人-环境互动。
3. 统一模型在多任务上灵活，但部分单任务指标仍不如专用模型，例如 T2M-GPT 的 text-to-motion FID。
4. 离散 motion token 的量化质量会限制动作细节和连续性。

## 与其他论文的关系

- 前置工作：[[MDM]] 用 diffusion 建立 text-to-motion 强基线；T2M-GPT 使用 VQ-VAE + GPT 做文本到动作生成；TM2T 支持 motion-to-text。
- 后续工作：MotionGPT 代表把 LLM / instruction tuning 引入 motion generation 的方向，后续可扩展到 avatar、具身智能和复杂交互。
- 相似方法：T2M-GPT 也是 motion token + transformer，但更偏单任务 text-to-motion；MotionGPT 强调统一多任务和 instruction prompt。
- 主要区别：MotionGPT 的核心不是更强 diffusion sampler，而是把动作 token 纳入语言模型的统一序列建模。

## 与当前研究方向的关系

如果研究目标涉及文生动作、人形角色控制、motion editing 或语言驱动动画，MotionGPT 是从 diffusion motion model 走向 motion foundation model 的关键参考。它也提示：把 3D/motion 表示离散化后接入 LLM，可能比为每个任务训练专用生成器更可扩展。

## 待确认问题

- [ ] 论文正式发表 venue 需要后续人工确认；本次仅确认 arXiv 与官方 GitHub。

## 阅读记录

### 摘要总结

MotionGPT 先把动作离散成 motion vocabulary，再用语言模型统一处理动作和文本 token。它的价值在于多任务统一接口，而不是在每个单项指标上全面超越专用模型。

### 粗读记录

重点读了 tokenizer、三阶段训练和 HumanML3D 多任务表。需要注意 MotionGPT 的优势要从“一个模型做多种任务”理解；如果只看 text-to-motion FID，它不总是优于专用方法。

### 完整总结

MotionGPT 将人体动作转成离散 token，使动作序列可以像外语一样被 seq2seq 语言模型翻译、补全和生成。它先训练 motion tokenizer，再进行 motion-language pre-training，最后用 instruction tuning 让同一个模型识别不同任务提示。这个设计把 text-to-motion、motion-to-text、motion prediction 和 in-between 都统一为 token sequence mapping。

实验表明，MotionGPT 在 HumanML3D 上能同时完成多任务，并在 motion captioning 等任务上优于专门的 TM2T。它的主要不足是数据规模和交互场景覆盖有限：当前动作主要是单人骨架，缺少脸、手、多人、人-物和环境约束。作为研究路线，它更像 motion-language foundation model 的早期形态，而不是单纯的 text-to-motion SOTA。

## 用户原始备注

> 暂无。

## 引用信息

~~~bibtex
@article{jiang2023motiongpt,
  title = {MotionGPT: Human Motion as a Foreign Language},
  author = {Jiang, Biao and Chen, Xin and Liu, Wen and Yu, Jingyi and Yu, Gang and Chen, Tao},
  journal = {arXiv preprint arXiv:2306.14795},
  year = {2023}
}
~~~

