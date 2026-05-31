---
title: "Fine Tuning Text-to-Image Diffusion Models for Subject-Driven Generation"
short_name: "DreamBooth"
authors: 
  - "Nataniel Ruiz"
  - "Yuanzhen Li"
  - "Varun Jampani"
  - "Yael Pritch"
  - "Michael Rubinstein"
  - "Kfir Aberman"
year: 2023
venue: "CVPR 2023"
url: "https://arxiv.org/abs/2208.12242"
doi: "10.48550/arXiv.2208.12242"
arxiv: "2208.12242"
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
priority: medium
read_date: "2026-05-31"
review_date: "2026-05-31"
related_projects: 
  - "https://dreambooth.github.io/"
pdf_path: "Literature/PDFs/DreamBooth.pdf"
retrieved_from: 
  - "https://arxiv.org/abs/2208.12242"
  - "https://arxiv.org/pdf/2208.12242"
  - "https://dreambooth.github.io/"
retrieval_date: "2026-05-31"
read_version: "arXiv v2, CVPR 2023 version, last revised 2023-03-15"
---

# Fine Tuning Text-to-Image Diffusion Models for Subject-Driven Generation

## 基本信息

- 简称：DreamBooth
- 作者：Nataniel Ruiz; Yuanzhen Li; Varun Jampani; Yael Pritch; Michael Rubinstein; Kfir Aberman
- 发表时间：2023
- 会议或期刊：CVPR 2023
- 原文链接：https://arxiv.org/abs/2208.12242
- arXiv 链接：https://arxiv.org/abs/2208.12242
- DOI：10.48550/arXiv.2208.12242
- 代码仓库：
- 本地 PDF 路径：Literature/PDFs/DreamBooth.pdf
- 所属方向：diffusion-model, text-to-image
- 当前状态：fully-summarized
- 实际阅读版本：arXiv v2, CVPR 2023 version, last revised 2023-03-15

## 一句话概括

DreamBooth 解决文本到图像模型无法保持特定主体身份的问题。它用 3 到 5 张主体图像微调预训练 diffusion text-to-image 模型，把一个稀有标识符绑定到该主体，并通过 class-specific prior preservation loss 避免语言漂移和类别先验崩塌。结果是用户可以用普通文本提示在新场景、新视角和新风格中生成同一主体。

## 研究问题

1. 大型文生图模型能生成“狗”“包”等类别，但不能可靠复现用户给定的某一只狗或某一个包。
2. 仅靠详细 prompt 或图像引导往往无法保持主体的关键视觉特征。
3. 少样本微调容易过拟合、降低多样性，并可能让模型把类别词本身漂移到特定实例。
4. 作者定义 subject-driven generation：给定少量主体图像，在保持主体身份的同时生成新语境、新姿态、新视角和新属性。

## 方法概述

1. 输入与输出：输入约 3-5 张同一主体图像和类别名；输出是一个个性化 text-to-image diffusion model，可用包含唯一标识符的 prompt 生成该主体。
2. Prompt 设计：训练图像统一配对为 “a [identifier] [class noun]”，例如 “a [V] dog”；类别名用于 tether class prior，唯一标识符用于绑定实例。
3. 稀有 token：作者避免使用普通英文词，改为从 tokenizer vocabulary 中选低频 token 并反解码成标识符，降低原语义先验干扰。
4. 微调策略：为最大主体保真度，作者微调整个模型；Imagen 和 Stable Diffusion 均可使用该思想。
5. Prior Preservation Loss：用冻结原模型生成 “a [class noun]” 的类别样本 $x_{pr}$，在微调时同时约束个体图像和类别先验样本，防止类别词漂移到单一实例。

## 关键公式

1. 基础 diffusion reconstruction loss（论文 Eq. 1）：

   $$
   \mathbb{E}_{x,c,\epsilon,t}[w_t\|\hat{x}_\theta(\alpha_t x+\sigma_t\epsilon,c)-x\|_2^2].
   $$

   其中 $x$ 是训练图像，$c$ 是文本条件，$\alpha_t,\sigma_t,w_t$ 来自 diffusion schedule。

2. Prior preservation loss（论文 Eq. 2）：

   $$
   \mathbb{E}[w_t\|\hat{x}_\theta(\alpha_t x+\sigma_t\epsilon,c)-x\|_2^2+\lambda w_{t'}\|\hat{x}_\theta(\alpha_{t'}x_{pr}+\sigma_{t'}\epsilon',c_{pr})-x_{pr}\|_2^2].
   $$

   第一项让模型记住给定主体；第二项用模型自身生成的类别样本保护类别先验，$\lambda$ 控制先验保持权重。

3. 训练经验参数：论文报告 Imagen 上约 1000 iterations、$\lambda=1$、learning rate $10^{-5}$，Stable Diffusion 上 learning rate $5\times10^{-6}$，3-5 张主体图像通常足够。

## 实验设置

1. 数据集：作者收集 30 个主体，21 个物体、9 个 live subjects/pets；每个主体有多个 prompt，生成四张图，总计 3000 张评估图。
2. 任务：subject recontextualization、art rendition、text-guided view synthesis、property modification、accessorization。
3. 指标：DINO 与 CLIP-I 评估 subject fidelity；CLIP-T 评估 prompt fidelity。作者偏好 DINO，因为 DINO 更不倾向忽略同类别主体差异。
4. 对比：Textual Inversion；同时比较 DreamBooth on Imagen 与 DreamBooth on Stable Diffusion。
5. 用户研究：72 位用户、25 个比较问题、共 1800 个回答；分别评估 subject fidelity 与 prompt fidelity。

## 主要结果

1. Table 1：DreamBooth (Imagen) 达到 DINO=0.696、CLIP-I=0.812、CLIP-T=0.306；Textual Inversion (Stable Diffusion) 为 DINO=0.569、CLIP-I=0.780、CLIP-T=0.255。该表说明 DreamBooth 在主体保真和 prompt 保真上都更强。
2. Table 2 用户研究：DreamBooth (Stable Diffusion) 在 subject fidelity 上获 68% 偏好，Textual Inversion 为 22%，10% undecided；prompt fidelity 上 DreamBooth 为 81%，Textual Inversion 为 12%，7% undecided。
3. Table 3 prior preservation ablation：DreamBooth (Imagen) with PPL 的 PRES=0.493、DIV=0.391、DINO=0.684、CLIP-I=0.815、CLIP-T=0.308；without PPL 的 PRES=0.664、DIV=0.371、DINO=0.712、CLIP-I=0.828、CLIP-T=0.306。PPL 降低 prior collapse 指标并提高多样性，但略牺牲主体指标。
4. Table 4 class-name ablation：正确类别名 DINO=0.744、CLIP-I=0.853；无类别名 DINO=0.303、CLIP-I=0.607；错误类别名 DINO=0.454、CLIP-I=0.728。类别名对稳定学习主体很关键。

## 论文贡献

1. 定义并系统研究 subject-driven generation / personalization 任务。
2. 提出通过稀有标识符 + 类别名微调整个 text-to-image diffusion model 来绑定特定主体。
3. 提出 autogenous class-specific prior preservation loss，缓解 language drift 和类别先验崩塌。
4. 构建 30 主体数据集和评估协议，并用 DINO/CLIP/user study 衡量主体与 prompt fidelity。
5. 展示 recontextualization、view synthesis、art rendition、property modification 等应用。

## 局限性

作者明确指出的局限性：
- 对罕见 prompt context 可能无法生成正确环境。
- context 与主体外观可能纠缠，例如环境颜色影响主体颜色。
- prompt 接近训练图像原始环境时，模型可能过拟合并生成类似训练集的图像。
- 某些主体比其他主体更容易学习；较罕见主体支持的变化更少。
- 生成图可能出现 hallucinated subject features，受模型先验强度和语义修改复杂度影响。

个人分析：
- 方法需要为每个主体微调模型，部署成本与存储成本高；也带来个性化图像滥用风险。

## 与其他论文的关系

- 前置工作：Imagen、Stable Diffusion、Textual Inversion、CLIP/DINO 评估。
- 后续工作：LoRA、personalization、subject-driven generation、DreamBooth-style fine-tuning 在 Stable Diffusion 社区广泛扩展。
- 相似方法：Textual Inversion。
- 主要区别：Textual Inversion 主要优化 token embedding；DreamBooth 微调整个模型并加入 prior preservation loss。

## 与当前研究方向的关系

DreamBooth 是文生图个性化方向的代表论文，也与文生 3D 有直接关系：许多 3D 个性化或单物体生成方法会借鉴“少样本主体绑定 + 类别先验保护”的思路。

## 待确认问题

- [ ] 未在官方项目页确认完整官方训练代码仓库；当前只记录项目页和数据集入口。
- [ ] 后续需要比较 DreamBooth 与 LoRA/Textual Inversion 在训练成本、主体保真和过拟合上的差异。

## 阅读记录

### 摘要总结

DreamBooth 用少量主体图像微调预训练文生图扩散模型，把一个稀有标识符绑定到具体主体。其 prior preservation loss 用原模型生成的类别样本保护类别先验，避免模型把类别词漂移为单一实例。

### 粗读记录

读这篇时要抓住“唯一标识符 + 类别名 + prior preservation”三件事。它不是单纯 prompt engineering，而是对模型做个性化微调。

### 完整总结

完整阅读 arXiv v2 / CVPR 2023 版本，并查看官方项目页。论文提出 subject-driven generation：从 3-5 张图学习一个特定主体，然后在新 prompt 下保持主体身份。方法上，训练图像用 “a [V] [class noun]” 标注，微调整个模型；同时生成类别先验图像加入 prior preservation loss。实验证明 DreamBooth 在 DINO/CLIP-I/CLIP-T 与用户研究中明显优于 Textual Inversion；PPL 降低 prior collapse 并增加多样性，但略牺牲主体指标。论文也明确讨论了 prompt context 失败、外观纠缠、过拟合和 hallucinated features。

## 用户原始备注

> 暂无。

## 引用信息

```bibtex
@article{ruiz2022dreambooth,
  title={DreamBooth: Fine Tuning Text-to-image Diffusion Models for Subject-Driven Generation},
  author={Ruiz, Nataniel and Li, Yuanzhen and Jampani, Varun and Pritch, Yael and Rubinstein, Michael and Aberman, Kfir},
  booktitle={arXiv preprint arxiv:2208.12242},
  year={2022}
}
```
