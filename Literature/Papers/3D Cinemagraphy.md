---
title: "3D Cinemagraphy from a Single Image"
short_name: "3D Cinemagraphy"
authors:
  - "Xingyi Li"
  - "Zhiguo Cao"
  - "Huiqiang Sun"
  - "Jianming Zhang"
  - "Ke Xian"
  - "Guosheng Lin"
year: 2023
venue: "CVPR 2023"
url: "https://arxiv.org/abs/2303.05724"
doi: ""
arxiv: "2303.05724"
code_url: "https://github.com/xingyi-li/3d-cinemagraphy"
topics: 
  - "3d-scene-generation"
  - "image-based-generation"
paper_type: "method"
task: "single-image-animation"
tags:
  - literature
status: skimmed
summary_level: abstract
source_level: abstract
priority: medium
read_date: "2026-05-31"
review_date:
related_projects:
  - "https://xingyi-li.github.io/3d-cinemagraphy/"
  - "https://github.com/xingyi-li/3d-cinemagraphy"
pdf_path: ""
retrieved_from:
  - "https://arxiv.org/abs/2303.05724"
  - "https://xingyi-li.github.io/3d-cinemagraphy/"
  - "https://github.com/xingyi-li/3d-cinemagraphy"
retrieval_date: "2026-05-31"
read_version: "arXiv abstract metadata and official project page; PDF download not completed in current environment"
---


# 3D Cinemagraphy from a Single Image

## 基本信息

- 简称：3D Cinemagraphy
- 作者：Xingyi Li; Zhiguo Cao; Huiqiang Sun; Jianming Zhang; Ke Xian; Guosheng Lin
- 发表时间：2023
- 会议或期刊：CVPR 2023
- 原文链接：https://arxiv.org/abs/2303.05724
- arXiv：2303.05724
- DOI：
- 代码仓库：https://github.com/xingyi-li/3d-cinemagraphy
- 本地 PDF 路径：未获得
- 所属方向：3d-scene-generation, image-based-generation
- 当前状态：skimmed

## 一句话概括

3D Cinemagraphy 试图从单张静态图像生成同时包含场景内容动画和相机运动的视频。根据摘要，作者认为直接组合 2D 图像动画和 3D photography 容易产生伪影，因此把图像转换为 feature-based layered depth images 和 feature point cloud，再把 2D motion lifting 到 3D scene flow 后进行双向位移和新视角合成。

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

- [ ] 本批次仅成功获取 arXiv 摘要元数据与官方项目页；CVF/项目 PDF 下载过慢并被中止，未获得可读本地全文，后续需重新下载 PDF 后再写完整总结。

## 阅读记录

### 摘要总结

论文目标是将单图动画与 3D 摄影结合，从一张输入图像生成既有视觉动态又有相机运动的视频。摘要描述的流程包括：预测深度并构建 feature-based layered depth images，反投影为 feature point cloud；估计 2D motion 并提升为 3D scene flow；对点云做双向位移，再投影和融合以合成目标视角。当前只读到摘要与项目页，因此不记录实验数值、公式细节或完整方法推导。

### 粗读记录

> 尚未总结。

### 完整总结

> 尚未总结。

## 用户原始备注

> 暂无。

## 引用信息

```bibtex

```
