---
title:
  "論文筆記 PointNet++: Deep Hierarchical Feature Learning on Point Sets in a
  Metric Space"
author: Peng-Yu Chen
date: 2018-10-14
tags:
  - NIPS
  - paper
categories:
  - paper
  - Chinese
---

[Paper Link](http://papers.nips.cc/paper/7095-pointnet-deep-hierarchical-feature-learning-on-point-sets-in-a-metric-space.pdf)

# 1 Introduction

PointNet 對局部特徵的處理並不完善，在 3D Object Part Segmentation 和 Semantic
Segmentation in Scenes 中（即：需要得到每一個點的分數時），原方法是將全局特徵
concatenate 在單點特徵後面，中間忽略了局部特徵的步驟，於是作者提出了 PointNet++
採用分層神經網路（hierarchical neural network）來對此改善。

另外，點雲的密度是不固定的，因此作者也在 PointNet++ 中提出了**密度適應**的網路結
構。

就全局來看，PointNet++ 比 PointNet 多了：

1. 局部特徵提取
1. 密度適應網路

# 2 Problem Statement

假設 \\(X = (M, d)\\) 是繼承歐基里德空間 \\(\mathbb R^n\\) 的離散度量空間，

其中：

- \\(M \subseteq \mathbb R^n\\)：點集
- \\(d\\)：是距離度量。

另外，歐基里德空間中的 $M$ 密度在各處可能不均勻。我們感興趣的是學習 set function
$f$，$f$ 的輸入為 $\mathcal X$（以及每個點的附加特徵）並產生重新劃分
$\mathcal X$ 的 semantic 訊息。

# 3 Method

## 3.1 Review of PointNet [20]: A Universal Continuous Set Function Approximator

## 3.2 Hierarchical Point Set Feature Learning

|                                                                                                                            ![](https://i.imgur.com/QTaXC0n.png)                                                                                                                            |
| :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| Figure 2: Illustration of our hierarchical feature learning architecture and its application for set segmentation and classiﬁcation using points in 2D Euclidean space as an example. Single scale point grouping is visualized here. For details on density adaptive grouping, see Fig. 3 |

這裡簡單說明一下網路架構：

**set abstraction**:

- 對點雲中的點進行局部劃分
- 提取整體特徵

簡單來說：set abstraction = sampling + grouping + pointnet

### Sampling layer

藉由 farthes point sampling (FPS) 來選取一 points 子集，相較與隨機選取，FPS 能更
好的覆蓋給定 centroid 的點集。

### Grouping layer

採用了 3.3 節的 MSG 和 MRG 方法。

## 3.3 Robust Feature Learning under Non-Uniform Sampling Density

|                      ![](https://i.imgur.com/gSS3JQB.png)                      |
| :----------------------------------------------------------------------------: |
| Figure 3: (a) Multi-scale grouping (MSG); (b) Multi-resolution grouping (MRG). |

### Multi-scale grouping (MSG)

把每種不同半徑的特徵到抓出來，但 MSG 有一個運算效能上的問題，因此作者提出了
MRG。

### Multi-resolution grouping (MRG)

MRG 由兩部分向量構成：

- 上一層即 $L\_{i - 1}$ 層的向量
- 輸入點雲上提取的特徵

權重調配方式：

- 點稀疏時，給從點雲提取的特徵較高權重
- 點稠密時，則給 $L\_{i - 1}$ 層提取的向量較高的權重，因為此時點雲的抽象程度可能
  不夠，而從 $L\_{i - 1}$ 層能讓我們看的更廣些。

## 3.4 Point Feature Propagation for Set Segmentation
