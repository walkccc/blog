---
title: 論文筆記 Pointwise Convolutional Neural Networks
author: Peng-Yu Chen
date: 2018-09-27
tags:
  - CVPR
  - paper
math: true
---

[Paper Link](https://arxiv.org/pdf/1712.05245.pdf)

在這篇 paper 中，展示了使用 CNNs 去處理 3D point clouds，並且應用在 semantic
segmentation 和 object recognition 中。

MathWorks 官網有對 Semantic Segmantation 做了一個基礎
的[介紹](https://www.mathworks.com/help/vision/ug/semantic-segmentation-basics.html)：

分割（Segmantation）對於圖像分析任務非常重要。語義分割（Segmantic Segmantation）
描述了將圖片中的每個像素與類標籤（class label），例如：花、人、道路、天空、海洋
或汽車，相關聯的過程。

![](https://i.imgur.com/o9O4kU3.png)

# 1. Introduction

現有的深度學習在處理 3D 資料，像是體積、point clouds、或 multi-view 圖片仍有不少
困難。

- Volume representation 最能夠「完整地」描述一張圖片，直觀上也是最容易實做的，但
  是礙於硬體資源（記憶體、容量等），所以不大可行。
- Multi-view representation 雖然不是最真實的描述方式，但能夠透過事先訓練好的 2D
  模型來加以實現。
- Point clouds 不但資訊量較小，而且較為彈性，但 point clouds 與神經網路的應用還
  未被深入挖掘，這也是此篇 paper 誕生的原因。

綜合來說，整篇 paper 的重點在於：

- A pointwise convolution operator that can output features at each point in a
  point cloud;
- Two pointwise convolutional neural networks for semantic scene segmentation
  and object recognition.

![](https://i.imgur.com/Y99QhI5.png)

# 2. Related Works

## 2.1. Shape descriptors

Hand-crafted 的 shape descriptors 在深度學習誕生以前，時常被運用在各種不同的
computer vision 應用上，直到深度學習的出現，傳統方法的眼淚也滴下來了。

## 2.2. Object recognition

結語：CNNs 被廣泛運用在 CV 和 AI 領域已經不是一兩天的事了。

像 [ImageNet](http://image-net.org) 就是一個著名的大型 RGB 3 維 image
dataset，CNNs 能夠從中成功地學習到 image descriptors 並大勝傳統方法。

而 [PointNet](https://arxiv.org/abs/1612.00593) 是第一個能夠處理 point cloud
data 的網路架構，他能夠學習到順序不變性（order-invariance）的函式。

此外，[PointCNN](https://arxiv.org/abs/1801.07791) 也探索了 equivariance 而不是
invariance 的觀點，並且得到和 PointNet 可比的效能。

為了達成縮放的目的，通過建立計算圖型，例如：

- [octree](https://zh.wikipedia.org/zh-tw/八叉樹)
- [kd-tree](https://zh.wikipedia.org/zh-tw/K-d樹)

也很常見。

Stanford 提出的 PointNet 雖然效能很猛，但網路複雜，這篇 paper 提出了相對簡單的
pointwise convolution，且能達到和 PointNet、PointCNN 等相仿的準確度。

## 2.3. Semantic segmentation

由 Silberman 所提出
[NYUv2](https://cs.nyu.edu/~silberman/datasets/nyu_depth_v2.html) dataset，帶來
了 RGB-D semantic segmentation 的風潮。

RGB-D 圖片其實是兩張圖片：

- 普通的 RGB 3 維圖片
- Depth 圖片（類似灰階圖片，只是每個像素值是距離物體的實際距離）

[SegNet](https://arxiv.org/abs/1511.00561) 就在此 dataset 得到不錯的效果，他使用
的方法為：

- encoder-decoder
- dilation filter

[McCormac](https://arxiv.org/abs/1609.05130) 則能透過 2D 預測 3D domain，但這些
預測並無法被直接應用在 3D domain。

[SSCNet](http://www.cs.toronto.edu/~rjliao/papers/iccv_2017_3DGNN.pdf) 應用 CNN
在 3D volume representation 去分類每個像素。

# 3. Pointwise Convolution

先來談談要麼**描述**一個 3D 物件，[VoxNet]() 使用了 $64 \times 64 \times 64$ 的
解析度去描述一個物件，但這有一個很大的缺點，很耗費記憶體，因為其實大部分的像素（
立體三維空間中的）皆是 $0$，但這可以被
[sparse representation](https://arxiv.org/abs/1611.05009) 解決。

Point clouds 能夠被 RGB-D reconstruction 和 CAD modeling 的特性，因此也是一個不
錯的表示法，PointNet 就是基於此產生的，但將 point cloud 餵給神經網路是不自然的，
因為傳統的 convolution operators 只設計給 grid 和 volumes。

## Convolution

每一個 kernel

- 以 point cloud 中每一個點為中心
- 有一個 size 或 radius value（可被調整）

![](https://i.imgur.com/Phxw1Mi.png)

數學示可表示成：

$$
x\_i^\ell =
\sum\_k w\_k \frac{1}{|\Omega\_i(k)|}
\sum\_{p\_j \in \Omega\_i(k)} x\_j^{\ell - 1},
\tag{1}
$$

其中，

- $k$：所有 sub-domains
- $\Omega\_i(k)$：當 kernel 以 point $i$ 為中心時，第 $k$-th sub-domain
- $p\_i$：point $i$ 的座標
- $|\cdot|$：計算所有 sub-domain 裡的 points 數量
- $w\_k$：$k$-th sub-domain 的 kernel weight
- $x\_i$ 和 $x\_j$：point $i$ 和 $j$ 的值
- $\ell - 1$ 和 $\ell$：input 和 output index

## Gradient backpropagation

令 $L$ 為 loss function，gradient 可被表示成：

$$
\frac{\partial L}{\partial x\_j^{\ell - 1}} =
\sum\_{i \in \Omega\_j}
\frac{\partial L}{\partial x\_i^\ell}
\frac{\partial x\_i^\ell}{\partial x\_j^{\ell - 1}},
\tag{2}
$$

我們遍歷所有點 $j$ 的 鄰居點 $i$，同時
$\partial x\_i^\ell / \partial x\_j^{\ell - 1}$ 可被寫成：

$$
\frac{\partial x\_i^\ell}{\partial x\_j^{\ell - 1}} =
\sum\_k w\_k
\frac{1}{|\Omega\_i(k)|}
\sum\_{p\_j \in \Omega\_i(k)} 1
\tag{3}
$$

$$
\frac{\partial L}{\partial w\_k} =
\sum\_i
\frac{\partial L}{\partial x\_i^\ell}
\frac{\partial x\_i^\ell}{\partial w\_k}
\tag{4}
$$

其中，

$$
\frac{\partial x\_i^\ell}{\partial x\_k} =
\frac{1}{|\Omega\_i(k)|}
\sum\_{p\_j \in \Omega\_i(k)}
x\_j^{\ell - 1}
\tag{5}
$$

上方的公式並沒有假定 convolution kernel 有固定的形狀，在此篇 paper 中，所有的
convolution kernels 大小皆為 $3 \times 3 \times 3$，而所有點的 weights 皆一樣大
。

和傳統立體 convolution 不同的是，他不使用 pooling。paper 提出以下的優點：

1. 不再需要處理 downsampling 和 upsampling
1. 鄰居 query 的加速結構只需要建構一次

## Point order

和 PointNet 顯著不同的是，如何將點餵進網路中？

在 PointNet 中，point cloud 是沒有順序性的，但在此篇 paper 中，他認為順序是有必
要的。他將 input points 根據特定的 order 做排序，例如：XYZ 或
[Morton curve](https://en.wikipedia.org/wiki/Z-order_curve)。

![](https://i.imgur.com/jXhD9vZ.jpg)

在 object recognition 中，順序性是必要的，但在 semantic segmentation 中，利用每
個點的局部特徵，因此不需要點順序。

## À-trous convolution

透過新增一項 stride 參數，可以將原始逐點卷積擴展到 à-trous (dilated) 卷積。透過
增大 kernel size，感知範圍了，這有助於加度訓練卻不失精度。

[GIF 來源](https://github.com/vdumoulin/conv_arithmetic)

| ![](https://raw.githubusercontent.com/vdumoulin/conv_arithmetic/master/gif/padding_strides.gif) | ![](https://raw.githubusercontent.com/vdumoulin/conv_arithmetic/master/gif/dilation.gif) |
| :---------------------------------------------------------------------------------------------: | :--------------------------------------------------------------------------------------: |
|                                        Padding, strides                                         |                             No padding, no stride, dilation                              |

## Point attributes

為了方便實現卷積，我們分別儲存點坐標和其他點屬性，例如顏色、法線或從先前卷積層輸
出的其他高維特徵。

# 4. Evaluations

## Semantic segmantation

|            ![](https://i.imgur.com/R3vdK5V.png)            |                 ![](https://i.imgur.com/fCDEoRe.png)                  |                  ![](https://i.imgur.com/pNtepqQ.png)                   |
| :--------------------------------------------------------: | :-------------------------------------------------------------------: | :---------------------------------------------------------------------: |
| Table 1: Comparison of scene segmentation on S3DIS dataset | Table 2: Per-class accuracy of semantic segmentation on S3DIS dataset | Table 3: Per-class accuracy of semantic segmentation on SceneNN dataset |

|    ![](https://i.imgur.com/KbfIQLW.png)    |   ![](https://i.imgur.com/gogbSN3.png)   |
| :----------------------------------------: | :--------------------------------------: |
| Semantic segmentation on the S3DIS dataset | Semantic segmentation on SceneNN dataset |

## Object recognition

|                                         ![](https://i.imgur.com/uBLyVBt.png)                                          |                    ![](https://i.imgur.com/5RhdvWQ.png)                    |
| :-------------------------------------------------------------------------------------------------------------------: | :------------------------------------------------------------------------: |
| Table 4: Comparison of performance of network architectures using 3D object representations on the ModelNet40 dataset | Table 5: Comparison of object recognition accuracy on the ObjectNN dataset |
