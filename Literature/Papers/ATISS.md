---
title: "ATISS: Autoregressive Transformers for Indoor Scene Synthesis"
short_name: "ATISS"
authors:
  - "Despoina Paschalidou"
  - "Amlan Kar"
  - "Maria Shugrina"
  - "Karsten Kreis"
  - "Andreas Geiger"
  - "Sanja Fidler"
year: 2021
venue: "NeurIPS 2021"
url: "https://arxiv.org/abs/2110.03675"
doi: ""
arxiv: "2110.03675"
code_url: "https://github.com/nv-tlabs/atiss"
topics: 
  - "3d-scene-generation"
  - "neural-3d-based-generation"
paper_type: "method"
task: "indoor-scene-synthesis"
tags:
  - literature
status: fully-summarized
summary_level: full
source_level: fulltext-and-code
priority: medium
read_date: "2026-06-01"
review_date:
related_projects:
  - "https://research.nvidia.com/labs/toronto-ai/ATISS/"
pdf_path: ""
retrieved_from:
  - "https://arxiv.org/abs/2110.03675"
  - "https://arxiv.org/e-print/2110.03675"
  - "https://research.nvidia.com/labs/toronto-ai/ATISS/"
  - "https://github.com/nv-tlabs/atiss"
retrieval_date: "2026-06-01"
read_version: "arXiv v1 e-print LaTeX source; official NVIDIA project page"
---


# ATISS: Autoregressive Transformers for Indoor Scene Synthesis

## 基本信息

- 简称：ATISS
- 作者：Despoina Paschalidou; Amlan Kar; Maria Shugrina; Karsten Kreis; Andreas Geiger; Sanja Fidler
- 发表时间：2021
- 会议或期刊：NeurIPS 2021
- 原文链接：https://arxiv.org/abs/2110.03675
- arXiv：2110.03675
- DOI：
- 代码仓库：https://github.com/nv-tlabs/atiss
- 项目页：https://research.nvidia.com/labs/toronto-ai/ATISS/
- 本地 PDF 路径：
- 所属方向：3d-scene-generation, neural-3d-based-generation
- 当前状态：fully-summarized
- 实际阅读版本：arXiv v1 e-print LaTeX source; official NVIDIA project page

## 一句话概括

ATISS 将 indoor scene synthesis 建模为 unordered set generation，用一个 autoregressive transformer 根据房间类型、floor plan 和已有物体集合逐个生成下一个 3D bounding box，从而支持自动布局、任意 partial scene completion、object suggestion 和 failure correction。

## 研究问题

室内场景自动生成需要在房间形状内放置类别、尺寸、朝向和位置合理的家具。早期方法把场景当成有序序列，常按空间顺序或物体频率排序，这会让模型依赖人为顺序并限制交互式应用。例如用户给出任意 partial room 时，模型可能因为输入物体顺序“不符合训练排序”而无法继续补全。ATISS 的核心问题是：能否在只使用 3D bounding box 标注的情况下，把 scene synthesis 建模为排列不变的集合生成，同时保持 autoregressive 生成能力。

## 方法概述

1. 场景表示：每个物体是带类别、尺寸、位置、朝向的 3D bounding box；房间形状用 top-down orthographic floor layout 表示。
2. 集合生成：训练时随机打乱场景中的物体顺序，给模型任意前缀集合和 floor plan，让它预测下一个物体；这样逼近“所有排列都应有高概率”的目标。
3. 网络结构：layout encoder 用 ResNet-18 编码 floor plan；structure encoder 把已有物体属性映射到 context embeddings；transformer encoder 不加 positional encoding，从而对已有物体集合顺序保持不变；query token 预测下一个物体。
4. 属性生成：按照类别、位置、朝向、尺寸的顺序自回归预测；连续属性用 mixture of logistics distributions 建模。
5. 推理：从空集合开始反复采样下一个物体，直到生成 end symbol；再按 bounding box 尺寸从数据集中检索最接近的家具模型。
6. 交互能力：同一模型可用于 scene completion、failure detection/correction、object suggestion、object placement 等任务。

## 关键公式

1. 场景集合条件概率：

   $$
   p_{\theta}(\mathcal{O}_i \mid F^i)
   =
   \sum_{\hat{\mathcal{O}}\in\pi(\mathcal{O}_i)}
   \prod_{j\in\hat{\mathcal{O}}}
   p_{\theta}(o_j^i\mid o_{<j}^i,F^i).
   $$

2. 训练时最大化所有排列的近似 log-likelihood：

   $$
   \log \hat{p}_{\theta}(\mathcal{X})
   =
   \sum_i
   \sum_{\hat{\mathcal{O}}\in\pi(\mathcal{O}_i)}
   \sum_{j\in\hat{\mathcal{O}}}
   \log p_{\theta}(o_j^i\mid o_{<j}^i,F^i).
   $$

3. 物体属性分解：

   $$
   p_{\theta}(o_j\mid o_{<j},F)
   =
   p_{\theta}(c_j\mid o_{<j},F)
   p_{\theta}(t_j\mid c_j,o_{<j},F)
   p_{\theta}(r_j\mid c_j,t_j,o_{<j},F)
   p_{\theta}(s_j\mid c_j,t_j,r_j,o_{<j},F).
   $$

4. 连续属性的 logistic mixture 示例：

   $$
   s_j \sim \sum_{k=1}^{K}\pi_k^s\,\mathrm{logistic}(\mu_k^s,\sigma_k^s).
   $$

## 实验设置

1. 数据集：3D-FRONT，包含 6,813 套房屋和约 14,629 个房间，家具来自 3D-FUTURE。
2. 评估房型：bedrooms、living rooms、dining rooms、libraries。预处理后分别得到 5,996、2,962、2,625、622 个房间。
3. baseline：FastSynth 和 SceneFormer，均在 3D-FRONT 上重新训练；另有 Ours+Order 作为带固定顺序的变体。
4. 指标：FID、real-vs-synthetic scene classification accuracy、category KL divergence；classification accuracy 越接近 0.5 越好。
5. 人类评测：Amazon Mechanical Turk 成对比较，与 FastSynth 和 SceneFormer 比较 realism 与 error frequency。
6. 运行效率：比较生成一个场景所需时间和模型参数量。

## 主要结果

1. 量化指标：ATISS 在四类房间上取得最低或接近最低 FID，并且分类器准确率更接近 0.5。Bedroom FID 为 38.39，Living 为 33.14，Dining 为 29.23，Library 为 35.24。
2. Category KL divergence：Living、Dining、Library 上 ATISS 最低，分别为 0.0034、0.0061、0.0098；Bedroom 上略高于 FastSynth。
3. 生成速度：ATISS 生成时间约为 Bedroom 102.38 ms、Living 201.59 ms、Dining 201.84 ms、Library 88.24 ms；SceneFormer 对应约 849.37、731.84、901.17、369.74 ms；FastSynth 更慢一个到两个数量级。
4. 参数量：ATISS 为 36.053M，FastSynth 为 38.180M，SceneFormer 为 129.298M。
5. 用户研究：ATISS 的平均 error frequency 为 0.232，FastSynth 为 0.414，SceneFormer 为 0.713；ATISS 相对两种 baseline 被判断更真实的比例为 0.783。
6. 交互应用：论文展示了 scene completion、failure case detection/correction、object suggestion、scene completion from arbitrary partial rooms 等基于 unordered set formulation 的用法。

## 论文贡献

1. 首次把 indoor scene synthesis 明确建模为 autoregressive unordered set generation。
2. 设计不依赖 positional ordering 的 transformer 结构，使模型能条件化在任意 partial scene 上。
3. 只需 3D bounding box 标注，不依赖 scene graph 或 support relation annotation。
4. 同一模型支持自动生成和多种交互式 scene authoring 任务。
5. 在 3D-FRONT 上相对 FastSynth 和 SceneFormer 提升布局真实性、减少错误，并显著加快推理。

## 局限性

1. 输出主要是家具类别和 3D bounding boxes，真实几何通过检索得到，不直接生成 novel object geometry。
2. 模型仍会继承 3D-FRONT / 3D-FUTURE 的室内风格、文化和数据偏差。
3. 论文补充材料中展示的 failure cases 包括物体重叠、物体朝向不自然、缺少关键家具等，虽然发生频率较低。
4. 作者指出未来希望把 order invariance 扩展到 object attributes，并纳入 style information。
5. 本次未获得可读本地 PDF，全文阅读来源为 arXiv e-print LaTeX source 与官方项目页。

## 与其他论文的关系

- 前置工作：FastSynth、SceneFormer、3D-FRONT、3D-FUTURE、Transformer set modeling。
- 后续工作：DiffuScene、MIDI、layout-conditioned 3D scene generation、interactive indoor scene authoring。
- 相似方法：SceneFormer 也用 transformer autoregressive generation，但依赖固定顺序；ATISS 用 unordered set 使任意 partial input 成为自然条件。
- 主要区别：ATISS 生成的是布局与 bounding boxes，不是 mesh-level 或 radiance-field scene；但它是许多后续场景生成方法的布局层基础。

## 与当前研究方向的关系

在 3D Scene Generation / Neural 3D-based Scene Generation 中，ATISS 是 indoor layout generation 的基础型方法。它不追求照片级 3D 几何，而是解决“房间里应该有哪些物体、放在哪里、朝向如何”的结构问题，可作为下游 mesh generation、retrieval-based scene construction 和 text/interaction controlled scene editing 的先验。

## 待确认问题

- [ ] 本地 PDF 下载超时，已删除 partial PDF；本次全文总结基于 arXiv e-print LaTeX source 和官方项目页。
- [ ] DOI 未在 arXiv 元数据和项目页中确认。

## 阅读记录

### 摘要总结

ATISS 用 autoregressive transformer 合成室内家具布局。与把场景当有序序列的方法不同，它把已有物体当无序集合条件，训练时通过随机排列让模型学会从任意 partial scene 继续生成。这使同一个模型能做自动布局、场景补全、物体建议和异常布局修正。

### 粗读记录

论文的关键不是更复杂的几何生成，而是正确的 probabilistic formulation。室内场景天然是集合，顺序是人为附加的；去掉顺序后，模型可以对任意用户输入的 partial layout 计算 likelihood 并采样补全。实验表明它比 FastSynth/SceneFormer 更真实、更少错误、更快。

### 完整总结

完整阅读 arXiv v1 LaTeX source 后，ATISS 的核心技术路径清晰：floor layout 经 ResNet 编码，已有物体经 structure encoder 变成一组 context tokens，不使用 positional encoding 的 transformer 从这组 tokens 和 query token 中预测下一个 object attribute distribution。训练时随机选择 prefix set，使模型不依赖固定顺序。结果上，ATISS 在 FID、分类器分辨难度、用户评测、推理速度上整体优于 FastSynth 和 SceneFormer，且展示了多种交互式布局应用。

## 用户原始备注

> 暂无。

## 引用信息

```bibtex
@inproceedings{paschalidou2021atiss,
  title={ATISS: Autoregressive Transformers for Indoor Scene Synthesis},
  author={Paschalidou, Despoina and Kar, Amlan and Shugrina, Maria and Kreis, Karsten and Geiger, Andreas and Fidler, Sanja},
  booktitle={Advances in Neural Information Processing Systems},
  year={2021}
}
```
