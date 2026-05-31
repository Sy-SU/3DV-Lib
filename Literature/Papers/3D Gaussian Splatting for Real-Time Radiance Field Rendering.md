---
title: "3D Gaussian Splatting for Real-Time Radiance Field Rendering"
short_name: ""
authors:
  - "Bernhard Kerbl"
  - "Georgios Kopanas"
  - "Thomas Leimkühler"
  - "George Drettakis"
year: 2023
venue: "ACM Transactions on Graphics 42(4), SIGGRAPH 2023"
url: "https://arxiv.org/abs/2308.04079"
doi: ""
arxiv: "2308.04079"
code_url: "https://github.com/graphdeco-inria/gaussian-splatting"
topics: 
  - "3d-vision"
paper_type: "method"
task: "novel-view-synthesis"
tags:
  - literature
status: skimmed
summary_level: abstract
source_level: abstract
priority: medium
read_date: "2026-05-31"
review_date:
related_projects:
  - "https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/"
  - "https://github.com/graphdeco-inria/gaussian-splatting"
pdf_path: ""
retrieved_from:
  - "https://arxiv.org/abs/2308.04079"
  - "https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/"
  - "https://github.com/graphdeco-inria/gaussian-splatting"
retrieval_date: "2026-05-31"
read_version: "arXiv abstract metadata and official project page; PDF download not completed in current environment"
---


# 3D Gaussian Splatting for Real-Time Radiance Field Rendering

## 基本信息

- 简称：
- 作者：Bernhard Kerbl; Georgios Kopanas; Thomas Leimkühler; George Drettakis
- 发表时间：2023
- 会议或期刊：ACM Transactions on Graphics 42(4), SIGGRAPH 2023
- 原文链接：https://arxiv.org/abs/2308.04079
- arXiv：2308.04079
- DOI：
- 代码仓库：https://github.com/graphdeco-inria/gaussian-splatting
- 本地 PDF 路径：未获得
- 所属方向：3d-vision
- 当前状态：skimmed

## 一句话概括

3D Gaussian Splatting 面向多视角图像或视频采集场景的新视角合成，目标是在保持高视觉质量的同时达到实时渲染。根据摘要，方法从 SfM 稀疏点出发，用 3D Gaussians 表示场景，进行交替优化与密度控制，并引入 visibility-aware splatting renderer 支持 1080p 级别实时显示。

## 研究问题

> 尚未总结。

## 方法概述

> 尚未总结。

## 关键公式

> 尚未总结。

## 实验设置

> 尚未总结。

## 主要结果

> 尚未总结。

## 论文贡献

> 尚未总结。

## 局限性

> 尚未总结。

## 与其他论文的关系

- 前置工作：
- 后续工作：
- 相似方法：
- 主要区别：

## 与当前研究方向的关系

> 尚未总结。

## 待确认问题

- [ ] 本批次仅成功获取 arXiv 摘要元数据与官方项目页；官方 PDF 下载过慢并被中止，未获得可读本地全文，后续需重新下载 PDF 后再写完整总结。

## 阅读记录

### 摘要总结

论文针对 NeRF 类 radiance field 方法训练和渲染成本高、实时性不足的问题，提出以 3D Gaussians 作为显式场景表示。摘要中的三个关键点是：从相机标定得到的稀疏点初始化并优化 3D Gaussians；通过交替优化和密度控制，尤其是各向异性协方差优化，提高场景表示精度；使用 visibility-aware 渲染算法加速训练和实时渲染。当前只读到摘要与项目页，因此不记录实验数值、公式细节或完整方法推导。

### 粗读记录

> 尚未总结。

### 完整总结

> 尚未总结。

## 用户原始备注

> 暂无。

## 引用信息

```bibtex

```
