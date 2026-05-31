---
title: "2D Gaussian Splatting for Geometrically Accurate Radiance Fields"
short_name: ""
authors:
  - "Binbin Huang"
  - "Zehao Yu"
  - "Anpei Chen"
  - "Andreas Geiger"
  - "Shenghua Gao"
year: 2024
venue: "SIGGRAPH 2024 Conference Papers"
url: "https://arxiv.org/abs/2403.17888"
doi: "10.1145/3641519.3657428"
arxiv: "2403.17888"
code_url: "https://github.com/hbb1/2d-gaussian-splatting"
topics: 
  - "3d-vision"
paper_type: "method"
task: "radiance-field-reconstruction"
tags:
  - literature
status: skimmed
summary_level: abstract
source_level: abstract
priority: medium
read_date: "2026-05-31"
review_date:
related_projects:
  - "https://surfsplatting.github.io/"
  - "https://github.com/hbb1/2d-gaussian-splatting"
pdf_path: ""
retrieved_from:
  - "https://arxiv.org/abs/2403.17888"
  - "https://surfsplatting.github.io/"
  - "https://github.com/hbb1/2d-gaussian-splatting"
retrieval_date: "2026-05-31"
read_version: "arXiv abstract metadata and official project page; PDF download not completed in current environment"
---


# 2D Gaussian Splatting for Geometrically Accurate Radiance Fields

## 基本信息

- 简称：
- 作者：Binbin Huang; Zehao Yu; Anpei Chen; Andreas Geiger; Shenghua Gao
- 发表时间：2024
- 会议或期刊：SIGGRAPH 2024 Conference Papers
- 原文链接：https://arxiv.org/abs/2403.17888
- arXiv：2403.17888
- DOI：10.1145/3641519.3657428
- 代码仓库：https://github.com/hbb1/2d-gaussian-splatting
- 本地 PDF 路径：未获得
- 所属方向：3d-vision
- 当前状态：skimmed

## 一句话概括

2DGS 针对 3D Gaussian Splatting 在表面几何表达上多视角不一致的问题，将体积高斯替换为一组有方向的 2D 平面高斯盘。根据摘要，方法通过透视校正的 ray-splat intersection、rasterization、深度 distortion 与 normal consistency 约束来提升几何重建质量，同时保持较快训练和实时渲染能力。

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

- [ ] 本批次仅成功获取 arXiv 摘要元数据与官方项目页；官方 PDF 多次下载过慢并被中止，未获得可读本地全文，后续需重新下载 PDF 后再写完整总结。

## 阅读记录

### 摘要总结

论文指出 3DGS 虽然能实现高质量新视角合成和快速渲染，但 3D Gaussians 的多视角不一致会限制表面几何准确性。2DGS 的核心思路是用 2D oriented planar Gaussian disks 内在地表示表面，并用透视校正 splatting、深度 distortion 与法线一致性约束改善薄结构和几何恢复。当前只读到摘要与项目页，因此不记录实验数值、公式细节或完整方法推导。

### 粗读记录

> 尚未总结。

### 完整总结

> 尚未总结。

## 用户原始备注

> 暂无。

## 引用信息

```bibtex

```
