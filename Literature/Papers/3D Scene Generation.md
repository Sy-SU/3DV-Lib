---
title: "3D Scene Generation: A Survey"
short_name: "3D Scene Generation Survey"
authors:
  - "Beichen Wen"
  - "Haozhe Xie"
  - "Zhaoxi Chen"
  - "Fangzhou Hong"
  - "Ziwei Liu"
year: 2025
venue: "arXiv 2025"
url: "https://arxiv.org/abs/2505.05474"
doi: ""
arxiv: "2505.05474"
code_url: "https://github.com/hzxie/Awesome-3D-Scene-Generation"
topics: 
  - "3d-scene-generation"
  - "survey"
paper_type: "survey"
task: "scene-generation"
tags:
  - literature
status: fully-summarized
summary_level: full
source_level: fulltext
priority: medium
read_date: "2026-05-31"
review_date:
related_projects:
  - "https://github.com/hzxie/Awesome-3D-Scene-Generation"
pdf_path: "Literature/PDFs/3D Scene Generation.pdf"
retrieved_from:
  - "https://arxiv.org/abs/2505.05474"
  - "https://arxiv.org/pdf/2505.05474.pdf"
  - "https://github.com/hzxie/Awesome-3D-Scene-Generation"
retrieval_date: "2026-05-31"
read_version: "arXiv v1 PDF, 27 pages, dated 2025-05-08; project repository metadata from Batch 003"
---


# 3D Scene Generation: A Survey

## 基本信息

- 简称：3D Scene Generation Survey
- 作者：Beichen Wen; Haozhe Xie; Zhaoxi Chen; Fangzhou Hong; Ziwei Liu
- 发表时间：2025
- 会议或期刊：arXiv 2025
- 原文链接：https://arxiv.org/abs/2505.05474
- arXiv：2505.05474
- DOI：
- 项目页：https://github.com/hzxie/Awesome-3D-Scene-Generation
- 本地 PDF 路径：Literature/PDFs/3D Scene Generation.pdf
- 所属方向：3d-scene-generation, survey
- 当前状态：fully-summarized

## 一句话概括

这篇综述把 3D scene generation 系统整理为 procedural generation、neural 3D-based generation、image-based generation 和 video-based generation 四类，并进一步梳理场景表示、生成模型、数据集、评估指标、下游应用、挑战与未来方向。

## 研究问题

论文关注的问题是：3D scene generation 研究快速增长，但已有综述常聚焦程序化生成、室内场景、自动驾驶、text-driven generation、一般 3D/4D 生成或 world models，缺少一个覆盖场景级生成、3D 表示、图像/视频扩展范式、数据集和评估协议的统一分类。

作者把 3D scene generation 定义为生成空间结构化、语义有意义、视觉真实的 3D 环境，应用包括沉浸式媒体、游戏、AR/VR、机器人仿真、自动驾驶、embodied AI 和 world models。与单个物体或 avatar 生成相比，场景生成更难，因为它涉及更大尺度、多对象关系、稀缺的高质量 3D 场景数据，以及对布局、区域、风格等细粒度控制的需求。

## 方法概述

论文不是提出新算法，而是建立一套层次化 taxonomy。

首先，论文回顾常用 3D scene representations：

- Voxel grid：结构化体素，适合显式占据或 SDF，但分辨率和内存成本受限。
- Point cloud：稀疏点集，来自深度传感器、LiDAR 或 SfM，存储高效但缺少显式拓扑。
- Mesh：用顶点、边和面表示表面，便于传统图形管线和物理交互。
- Neural fields：包括 SDF 和 NeRF，以连续隐式函数表达几何或辐射场。
- 3D Gaussians：用带位置、各向异性协方差、颜色和透明度的高斯 primitive 表示场景，渲染效率高。
- Image sequence：用多视角图像序列隐式编码场景结构，是 image/video-based 方法常用表示。

其次，论文总结生成模型基础：autoregressive model、VAE、GAN、diffusion model 和 procedural generator。然后按生成范式把方法分为四类：

- Procedural generation：用显式规则、优化约束或 LLM 调用程序化系统生成场景。
- Neural 3D-based generation：训练 3D-aware 模型生成 scene parameters、scene graph、semantic layout 或 implicit layout，再得到 mesh、NeRF、3D Gaussians 等场景表示。
- Image-based generation：利用 2D image generator 生成全景或沿轨迹逐步外推图像，再通过深度、点云、mesh、NeRF 或 3DGS 等方式维持几何一致性。
- Video-based generation：把 3D/4D 场景生成转化为单视角或多视角视频生成，常用于动态环境、开放世界游戏和自动驾驶 world model。

## 关键公式

论文用一个通用映射定义 3D scene generation：

$$
G: x \to S
$$

其中 $x$ 可以是随机噪声、文本、图像或其他条件，$S$ 是生成的 3D scene representation。

典型场景表示包括：

$$
V \in \mathbb{R}^{H \times W \times D}
$$

表示 voxel grid；

$$
P = \{p_i \mid p_i \in \mathbb{R}^3\}_{i=1}^{N}
$$

表示 point cloud；

$$
\mathcal{G} = \{(\mu_i, \Sigma_i, c_i, \alpha_i)\}_{i=1}^{N}
$$

表示 3D Gaussian primitives，其中 $\mu_i$ 是中心，$\Sigma_i$ 是各向异性形状，$c_i$ 是颜色，$\alpha_i$ 是不透明度。

程序化生成可概括为状态递推：

$$
S_{t+1} = R(S_t, \Theta)
$$

其中 $R$ 是规则集合，$\Theta$ 是可调参数。

论文还回顾了 GAN 与 diffusion 的通用公式。GAN 的目标写作：

$$
\min_G \max_D \mathbb{E}_{x \sim p_{\mathrm{data}}(x)}[\log D(x)] + \mathbb{E}_{z \sim p_z(z)}[\log(1 - D(G(z)))]
$$

扩散前向加噪过程写作：

$$
x_t = \sqrt{1-\beta_t}x_{t-1} + \sqrt{\beta_t}\epsilon_t
$$

其中 $\epsilon_t$ 是 Gaussian noise，$\beta_t$ 控制噪声调度。

## 实验设置

作为 survey，论文没有提出单一实验 pipeline，而是从以下角度组织已有工作：

- 方法分类：用 Table 1 比较四类范式在 realism、diversity、view consistency、semantic consistency、efficiency、controllability、physical plausibility 上的取舍。
- 代表工作：用 Table 2 汇总不同类别的代表方法，标注 scene type、condition、3D scene representation 和 generation model。
- 数据集：用 Table 3 汇总室内、自然、城市三类数据集，记录来源、图像数、场景数、面积、3D model 与 annotation 类型。
- 评估：按 metrics-based、benchmark-based 和 human evaluation 三类整理。
- 应用：单独讨论 3D scene editing、human-scene interaction、embodied AI、robotics 和 autonomous driving。

## 主要结果

论文的主要结论是 taxonomy 和 trade-off，而不是单一模型性能：

- Procedural methods 具备 3D 一致性、可控性和图形引擎兼容性，但受规则、资产和人工设计限制，纹理和多样性可能不足。
- Neural 3D-based methods 能利用 NeRF、3D Gaussians、scene graph、semantic layout 等 3D prior 维持结构一致性，但依赖昂贵的 3D 数据和 annotation，效率与可控性仍受限。
- Image-based methods 利用强大的 2D 生成模型获得高 photorealism 和多样性，但 depth accuracy、长程 semantic consistency 和跨视角一致性是主要难点。
- Video-based methods 通过 temporal modeling 提供动态场景和较强视觉连续性，适合 world model、自动驾驶和游戏环境，但 view alignment 和 3D geometric consistency 仍不稳定。
- 数据集层面，室内数据相对丰富，真实自然场景数据仍少，城市/驾驶数据既有真实采集也有仿真生成，但物理 affordance、材质、交互等高层 annotation 仍不足。
- 评估层面，FID、KID、IS、CLIP Score、FVD、FVMD、depth/pose error、collision rate 等各自覆盖局部维度，缺少统一、场景级、物理一致且任务对齐的评估协议。

## 论文贡献

- 给出覆盖 procedural、neural 3D-based、image-based、video-based 四大范式的 3D scene generation 分类。
- 统一回顾 voxel、point cloud、mesh、neural fields、3D Gaussians、image sequence 等表示，并解释它们与不同生成路线的关系。
- 系统整理常用数据集、评估指标、benchmark 和 human evaluation 方式。
- 把 3D scene generation 与 scene editing、human-scene interaction、embodied AI、robotics、autonomous driving 等应用连接起来。
- 总结当前挑战：生成能力、3D 表示、数据与 annotation、评估协议，并提出未来方向：更高保真度、physics-aware generation、interactive scene generation、unified perception-generation。

## 局限性

这篇综述覆盖面很广，但因此对每个具体方法的实现细节和实验设置展开有限。论文主要给出结构化分类与代表性方法汇总，不提供统一复现实验或跨方法重新评测。由于领域发展很快，2025 年之后的新方法需要通过作者维护的项目页继续跟踪。

## 与其他论文的关系

- 前置工作：综述中把 3DGS、NeRF、Infinigen、3D-GPT、3D Cinemagraphy、Text2NeRF、LucidDreamer 等作为不同范式的代表节点。
- 后续工作：适合作为本库 3D scene generation 主题的总览入口，后续新论文可以挂到其四类 taxonomy 下。
- 相似方法：与单一方法论文不同，它不解决某个具体生成任务，而是建立领域地图。
- 主要区别：它把 procedural/LLM、3D-aware neural generation、image-to-3D expansion、video/world-model 生成放在同一框架中比较。

## 与当前研究方向的关系

这篇综述应作为 3D Scene Generation 主题导航的核心参考。它能帮助整理现有笔记中的多个分支：procedural generation 连接 3D-GPT 和 Infinigen，image-based generation 连接 3D Cinemagraphy，neural 3D-based generation 连接 NeRF/3DGS/2DGS，video-based generation 连接动态 3D/4D scene 和 world model。

对 Obsidian 图谱来说，它适合作为 topic/survey 级节点，而不是与每篇 paper 逐一形成管理型高出度链接。应在 `Literature/Topics/3D Scene Generation.md` 中保留 curated link，而避免在 `Literature Index.md` 或 Retrieval Log 中批量链接。

## 待确认问题

- [ ] 本次只阅读 arXiv v1 PDF；正式发表版本、DOI 或后续修订版本待后续人工确认。
- [ ] 项目仓库是持续更新的 Awesome list，本次没有逐项审查仓库中新增条目的完整性。

## 阅读记录

### 摘要总结

论文把 3D scene generation 定义为合成空间结构化、语义有意义且照片真实的 3D 环境，应用包括沉浸式媒体、机器人、自动驾驶和 embodied AI。摘要称综述将方法分为四个范式：程序化生成、神经 3D 表示驱动生成、图像驱动生成和视频驱动生成，并分析技术基础、权衡、代表性结果、数据集、评估协议和下游应用。

### 粗读记录

粗读确认论文的主线是 taxonomy。它先用表示和生成模型搭建背景，再把方法拆为四大类，并用表格列出代表方法、场景类型、条件输入和 3D 表示。后半部分重点转向数据、评估和应用，尤其强调 scene-level 评估、物理合理性、交互性和 perception-generation 统一模型仍是开放问题。

### 完整总结

完整阅读 PDF 后，确认这篇综述适合作为本库 3D scene generation 方向的总览文献。它的价值在于把看似分散的方向组织为四条路线：程序化/LLM 控制、3D-aware neural generation、图像驱动的全景或迭代外推、视频驱动的动态世界生成。论文还明确指出每类路线的核心 trade-off：procedural 强可控与一致性但真实感受限，neural 3D-based 有结构 prior 但受数据与效率限制，image-based 真实感强但几何一致性弱，video-based 支持动态与世界模型但 view alignment 不稳。数据与评估部分对后续整理本库很有用，尤其是把 dataset 按 indoor/natural/urban 分类，把 metric 按 fidelity、spatial consistency、temporal coherence、controllability、diversity、plausibility 分类。

## 用户原始备注

> 暂无。

## 引用信息

```bibtex

```
