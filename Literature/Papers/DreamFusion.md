---
title: "Text-to-3D using 2D Diffusion"
short_name: "DreamFusion"
authors: 
  - "Ben Poole"
  - "Ajay Jain"
  - "Jonathan T. Barron"
  - "Ben Mildenhall"
year: 2022
venue: "arXiv"
url: "https://arxiv.org/abs/2209.14988"
doi: "10.48550/arXiv.2209.14988"
arxiv: "2209.14988"
code_url: ""
topics: 
  - "text-to-3d"
paper_type: "method"
task: "text-to-3d"
tags:
  - literature
status: fully-summarized
summary_level: full
source_level: fulltext
priority: medium
read_date: "2026-05-31"
review_date: "2026-05-31"
related_projects: 
  - "https://dreamfusion3d.github.io/"
pdf_path: "Literature/PDFs/DreamFusion.pdf"
retrieved_from: 
  - "https://arxiv.org/abs/2209.14988"
  - "https://arxiv.org/pdf/2209.14988"
  - "https://dreamfusion3d.github.io/"
retrieval_date: "2026-05-31"
read_version: "arXiv v1, submitted 2022-09-29"
---

# Text-to-3D using 2D Diffusion

## 基本信息

- 简称：DreamFusion
- 作者：Ben Poole; Ajay Jain; Jonathan T. Barron; Ben Mildenhall
- 发表时间：2022
- 会议或期刊：arXiv
- 原文链接：https://arxiv.org/abs/2209.14988
- arXiv 链接：https://arxiv.org/abs/2209.14988
- DOI：10.48550/arXiv.2209.14988
- 代码仓库：
- 本地 PDF 路径：Literature/PDFs/DreamFusion.pdf
- 所属方向：text-to-3d
- 当前状态：fully-summarized
- 实际阅读版本：arXiv v1, submitted 2022-09-29

## 一句话概括

DreamFusion 解决缺少大规模带文本 3D 数据时如何做 text-to-3D 的问题。它不训练 3D diffusion model，而是把冻结的 2D text-to-image diffusion model 当作先验，通过 Score Distillation Sampling 优化一个随机初始化的 NeRF，使随机视角渲染符合文本。结果可从任意角度查看、重光照，并可导出 mesh，但受 2D prior、SDS 和 NeRF 优化局限影响。

## 研究问题

1. 文生图扩散模型依赖海量图文数据，而 text-to-3D 缺少同等规模的带文本 3D 数据，也缺少高效的 3D denoising 架构。
2. Dream Fields 等 CLIP-guided text-to-3D 方法能利用 2D 模型，但生成质量和几何一致性有限。
3. 作者要解决：能否直接复用强 2D diffusion prior，不用 3D 数据、不改 2D 模型，优化出一个 coherent 3D asset。

## 方法概述

1. 输入与输出：输入文本 caption；输出为一个从零优化出的 NeRF-like 3D scene，可多视角渲染、重光照、组合到 3D 环境。
2. 2D prior：使用冻结的 Imagen 64x64 base text-to-image diffusion model，不使用 super-resolution cascade。
3. Score Distillation Sampling：将 NeRF 渲染图加噪到 timestep $t$，让 Imagen 预测噪声；用预测噪声与注入噪声的 residual 作为图像空间更新方向，反传到 NeRF 参数。
4. NeRF 表示：基于 mip-NeRF 360 思路，MLP 输出密度和 albedo；用密度梯度估计 normal，并用 Lambertian shading 生成受控光照。
5. 优化循环：每步随机采样相机和光源，渲染 64x64 图像，追加 view-dependent text conditioning，计算 SDS 梯度，优化 15,000 iterations。
6. 几何改进：使用大视角范围、view-dependent prompts、lighting、textureless renders、opacity/orientation regularizers，减少“画在平面上”的退化解。

## 关键公式

1. SDS 梯度（论文 Eq. 3）：

   $$
   \nabla_\theta L_{SDS}(\phi,x=g(\theta)) = \mathbb{E}_{t,\epsilon}[w(t)(\hat{\epsilon}_\phi(z_t;y,t)-\epsilon)\frac{\partial x}{\partial \theta}].
   $$

   $\hat{\epsilon}_\phi$ 是冻结 Imagen 的噪声预测，$\epsilon$ 是注入噪声，$g(\theta)$ 是可微图像参数化（这里是 NeRF 渲染）。该式省略 U-Net Jacobian，让扩散模型像冻结 critic 一样给出图像编辑方向。

2. SDS 的 KL 解释（论文 Eq. 4）：

   $$
   \nabla_\theta L_{SDS}=\nabla_\theta \mathbb{E}_t[\sigma_t/\alpha_t w(t)KL(q(z_t \mid g(\theta);y,t)\|p_\phi(z_t;y,t))].
   $$

   作者用它说明 SDS 不是任意 heuristic，而是概率密度蒸馏相关目标的梯度。

3. NeRF volume rendering（论文 Eq. 5）：

   $$
   C=\sum_i w_i c_i,\quad w_i=\alpha_i\prod_{j<i}(1-\alpha_j),\quad \alpha_i=1-\exp(-\tau_i\|\mu_i-\mu_{i+1}\|).
   $$

   该式把沿相机射线采样点的颜色和密度合成为像素颜色。

4. Lambertian shading（论文 Eq. 7）：

   $$
   c=\rho\circ(\ell_\rho\circ\max(0,n\cdot(\ell-\mu)/\|\ell-\mu\|)+\ell_a).
   $$

   albedo $\rho$、normal $n$、点光源 $\ell$ 和 ambient light 一起生成受控光照，帮助暴露几何。

## 实验设置

1. 任务：zero-shot text-to-3D generation，无 3D 或多视角训练数据。
2. 评估集：Dream Fields object-centric COCO validation subset 的 153 个 prompts。
3. 指标：CLIP R-Precision；同时评估 color renders 和 textureless geometry renders，后者用于检查是否只是把纹理画到平面上。
4. 对比方法：Dream Fields、Dream Fields reimplementation、CLIP-Mesh、MS-COCO ground-truth images。
5. 消融：逐步加入 ViewAug、ViewDep、Lighting、Textureless，观察 albedo、shaded、textureless render 的 R-Precision 和几何质量。
6. 优化设置：每个 prompt 从随机 NeRF 初始化训练；随机相机 elevation $[-10^\circ,90^\circ]$、azimuth $[0^\circ,360^\circ]$、distance [1,1.5]；$t\sim U(0.02,0.98)$；CFG guidance weight $\omega=100$；15,000 iterations，约 1.5 小时。

## 主要结果

1. Table 1：DreamFusion 在 CLIP B/32 上 color R-Precision=75.1、geometry R-Precision=42.5；CLIP B/16 上 color=77.5、geometry=46.6；CLIP L/14 上 color=79.7、geometry=58.5。
2. Table 1：DreamFusion 的 color R-Precision 接近 GT Images（CLIP B/32 77.1、CLIP B/16 79.1），并高于 Dream Fields / CLIP-Mesh 的多数 color scores。
3. Table 1：Dream Fields reimplementation 的 geometry R-Precision 仅 1.3 / 0.8 / 1.4，而 DreamFusion 在对应 geometry render 上为 42.5 / 46.6 / 58.5，说明 SDS + 几何策略改善了 3D coherence。
4. Figure 6 消融：作者报告加入 view-dependent prompts、lighting、textureless renders 后 geometry 明显改善，full render R-Precision 提升 +12.5%。

## 论文贡献

1. 提出 Score Distillation Sampling，把冻结 2D diffusion model 用作可微参数化图像/3D 优化的 prior。
2. 展示无需 3D 数据、无需修改 2D diffusion model 的 text-to-3D 流程。
3. 将 SDS 与 NeRF-like 表示、随机多视角渲染、view-dependent prompts、lighting 和 textureless regularization 组合起来改善几何。
4. 在 CLIP R-Precision 和可视化结果上显示相对 Dream Fields / CLIP-Mesh 的更好文字一致性与几何一致性。
5. 官方项目页提供大量交互式结果和 mesh export 展示。

## 局限性

作者明确指出的局限性：
- SDS 用于图像采样并不完美，容易产生 oversaturated、oversmoothed 结果。
- SDS 结果相比 ancestral sampling 多样性较低；不同 random seeds 的 3D 结果差异较少，可能与 reverse KL 的 mode-seeking 性质有关。
- 使用 Imagen 64x64 base model，生成的 3D 模型缺少细节；更高分辨率扩散模型和更大 NeRF 会使合成速度不实际。
- 从 2D observation lift 到 3D 本质上 ill-posed，仍会出现内容画到单个平面的局部最小值。
- 继承 Imagen 的数据偏差、LLM 条件偏差和潜在滥用风险。

个人分析：
- 论文完整 DreamFusion 实现未公开，且 Imagen 不公开，复现依赖替代扩散模型和开源 NeRF 组件。

## 与其他论文的关系

- 前置工作：DDPM、CFG、Imagen、NeRF/mip-NeRF 360、Dream Fields、CLIP-Mesh。
- 后续工作：Score Distillation Sampling 成为 ProlificDreamer、Magic3D、text-to-3D 系列的重要基础。
- 相似方法：Dream Fields 用 CLIP loss 优化 NeRF；DreamFusion 用 2D diffusion score distillation 替代 CLIP。
- 主要区别：DreamFusion 使用扩散模型噪声残差作为优化方向，并显式加入几何正则与渲染策略。

## 与当前研究方向的关系

DreamFusion 是当前文献库 text-to-3D 方向的关键起点。它把 DDPM/CFG 里的 2D diffusion prior 迁移到 3D asset 优化，是后续 VSD、SDS 改进和 text-to-3D 质量提升论文的共同参照。

## 待确认问题

- [ ] 官方完整 DreamFusion 代码未确认公开；论文仅说明 MultiNeRF 组件公开、Imagen 模型不公开。
- [ ] 需要继续读 ProlificDreamer，比较 SDS 与 VSD 如何缓解过饱和、过平滑和低多样性问题。
- [ ] DreamFusion 的 CLIP R-Precision 指标是否充分反映几何质量，需要结合后续 3D metrics 论文审视。

## 阅读记录

### 摘要总结

DreamFusion 用冻结的 2D text-to-image diffusion model 指导 3D NeRF 优化。核心 SDS 梯度把 diffusion 模型预测噪声和实际噪声的 residual 反传到 NeRF 参数，使随机视角渲染逐步符合文本 prompt。

### 粗读记录

重点记住 SDS：不训练 3D diffusion，不需要 3D 数据，而是用 2D diffusion prior 优化一个可微 3D 表示。它的成功和问题都来自这个设置：强 prior 带来生成能力，也带来 oversmoothing、低多样性和 3D 歧义。

### 完整总结

完整阅读 arXiv v1 并查看官方项目页。论文提出把 2D Imagen 当作冻结 prior，用 SDS 更新 NeRF 参数。每次迭代随机采样相机和光源，渲染 NeRF，给渲染图加噪，让 Imagen 预测噪声，然后把 residual 反传回 NeRF。作者用 view-dependent prompts、lighting、textureless rendering 和正则项解决多脸、平面贴图等几何退化。评估上，DreamFusion 的 color R-Precision 接近真实 COCO 图像，textureless geometry R-Precision 明显高于 Dream Fields reimplementation。局限性集中在 SDS 的 mode-seeking/oversmoothing、低分辨率 Imagen、3D 歧义和代码/模型不可完全复现。

## 用户原始备注

> 暂无。

## 引用信息

```bibtex
@article{poole2022dreamfusion,
  author = {Poole, Ben and Jain, Ajay and Barron, Jonathan T. and Mildenhall, Ben},
  title = {DreamFusion: Text-to-3D using 2D Diffusion},
  journal = {arXiv},
  year = {2022}
}
```
