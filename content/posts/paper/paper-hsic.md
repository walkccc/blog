---
title:
  論文筆記 Highly-Economized Multi-View Binary Compression for Scalable Image
  Clustering
author: Peng-Yu Chen
date: 2018-09-19
tags:
  - ECCV
  - paper
---

{{< katex >}}

[Paper Link](https://arxiv.org/pdf/1809.05992.pdf)

講到 image clustering，就必需先來談談什麼是 cluster analysis（聚類分析），在維基
百科上給出了下列一段的[定義](https://zh.wikipedia.org/zh-tw/聚類分析)：

<!-- more -->

> 聚類分析（英語：Cluster analysis，亦稱為群集分析）是對於統計數據分析的一門技術
> ，在許多領域受到廣泛應用，包括機器學習，數據挖掘，模式識別，圖像分析以及生物信
> 息。聚類是把相似的對象通過靜態分類的方法分成不同的組別或者更多的子集（subset）
> ，這樣讓在同一個子集中的成員對象都有相似的一些屬性，常見的包括在坐標系中更加短
> 的空間距離等。
>
> 一般把數據聚類歸納為一種非監督式學習。

為避免混淆，此篇文章一律使用原文 image clustering，而不多做無謂的翻譯。

# 1 Introduction

Image clustering 的主要目的為：

1. 發現圖片表示的**自然**和**可解釋結構**
1. 將彼此相似的圖片分組到同一個群集中

現有的 clustering methods 主要有以下兩種：

1. single-view image clustering (SVIC)
1. multi-view image clustering (MVIC)

而現在 MVIC 較廣泛的被使用，因為他有「從單張圖片中提取多個不同特徵的靈活性」。

再更細分下去，現有的 MVIC 可再被分成三類：

1. multi-view spectral clusting
1. multi-view subspace clustering
1. multi-view matrix factorization

這是整個 paper 所提到的流程：

![](https://i.imgur.com/qeHbf9A.png)

透過 paper 中的方法，能夠將本來的資料從「歐基里德距離時間複雜度 \\(O(Nd)\\)」降
到「\\(O(1)\\)」（因為編碼成 Binary Codes 後只需做 XOR operations）！

其中，

- \\(N\\): data size（例如一張圖有幾個 pixels）
- \\(d\\): dimension（例如使用 R、G、B 三個維度來描述一個 pixel）

另外補充一下 LBP 和 HOG 的用途：

|      | LBP (Local Binary Pattern)  |                                HOG (Histogram of Oriented Gradient)                                |
| :--: | :-------------------------: | :------------------------------------------------------------------------------------------------: |
| 優點 | 1. 對光不敏感 2. 運算速度快 |                 忽略了光照顏色對圖片造成的影響，使得圖片所需要的 features 維度較低                 |
| 缺點 |      對方向資訊較敏感       | 1. 描述子生成過程冗長，導致速度慢，難以 realtime 2. 難處理遮擋問題 3. 由於梯度的性質，對噪音很敏感 |
| 用途 |     人臉識別、照片分類      |                                              行人檢測                                              |

# 2 Highly-economized Scalable Image Clustering (HSIC)

給定一個 multi-view image features 的集合：

$$\mathcal X = \\{\boldsymbol X^1, \dots, \boldsymbol X^m\\},$$

其中 \\(m\\) 代表有 \\(m\\) 個 views，而每一個 \\(\boldsymbol X^v\\)（\\(m\\) 個
views 中的第 \\(v\\) 個 view），\\(1 \le v \le m\\) 又可以表示成：

$$\boldsymbol X^v = [\boldsymbol x\_1^v, \dots, \boldsymbol x\_N^v] \in \mathbb R^{d\_v \times N},$$

其中，

- \\(d_v\\)：dimensionality
- \\(N\\): \\(\boldsymbol X^v\\) 中的 data points

整篇論文的主要目標是：**「將 \\(\mathcal X\\) 分成 \\(c\\) 個群集（clusters）。
」**

而 HSIC 的方法是：透過更低維度
的[漢明距離（Hamming space）](https://zh.wikipedia.org/zh-tw/漢明距離)進行
binary clustering。

更精確來說，透過將 multi-view features 投影到 Hamming space 中去學習以下兩點：

- common binary representation
- robust binary cluster structures

在資料預處理的步驟如下：

1. 將每個 view 中的特徵，標準化為零中心向量（zero-centered vectors）
1. 每一個 feature vector 都會由 nonlinear RBF (Radial Basis Function) kernel
   mapping 編碼，即：

   （這裡補充說明一下什麼是 nonlinear kernel mapping，主要用途為：mapping data
   到 higher dimensions，這樣一來他就有了線性的性質，讓在低維度無法線性分割的
   data，在高維度時就能線性分割。）

   $$\psi(\boldsymbol x\_i^v) = \Bigg [e^{\frac{-||\boldsymbol x\_i^v - \boldsymbol a\_1^v||^2}{\gamma}}, \dots, e^{\frac{-||\boldsymbol x\_i^v - \boldsymbol a\_l ^v||^2}{\gamma}} \Bigg]^\top \in \mathbb R^{l \times 1},$$

   其中，

   - \\(\gamma\\): 事先定義好的 kernel width
   - \\(\psi(\boldsymbol x_i^v) \in \mathbb R^{l \times 1}\\)（我們可以將
     \\(\psi\\) 函數理解成一個將 \\(\boldsymbol x_i^v\\) 由 \\(\mathbb R^{d_v
     \times 1}\\) mapping 到 \\(\psi(\boldsymbol x_i^v) \in \mathbb R^{l \times
     1}\\) 的線性轉換，其目的是因為每一個 \\(\boldsymbol x_i^v\\) 的維度
     （\\(d^v\\)）不同，一同 mapping 至 \\(\mathbb R^{l \times 1}\\) 較有利於運
     算。）
   - \\(\\{a_i^v\\}\_{i = 1}^l\\): 由 \\(\boldsymbol X^v\\) 中隨機選取 \\(l\\)
     個 anchor points（此篇 paper 取 \\(l = 1000\\)）

接下來，我們更進一步的敘述詳細的方法：

## 2.1 **Common Binary Representation Learning**

在 HSIC 中要學習的 hashing functions 共有 $K$ 個。

（補充一下：hashing functions 在 paper 中並沒有明確指出是什麼，而我個人理解
為**投影矩陣** \\(\boldsymbol (P^v)^\top = [\boldsymbol p\_1^v, \dots,
\boldsymbol p\_K^v]^\top \in \mathbb R^{K \times l}\\)，指的是找到那 \\(K\\) 個
維度為 \\(\mathbb R^{1 \times l}\\) 的 **hashing functions** \\(p_k^v\\), \\(1
\le k \le K\\)，能夠將 \\(\psi(\boldsymbol x_i^v) \in \mathbb R^{l \times 1}\\)
轉換成一個單一的 Binary Code，但共要做 \\(K\\) 次，得到一個 \\(\boldsymbol b_i
\in \mathbb R^{K \times 1}\\) 的垂直向量）

<!-- $$\psi(\boldsymbol x\_i^v) \in \mathbb R^{l \times 1} \to \boldsymbol b\_i^v = [b\_{i1}^v, \dots, b\_{iK}^v]^T \in \\{-1, 1\\}^{K \times 1} \in \mathbb R^{K \times 1}, 1 \le i \le N$$ -->

HSIC 同時也會將多個 views 的 features 投射到共同的 Hamming space，所以我們可得到
\\(\boldsymbol b_i\\)（Binary Codes）的算法如下：

$$\boldsymbol b\_i = sgn((\boldsymbol P^v)^\top \psi(\boldsymbol x\_i^v)) \in \mathbb R^{K \times 1},$$

其中，

- \\(\boldsymbol b_i\\): 不同 views 中，第 \\(i\\)-th 特徵的共同 Binary Code（即
  ：\\(\boldsymbol x_i^v, \forall v = 1, \dots, m\\)）

  （我的理解是：\\(\forall\\) views 中（\\(1, \dots, m\\)），第 \\(i\\)-th
  column 的 \\(d^v \times 1\\) 個向量，共同代表他們的 \\(\mathbb b_i \in \mathbb
  R^{K \times 1}\\) Binary Codes）

- \\(sgn(\cdot)\\): element-wise sign function

  - < 0 取 -1
  - \> 0 取 1

- \\((\boldsymbol P^v)^\top = ([\boldsymbol p\_1^v, \dots, \boldsymbol
  p\_K^v])^\top \in \mathbb R^{K \times l}\\): 第 \\(v\\)-th view 的映射矩陣
  （mapping matrix），可想像成一個將維度為 \\(\mathbb R^{l \times 1}\\) 的
  \\(\psi(\boldsymbol x_i^v)\\) mapping 到維度為 \\(K\\) 的向量空間（dim(hashing
  functions) = \\(K\\)）

由上，我們能透過最小化以下的 quantization loss 來建立學習函數：

$$
\min\_{\boldsymbol P^v, \boldsymbol b\_i} \sum\_{v = 1}^m \sum\_{i = 1}^N ||\boldsymbol b\_i - (\boldsymbol P^v)^\top \psi(\boldsymbol x\_i^v)||\_F^2.
$$

\\(\sum\_{i = 1}^N\\) 代表共有 \\(N\\) 個 \\(b_i\\) 要被學習，其中這些 \\(b_i\\)
來自 \\(\sum\_{v = 1}^m\\)。

投影 \\(\\{\boldsymbol P^v\\}\_{v = 1}^m\\) 可以補捉到能夠使相似度最大化的共享資
訊，所以我們又可以得到

$$\boldsymbol P^v = [\boldsymbol P\_S, \boldsymbol P\_I^v],$$

其中，

- \\(\boldsymbol P_S \in \mathbb R^{l \times K_S}\\): the shared projection
  across multiple views
- \\(\boldsymbol P_I^v \in \mathbb R^{l \times K_I}\\): the individual
  projection for the \\(v\\)-th view
- \\(K = K_S + K_I\\)

- \\(\boldsymbol P_I^v \in \mathbb R^{l \times K_I}\\): the individual
  projection for the \\(v\\)-th view
- \\(K = K_S + K_I\\)

因此，HSIC 可以藉由 multiple views 學到 common binary representation：

$$
\min\_{\boldsymbol P^v, \boldsymbol B, \alpha^v} \sum\_{v = 1}^m (\alpha^v)^r (||\boldsymbol B - (\boldsymbol P^v)^\top \psi(\boldsymbol X^v)||\_F^2 + \lambda\_1||\boldsymbol P^v||\_F^2),
$$

$$
s.t. \sum\_v \boldsymbol \alpha^v = 1, \boldsymbol \alpha^v > 0, \boldsymbol B = [\boldsymbol B\_s; \boldsymbol B\_I] \in \\{-1, 1\\}^{K \times N}, \boldsymbol P^v = [\boldsymbol P\_s, \boldsymbol P\_I^v],
$$

其中，

- \\(\boldsymbol B = [\boldsymbol b\_1, \dots, \boldsymbol b\_N]\\) 和
  \\(\boldsymbol \alpha = [\alpha^1, \dots, \alpha^m] \in \mathbb R^m\\) 能夠決
  定每一個不同的 views（\\(m\\) views）的權重
- \\(r > 1\\): 控制權重的分布
- \\(\lambda_1\\): regularization parameter

又透過 maximum entropy principle，即：

$$
\max \sum\_k \mathbb E[||(\boldsymbol p\_i^v)^\top \psi(\boldsymbol x\_i^v)||^2] = \frac 1 N tr((\boldsymbol P^v)^\top \psi(\boldsymbol X^v) \psi(\boldsymbol X^v)^\top \boldsymbol P^v) = g(\boldsymbol P^v).
$$

將 (2)、(3) 式合併能得到：

$$
\min\_{\boldsymbol P^v, \boldsymbol B} \sum\_{v = 1}^m (\alpha^v)^r (||\boldsymbol B - (\boldsymbol P^v)^\top \psi(\boldsymbol X^v)||\_F^2 + \lambda\_1||\boldsymbol P^v||\_F^2 - \lambda\_2 g(\boldsymbol P^v)),
$$

$$
s.t. \sum\_v \alpha^v = 1, \alpha^v > 0, \boldsymbol B = [\boldsymbol B\_s; \boldsymbol B\_I] \in \\{-1, 1\\}^{K \times N}, \boldsymbol P^v = [\boldsymbol P\_s, \boldsymbol P\_I^v],
$$

其中，

- \\(\lambda_2\\): weighting parameter

## 2.2 **Robust Binary Clust Structure Learning**

（暫時略過）
