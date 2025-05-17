---
title: 論文筆記 Video-to-Video Synthesis
author: Peng-Yu Chen
date: 2018-11-26
tags:
  - NIPS
  - paper
---

[Paper Link](https://arxiv.org/pdf/1808.06601.pdf)

現在 image to image 的 tasks 已經是當紅炸子雞，NVIDIA 這回和 MIT CSAIL 直接更上
一層樓，開發出了 video to video 的轉換系統，並且能夠支援到 2K 的解析度，就讓我們
來看看這篇 vid2vid 吧！

# 1 Introduction

建模和動態地重建視覺世界的能力對於構建 intelligent agents 是至關重要的。除了科學
興趣導向外，學習連續視覺經驗在計算機視覺、機器人和計算機圖形學中有廣泛的應用。

|                                                         ![](https://tcwang0509.github.io/vid2vid/paper_gifs/cityscapes_comparison.gif)                                                         |
| :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| Figure 1: Generating a photorealistic video from an input segmentation map video on Cityscapes. Top left: input. Top right: `pix2pixHD`. Bottom left: `COVST`. Bottom right: `vid2vid` (ours). |

# 2 Related Work

## Generative Adversarial Networks (GANs)

Generator 和 discriminator 要儘量達成 zero-sum。本篇 paper 使用的主要方法為
conditional GAN。此篇 paper 的方法不只是預測未來影片對當前觀察到圖片，並且在可操
作的語義表示上合成照片般逼真的影片，例如 segmentation masks、草圖、姿勢。

## Image-to-image translation

不但希望圖片擬真，還希望能製照出 temporally coherant frames。這在長時間的影片是
非常具有挑戰性的。

## Unconditional video synthesis

因為 unconditional 的設定，VGAN、TGAN 和 MoCoGAN 都得到這解析度較差、時間較短的
影片。

## Future video prediction

作者在生成未來影片時，是根㯫「當前已經存在的影片」，所以有別於傳統的生成方式，會
遇到 regress-to-the-mean 的問題，本篇做出來的結果往往會好很多，解析度也較高。

## Video-to-video synthesis

# 3 Video-to-Video Synthesis

令 \\(\\textbf s_1^T \equiv \\{\\textbf s_1, \\textbf s_2, \dots, \\textbf
s_T\\}\\)：一系列用於視頻合成的源圖像。例如：semantic segmentation masks。

令 \\(\\textbf x_1^T \equiv \\{\\textbf x_1, \\textbf x_2, \dots, \\textbf
x_T\\}\\)：對應的真實圖片。

Video-to-video synthesis 的目標是學習一個：從 \\(\\text s_1^T\\) mapping 到
\\(\\tilde{\\textbf x}\_1^T \equiv \\{\\tilde{\\textbf x}\_1, \\tilde{\\textbf
x}\_2, \dots, \\tilde{\\textbf x}\_T\\}\\) 的 function，因此給定 \\(\\textbf
s_1^T\\) 所產生的條件分佈，\\(\\tilde{\\textbf x}\_1^T\\) 要和給定 \\(\\textbf
s_1^T\\) 所產生的條件分佈 \\(\\textbf x_1^T\\) 相同。

$$
p(\tilde{\\textbf x}\_1^T \mid \\textbf s\_1^T) = p(\\textbf x\_1^T \mid \\textbf s\_1^T). \tag{1}
$$

本篇 paper 使用的架構為 [conditional GAN](https://arxiv.org/pdf/1411.1784.pdf)。

令 \\(G\\)（generator）：將 input source sequence 映射到對應的 output image
sequence，即 \\(\\textbf x_1^T = G(\\textbf s_1^T)\\)，我們藉由底下的 minimax
optimization problem 來訓練 generator：

$$
\max\_D \min\_G E\_{(\textbf x\_1^T, \textbf s\_1^T)} [\log D(\textbf x\_1^T, \textbf s\_1^T)] + E\_{\textbf s\_1^T} [\log(1 - D(G(\textbf s\_1^T), \textbf s\_1^T))], \tag{2}
$$

## Sequential generator

![](https://imgur.com/qy0vdo8.png)

[圖片來源](https://www.youtube.com/watch?v=GrP_aOSXt5U)

為了簡化 video-to-video 合成問題，我們做了 Markov 假設，將條件分佈 \\(p \left(
\tilde { \mathbf { x } } _ { 1 } ^ { T } | \mathbf { s } _ { 1 } ^ { T }
\right)\\) 分解為以下的乘積形式：

$$
p(\tilde{\textbf x}\_1^T \mid \textbf s\_1^T) = \prod\_{t = 1}^T p(\tilde{\textbf x}\_t \mid \tilde{\textbf x}\_{t - L}^{t - 1}, \textbf s\_{t - L}^t). \tag{3}
$$

也就是說，我們假定這些影片的的每一幀可以按順序生成，而 \\(t\\)-th frame
\\(\\tilde{\\textbf x}\_t\\) 只被以下三件事所決定：

1. 目前的 source image \\(\\textbf s_t\\)
1. 過去 \\(t - L\\) 到 \\(t - 1\\) 共 \\(L\\) 個 source image \\(\\textbf
   s\_{t - L}^{t - 1}\\)
1. 過去 \\(t - L\\) 到 \\(t - 1\\) 共 \\(L\\) 個生成的圖片 \\(\\tilde{\\textbf
   x}\_{t - L}^{t - 1}\\)

這篇論文透過實驗決定了 \\(L = 2\\)。

視頻訊號在連續幀中包含大量冗餘訊息。如果從當前幀到下一幀的光流是已知的，我們可以
用它來扭曲當前幀以估計下一幀。除了遮擋區域外，這種估計在很大程度上是正確的。基於
這種觀察，我們將 \\(F\\) 模型化為

$$
\tilde{\textbf x}\_t = F(\tilde{\textbf x}\_{t - L}^{t - 1}, \tilde{\textbf s}\_{t - L}^t) = (\textbf 1 - \tilde{\textbf m}\_t) \odot \tilde{\textbf w}\_{t - 1} (\tilde{\textbf x}\_{t - 1}) + \tilde{\textbf m}\_t \odot \tilde{\textbf h}\_t, \tag{4}
$$

其中：

- \\(\\odot\\)：element-wise 相乘 operator
- \\(\\textbf 1\\)：全為 \\(1\\) 的 image
- 第一項：前一幀扭曲的像素（估計由 \\(t - 1\\)th frame 到 \\(t\\)th frame 透過
  warping 的改變）
- 第二項：模糊新的像素（會有比較大的變動）
- \\(\\tilde{\\textbf w}\_{t - 1} = W(\\tilde{\\textbf x}\_{t - L}^{t - 1},
  \\tilde{\\textbf s}\_{t - L}^t)\\)：由 \\(\\tilde{\\textbf x}\_{t - 1}\\) 到
  \\(\\tilde{\\textbf x}\_t\\) 的估計光流
  - \\(W\\)：optical flow prediction function
- \\(\\tilde{\\textbf h}\_t = H(\\tilde{\\textbf x}\_{t - L}^{t - 1}, \\textbf
  s\_{t - L}^t)\\)：模糊後的圖片（generated from scratch）。
- \\(\\tilde{\\textbf m}\_t = M(\\tilde{\\textbf x}\_{t - L}^{t - 1},
  \\tilde{\\textbf s}\_{t - L}^t)\\)：遮擋 mask，其中值為 \\(0\\) 到 \\(1\\) 之
  間
  - \\(M\\)：mask prediction function

## Conditional image discriminator

$D\_I$ output

- $1$ for a true pair $(\textbf x\_t, \textbf s\_t)$
- $0$ for a fake pair $(\tilde{\textbf x}\_t, \textbf s\_t)$

## Conditional video discriminator

令 $\textbf w\_{t - K}^{t - 2}$：$K$ 個連續圖片 $\textbf x\_{t - K}^{t - 1}$ 的
$K - 1$ 個光流

$D\_V$ output

- $1$ for a true pair $(\textbf x\_{t - K}^{t - 1}, \textbf w\_{t - K}^{t - 2})$
- $0$ for a fake pair
  $(\tilde{\textbf x}\_{t - K}^{t - 1}, \textbf w\_{t - K}^{t - 2})$

另外，我們介紹兩個 sampling operators

- $\phi\_I$（random image sampling operator）
  ：$\phi_I(\textbf x\_1^T, \textbf s\_1^T) = (\textbf x\_i, \textbf s\_i)$

  其中：$i$ 為 $1$ 到 $T$ 的整數

- $\phi\_V$（randomly retrieve $K$ consecutive frames）
  ：$\phi\_V(\textbf w\_1^{T - 1}, \textbf x\_1^T, \textbf s\_1^T) = (\textbf w\_{i - K}^{i - 2}, \textbf x\_{i - K}^{i - 1}, \textbf s\_{i - K}^{i - 1})$

  其中：$i$ 為 $K + 1$ 到 $T + 1$ 的整數

## Learning objective function

$$\min\_F (\max\_{D\_I} \mathcal L\_I(F, D\_I) + \max\_{D\_V} \mathcal L\_V(F, D\_V)) + \lambda\_W \mathcal L\_W(F), \tag{5}$$

其中：

- $\mathcal L\_I$：GAN loss for $D\_I$
- $\mathcal L\_V$：GAN loss on $K$ consecutive frames for $D\_V$
- $\mathcal L\_W(F)$：flow estimation loss
- $\lambda\_W$：實驗後，設為 $10$

GAN loss $\mathcal L\_I$：

$$
E _ { \phi _ { I } \left( \mathbf { x } _ { 1 } ^ { T } , \mathbf { s } _ { 1 } ^ { T } \right) } \left[ \log D _ { I } \left( \mathbf { x } _ { i } , \mathbf { s } _ { i } \right) \right] + E _ { \phi _ { I } \left( \tilde { \mathbf { x } } _ { 1 } ^ { T } , \mathbf { s } _ { 1 } ^ { T } \right) } \left[ \log \left( 1 - D _ { I } \left( \tilde { \mathbf { x } } _ { i } , \mathbf { s } _ { i } \right) \right) \right]. \tag{6}
$$

GAN loss $\mathcal L\_V$：

$$
E _ { \phi _ { V } \left( \mathbf { w } _ { 1 } ^ { T - 1 } , \mathbf { x } _ { 1 } ^ { T } , \mathbf { s } _ { 1 } ^ { T } \right) } \left[ \log D _ { V } \left( \mathbf { x } _ { i - K } ^ { i - 1 } , \mathbf { w } _ { i - K } ^ { i - 2 } \right) \right] + E _ { \phi _ { V } \left( \mathbf { w } _ { 1 } ^ { T - 1 } , \tilde { \mathbf { x } } _ { 1 } ^ { T } , \mathbf { s } _ { 1 } ^ { T } \right) } \left[ \log \left( 1 - D _ { V } \left( \tilde { \mathbf { x } } _ { i - K } ^ { i - 1 } , \mathbf { w } _ { i - K } ^ { i - 2 } \right) \right) \right]. \tag{7}
$$

我們透過遞迴的呼叫 $F$ 來取得 $\tilde{\textbf x}\_1^T$

The flow loss $\mathcal L\_W$ 擁有兩項，第一項是 ground truth 和估計流量之間點對
點誤差，第二項則是將前一幀扭曲到下一幀時的 warping loss：

$$
\mathcal { L } _ { W } = \frac { 1 } { T - 1 } \sum _ { t = 1 } ^ { T - 1 } \left( \left\| \tilde { \mathbf { w } } _ { t } - \mathbf { w } _ { t } \right\| _ { 1 } + \left\| \tilde { \mathbf { w } } _ { t } \left( \mathbf { x } _ { t } \right) - \mathbf { x } _ { t + 1 } \right\| _ { 1 } \right). \tag{8}
$$

## Foreground-background prior

$$
F \left( \tilde { \mathbf { x } } _ { t - L } ^ { t - 1 } , \mathbf { s } _ { t - L } ^ { t } \right) = \left( \mathbf { 1 } - \tilde { \mathbf { m } } _ { t } \right) \odot \tilde { \mathbf { w } } _ { t - 1 } \left( \tilde { \mathbf { x } } _ { t - 1 } \right) + \tilde { \mathbf { m } } _ { t } \odot \left( \left( \mathbf { 1 } - \mathbf { m } _ { B , t } \right) \odot \tilde { \mathbf { h } } _ { F , t } + \mathbf { m } _ { B , t } \odot \tilde { \mathbf { h } } _ { B , t } \right) \tag{9}
$$

# 4 Experiments

![](https://imgur.com/SHsy9Na.png)

|                           ![](https://tcwang0509.github.io/vid2vid/paper_gifs/face.gif)                            |
| :----------------------------------------------------------------------------------------------------------------: |
| Figure 5: Example sketch-to-face video results. Our method can generate realistic expressions given the edge maps. |

|                                                                             ![](https://tcwang0509.github.io/vid2vid/paper_gifs/pose.gif)                                                                             |
| :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| Figure 6: Example pose-to-dance video results. The left image pair shows the result on the same dancer (with different clothing) doing different motions, while the other two pairs are results on different dancers. |

# 5 Conclusion

## Limitaions and future work

- 因為 label maps 訊息不足，導致在合成轉彎中的車子時還是滿困難的。猜測解決方式：
  添加額外的 3D 訊息，例如：depth maps。
- 偶爾汽車的顏色會逐漸改變，這個問題可透過 object tracking 去強迫同一個物件顏色
  始終保持一致。
- 執行語義操作的時候，偶爾會出現明顯的人工痕跡，這可能是因為 label 時太過粗糙。
