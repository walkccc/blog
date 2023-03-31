---
title:
  論文筆記 Realtime Multi-Person 2D Pose Estimation using Part Affinity Fields
author: Peng-Yu Chen
date: 2018-08-31
tags:
  - CVPR
  - paper
math: true
---

[Paper Link](https://arxiv.org/pdf/1611.08050.pdf)

# 1. Introduction

這篇 paper 是卡內基美隆大學（Carnegie Mellon University）在
[CVPR 2017](https://cvpr2017.thecvf.com) 所發表的，論文中首先指出了 Pose
Estimation 中三個具挑戰性的關鍵：

1. 一張圖片裡有多少人，而這些人擺什麼姿勢和人的大小？
1. 有幾個人是相互疊在一起（overlap）的，他們彼此摭蓋面積？
1. 無法即時（realtime）

另外論文中也提到了一些現有方法存在的瓶頸，現有方法主要是透過 top-down 的方式：

1. person detector
1. single-person pose estimation

來解決此類問題，而這很依賴效能，如果 person detector 失敗了，那方法就沒用了，另
外時間也是一項考驗。

# 2. Method

**Figure 2:**

![](https://i.imgur.com/eHuzvJM.jpg)

Fig. 2 給出了模型的整個處理過程：

1. 讀進一張圖片大小為 $w \times h$ 的圖片 $\textbf I$。
1. 送進 model VGG-19 的前 10 層 layer train 出大小一樣為 $w \times h$ 的
   features $\textbf F$。
1. 再送進 paper 中提到的 model，會得到以下兩個：

   - 2D confidense maps
     $\textbf S = (\textbf S_1, \textbf S_2, \dots, \textbf S_J)$，其中 $J$ 代表
     人體共有 $J$ 個部位（part）。

     $$\textbf S_j \in \mathbb R^{w \times h}, j \in \\{1 \dots J\\}.$$

   - 2D vector fields
     $\textbf L = (\textbf L_1, \textbf L_2, \dots, \textbf L_C)$ ，其中 $C$ 代
     表兩個 part 之間連線總數。

     $$\textbf L_c \in \mathbb R^{w \times h \times 2}, c \in \\{1, \dots, C\\}.$$

1. 再將 confidence maps $\textbf S$ 和 affinity fields $\textbf L$ 送到 greedy
   inference，就能產生所有人的 2D keypoints。

## 2.1 Simultaneous Detection and Association

**Figure 3:**

![](https://i.imgur.com/aVpp6Qm.png)

Fig. 3 給出了模型示意圖，圖片輸入進去，然後同時 預測出 confidence maps
$\textbf S$ 和 affinity fields $\textbf L$。

神經網路分成兩個部分：

- 上方淺橘色部分預測 confidence map
- 下方淺藍色部分預測 affinity fields

每個分支都是一個遞迴的預測結構，整個 model 包含了 $T$ 個 stage，每個 stage 中都
加入中間監督（intermediate supervision）
，[這是用來處理 vanishing gradient 的問題]()。

1. 圖片首先經由微調過的 VGG19 前十層得到一組大小為 $w \times h$ 的 feature maps
   $\textbf F$，將其做為 input 輸入到兩個分支裡第一個 stage。
1. 在第一個 stage 中，網路輸出：
   - detection confidence maps $\textbf S^1 = \rho^1(\textbf F)$
   - part affinity fields $\textbf L^1 = \phi^1(\textbf F)$ 其中 $\rho^1$ 和
     $\phi^1$ 表示第一個 stage 的 CNN 架構。
1. 在往後每個 stage 中，模型會將每前個階段的輸出和 $\textbf F$（原本的 feature
   maps）做 concatenate 後，送給下一個 stage 當 input，輸出 refined
   predictions，寫成數學式如下：

   $$\textbf S^t = \rho^t(\textbf F, \textbf S^{t - 1}, \textbf L^{t - 1}), \forall t \ge 2,$$

   $$\textbf L^t = \phi^t(\textbf F, \textbf S^{t - 1}, \textbf L^{t - 1}), \forall t \ge 2,$$

   其中 $\rho^t$ 和 $\phi^t$ 為第 $t$ 階段的 CNN。

**Figure 4:**

![](https://i.imgur.com/7kFwQiu.jpg)

Fig. 4 秀出了每一 個階段 confidence maps 和 affinity fields 改善的情況。

在預測的 predictions 和 groundtruth maps and fileds 使用了 loss function $L_2$，
論文中特別提到 loss functions 是隨著空間而變的（spatially），因為有些 datasets
不見得會完整地標示所有人。

在 $t$ 階段中的 loss functions 如下：

$$f\_{\textbf S}^t = \sum_{j = 1}^J \sum\_{\textbf p} \textbf W(\textbf p) \cdot || \textbf S_j^t(\textbf p) - \textbf S_j^\*(\textbf p)||\_2^2,$$
$$f\_{\textbf L}^t = \sum_{c = 1}^C \sum\_{\textbf p} \textbf W(\textbf p) \cdot || \textbf L\_c^t(\textbf p) - \textbf L\_c^\*(\textbf p)||\_2^2,$$
$$f = \sum\_{t = 1}^T (f\_\textbf S^t + f\_\textbf L^t).$$

其中：

- $\textbf S_j^*$: groundtruth part confidence map
- $\textbf L_c^*$: groundtruth part affinity vector field
- $\textbf W$: binary mask 且 $\textbf W(\textbf p) = 0$ 當 annotation 在位置
  $\textbf p$ 不存在，這是用來避免在 training 的過程中，即使正確預測了，仍有
  penalty。

## 2.2 Confidence Maps for Part Detection

下邊給出根據 annotation 計算 groundtruth confidence maps $\textbf S^\*$ 的方法，
每個 confidence map 都是一個 2D 的表示。理想情況下，

- 當圖片中只包含一個人時：如果一個 keypoint 是可見的話，對應的 confidence map 中
  只有一個峰值。
- 當圖片中有多個人時：對於每一個人 $k$ 的每一個可見 keypoint $j$，在對應的
  confidence map 中都會有一個峰值。

詳細方法如下：

1.  先找出每個人 $k$ 的某一部位 $j$

    - 每一個人 $k$ 的單個 confidence maps $\textbf S\_{j, k}^\*$ 和
    - $\textbf x\_{j, k} \in \mathbb R^2$ 表示圖片中人 $k$ 的 part $j$ 對應的
      groundtruth position

    計算方式如式 (6) 所示，其中 $\sigma$ 用來控制峰值在 confidence map 中的傳播
    範圍。

    （這裡可以理解成，$\forall \textbf p \in \mathbb R^2$，$\textbf p$ 點越接近
    $\textbf x\_{j, k} \in \mathbb R^2$，$||\textbf p - \textbf x\_{j, k}||\_2^2$
    值趨近於 $0$，$\textbf S\_{j, k}^\*(\textbf p)$ 也就越靠近極大值 $1$。）

    $$\textbf S\_{j, k}^\*(\textbf p) = \exp \Big(- \frac{||\textbf p - \textbf x\_{j, k}||\_2^2}{\sigma^2}\Big),$$

1.  再找出所有人的部位 $j$，這裡取最大值而不是平均值能夠更準確地將同一個
    confidence map 中的峰值保存下來，即：對整張圖 $w \times h$ 每一個點，找該點
    在所有人之中的最大值！

$$\textbf S\_j^\*(\textbf p) = \max\_k \textbf S\_{j, k}^\*(\textbf p).$$

## 2.3 Part Afﬁnity Fields for Part Association

給定一組 keypoints，如 Fig.5(a) 所示，我們如何把它們組裝成，未知數量人的整個身體
的 pose 呢？

我們需要一個好方法來確定每對 keypoints 之間的連接，即：它們屬於同一個人。

一個可能的方法是找到一個位於每一對 keypoints 之間的一個中間點，後檢查中間點是真
正的中間點的機率，如 Fig. 5(b) 所示。但是當人們擠在一起時，中間點可能是錯誤的連
線，如 Fig. 5(b) 中綠線所示。出現這種情況的原因有兩個：

1. 這種方式只編碼了位置資訊，沒有方向
1. 身體的支撐區域已經縮小到一個點上。

為解決這些限制，我們提出了稱為 PAF(part affinity fields) 的特徵表示來保存身體的
支撐區域的位置信息和方向信息，如 Fig. 5(c) 所示。對於每一條軀幹來說，the part
affinity 是一個 2D 的向量區域。在屬於一個軀幹上的每一像素都對應一個 2D 的向量，
這個向量表示軀幹上從一個 keypoint 到另一個 keypoint 的方向。

![](https://i.imgur.com/0Wc2cEg.png)

考慮下圖中給出的一個軀幹（手臂），令 $\textbf x\_{j\_1, k}$ 和
$\textbf x\_{j\_2, k}$ 表示圖中的某個人 $k$ 的兩個 keypoints 對應的真實像素點，
如果一個像素點 $\textbf p$ 位於這個軀幹上，$\textbf L\_{c, k}^\*(\textbf p)$ 表
示一個從 keypoint $j\_1$ 到 keypoints $j\_2$ 的單位向量，對於不在軀幹上的像素點
，對應的向量則是 $\textbf 0$。

![](https://i.imgur.com/RFDqCzD.png)

下面這個公式給出了 the groundtruth part affinity vector，對於圖片中的一個點
$\textbf p$ 其值 $\textbf L\_{c, k}^\*(\textbf p)$ 的值如下：

$$
\textbf L\_{c, k}^\*(\textbf p) =
\begin{cases}
\textbf v \text{ if $\textbf p$ on limb $c, k$}; \\\\
\textbf 0 \text{ otherwise.}
\end{cases}
$$

其中，

- $\textbf v = (\textbf x\_{j\_2, k} - \textbf x\_{j\_1, k}) / ||\textbf x\_{j\_2, k} - \textbf x\_{j\_1, k}||\_2$:
  軀幹對應的單位方向向量。屬於這個軀幹上的像素點滿足下面的不等式：

$$0 \le \textbf v \cdot (\textbf p - \textbf x\_{j\_1, k}) \le l\_{c, k} \text{ and } |\textbf v\_\bot \cdot (\textbf p - \textbf x\_{j\_1, k})| \le \sigma\_l.$$

其中，

- $\sigma\_l$: limb 寬度（注意：不同於軀幹）
- 軀幹長度：$l\_{c, k} = ||\textbf x\_{j\_2, k} - \textbf x\_{j\_1, k}||\_2$
- $\textbf v\_\bot$: 垂直於 $\textbf v$ 的向量

整張圖片的 the groundtruth part affinity field 取圖片中所有人對應的 affinity
field 的平均值，其中 $n\_c(\textbf p)$ 是圖片中 $k$ 個人在像素點 $\textbf p$ 對
應的非零向量的個數，即：我們只考慮
$\forall \textbf p \in \mathbb R^2$，$\forall k$ 個人中，有向量的平均。

$$\textbf L\_c^\*(\textbf p) = \frac{1}{n\_c(\textbf p)} \sum_k \textbf L\_{c, k}^\*(\textbf p).$$

在預測的時候，我們用候選 keypoints 之間的 PAF 來衡量這對 keypoints 是不是屬於同
一個人。詳細的說，對於兩個候選 keypoints 對應的像素點 $\textbf d\_{j\_1}$ 和
$\textbf d\_{j_2}$，我們去計算 PAF，如下式所示：

$$E = \int\_{u = 0}^{u = 1} \textbf L\_c(\textbf p(u)) \cdot \frac{\textbf d\_{j\_2} - \textbf d\_{j\_1}}{||\textbf d\_{j\_2} - \textbf d\_{j\_1}||\_2}du,$$

其中 $\textbf p(u)$ 表示兩個像素點 $\textbf d\_{j\_1}$ 和 $\textbf d\_{j\_2}$ 之
間的像素點：

$$\textbf p(u) = (1 - u)\textbf d\_{j\_1} + u \textbf d\_{j\_2}.$$

## 2.4 Multi-Person Parsing using PAFs

藉由 non-maximum suppression，我們從預測出的 confidence maps 得到一組離散的
keypoints 候選位置。因為圖片中可能有多個人或者存在 false positive，每個 keypoint
可能會有多個候選位置，因此也就組成了很大數量的 keypoints pair，如 Fig. 6(b) 所示
。按照式 (10)，我們給每一個候選 keypoints pair 計算一個分數。

從這些 keypoint pair 中找到最佳結果，是一個 NP-Hard 問題。下面給出本文的方法：

![](https://i.imgur.com/HjueVFa.png)

假設模型得到的所有候選 keypoints 組成集合
$\mathcal D\_\mathcal J = \\{\textbf d\_j^m: \text{ for } j \in \\{1 \dots J\\}, m \in \\{1 \dots N\_j\\}\\},$

其中，

- $N\_j$: keypoint $j$ 的候選位置數量
- $\textbf d\_j^m \in \mathbb R^2$: keypoint $j$ 的第 $m$ 個候選位置的像素坐標。

我們需要做的是將屬於同一個人的 keypoints 連成軀幹（胳膊、腿等），為此我們定義變
數 $z\_{j\_1j\_2}^{mn} \in \\{0, 1\\}$ 表示候選 keypoints $\textbf d\_{j\_1}^m$
和 $\textbf d\_{j\_2}^n$ 是否可以連起來。如此以來便得到了集合

$$\mathcal Z = \\{z\_{j\_1j\_2}^{mn} \in \\{0, 1\\}: \text{ for } j\_1, j\_2 \in \\{1 \dots J\\}, m \in \\{1 \dots N\_{j\_1}\\}, n \in \\{1 \dots N\_{j\_2}\\}\\}.$$

現在單獨考慮第 $c$ 個軀幹（例如脖子），其對應的兩個 keypoints 應該是 $j\_1$ 和
$j\_2$，這兩個 keypoints 對應的候選集合分別是 $\mathcal D\_{j\_1}$ 和
$\mathcal D\_{j\_2}$，可透過線性方程式如下找出正確 keypoints：

$$\max\_{\mathcal Z\_c} E\_c = \max\_{\mathcal Z\_c} \sum\_{m \in \mathcal D\_{j\_1}}\sum\_{n \in \mathcal D\_{j\_2}} E\_{mn} \cdot z\_{j\_1j\_2}^{mn},$$
$$\text{s.t. } \forall m \in \mathcal D\_{j\_1}, \sum\_{n \in \mathcal D\_{j\_2}} z\_{j\_1j\_2}^{mn} \le 1,$$
$$\forall n \in \mathcal D\_{j\_2}, \sum\_{m \in \mathcal D\_{j\_1}} z\_{j\_1j\_2}^{mn} \le 1$$

其中，

- $E\_c$: 軀幹 $c$ 對應的權值總和
- $\mathcal Z\_c$: 軀幹 $c$ 對應的 $\mathcal Z$ 的子集
- $E\_{mn}$: keypoint $d\_{j\_1}^m$ 和 keypoint $d\_{j\_2}^n$ 對應的 part
  affinity

式 (13) 和式 (14) 限制了任意兩個相同類型的軀幹（例如兩個脖子）不會共享關鍵點。問
題擴展到所有 $C$ 個軀幹上，我們優化目標就變成了公式 (15)。

$$\max\_\mathcal Z E = \sum_{c = 1}^C \max\_{\mathcal Z\_c} E\_c.$$

# 3. Results

## 3.1. Results on the MPII Multi-Person Dataset

## 3.2. Results on the COCO Keypoints Challenge

## 3.3. Runtime Analysis

# 4. Discussion
