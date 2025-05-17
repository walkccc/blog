---
title: 論文筆記 Prioritized Experience Reply
author: Peng-Yu Chen
date: 2018-12-13
tags:
  - ICLR
  - paper
math: true
---

[Paper Link](https://arxiv.org/pdf/1511.05952.pdf)

Experience replay 使得 RL agents 能夠記住並重用過去的經驗。

以前的做法是，從 replay memory 中均勻的 sample experience transition。但這種方法
只根據他們最初被體驗到的頻率，而不管它們的重要性。

在本文中，開發了一個優先驗體的框架，以便更頻繁地重放重要的 transition，從而更有
效地學習。

本文表示，採用此種經驗回放的算法，能夠在 49 種遊戲中，打贏傳統算法 41 次。

# 1 Introduction

RL agents 在觀察到一連串的經驗時，增量地更新其參數。最簡單的做法，即：在更新後，
立刻扔掉到來的數據，但這有幾下兩種問題：

1. 強烈的相關性更性破壞了許多隨機梯度下降為基礎的演算法所假設的 i.i.d.。
1. 快速的忘記罕見，但可能會再用到的經驗。

Experience replay 解決了上述問題：在一個 replay memory 中儲存 experience，通過混
合或多或少的最近經驗來更新就有可能破壞 temporal correlation，rare experience 將
會被用來不止一次更新，這就被用在 NeurIPS 2013 和 Nature 2015 的論文中。

DQN 利用一個大的滑動窗口 replay memory，隨機的從中均勻採樣，平均重複訪問一個
transition 8 次。

Experience replay 可以大量的降低需要去學習的經驗，而是用更多的計算和更多的
memory 來替換，這樣在計算資源上是比該 RL agents 和環境互動還便宜的。

註：在這裡的一次 transition 指的是：agent 在環境中一次性的操作，即 tuple：

$$
(\text{state} S_{t - 1}, \text{action} A_{t - 1}, \text{reward} R_t, \text{discount} \gamma_t, \text{next state} S_t).
$$

## 2 Background

## 3 Prioritized Replay

設計一個 prioritized replay memory，涉及到兩個層次，即：

1. 選擇哪些儲存
1. 選擇哪些回放

本文主要研究後者，如何最有效的利用 replay memory 來學習。

## 3.1. A Motivating Example

本文給出了一個例子來充分的說明優先的潛在好處。引入了稱為 'Blind Cliffwalk' 的環
境，來說明：當 reward 非常罕見時，探索所遇到的挑戰。假設僅有 $n$ 個狀態，這個環
境就要求足夠的隨機步驟知道得到第一個非零 reward；確切的說，隨機的選擇動作情況下
reward = $2^{-n}$。此外，最相關的 transitions 卻藏在大量的失敗與嘗試中。

本文利用這個例子來 highlight 兩個 agents 的學習次數的不同。可以看到這兩個 agent
都從同一個 replay memory 中去獲取 Q-learning 的更新。

第一個 agent 隨機均勻的回放 transitions，第二個喚醒一個 oracle（神喻）來進行優先
轉移。這個 oracle 貪婪的選擇使得在當前狀態下最大化的降低全局損失的 transitions。

從 Figure 1. 右側的圖可以看出，按照一定優化序列得到的轉移比隨機均勻採樣，少花費
很多嘗試，這明顯的提升了訓練的速度。

|                                                                                                                                                                                                                                                                                                                                                                                                                        ![](https://imgur.com/bUZJqVG.png)                                                                                                                                                                                                                                                                                                                                                                                                                        |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| Figure 1: **Left**: Illustration of the 'Blind Cliffwalk' example domain: there are two actions, a 'right' and a 'wrong' one, and the episode is terminated whenever the agent takes the 'wrong' action (dashed red arrows). Taking the 'right' action progresses through a sequence of n states (black arrows), at the end of which lies a final reward of 1 (green arrow); reward is 0 elsewhere. We chose a representation such that generalizing over what action is 'right' is not possible. **Right**: Median number of learning steps required to learn the value function as a function of the size of the total number of transitions in the replay memory. Note the log-log scale, which highlights the exponential speed-up from replaying with an oracle (bright blue), compared to uniform replay (black); faint lines are min/max values from 10 independent runs. |

## 3.2 Prioritizing with TD-Error

Prioritized replay 評判優先的準則是：衡量每一 transitions 的重要性。一個理想的標
準是當前狀態下，RL agent 能夠學習到的量，也就是期望的學習過程。但是這個標準並不
能直接訪問到，一個比較合理的且能夠表示重要性的另一個衡量方示是：一個 transition
的 TD error $\delta$ 的規模，來表示該 transition 出乎意料的程度：這非常適合增量
的在線 RL 算法，比如：SARSA 或者 Q-learning，已經計算 TD-error 並且 更新和
$\delta$ 成比例的參數。但是有些情況下，TD-error 仍然是非常差的預測，例如：當
reward 充滿噪音時。

為了說明通過 TD-error prioritized replay 的有效性，我們對比了 uniform 和 oracle
baselines 在 Blind Cliffwalk 上的 'greedy TD-error prioritization' 算法。該算法
存儲了在 replay memory 中每次 transition 後，最後遇到的 TD error。將最大絕對值
TD-error 的 transition 從 memory 中 replay。 transition 的 Q-learning 更新和
TD-error 相比的 權重。新的 transition 到來之後，沒有已知的 TD-error，所以我們將
其放到最大優先級的行列，確保所有的 experience 至少 replay 一次。

關於這樣做的好處，從 Figure 2. 可以看出，oracle 的做法可以極大的降低無用的嘗試，
加速了算法的執行速度。

|                                                                                                                                                                                                                                                                                                                                                                                                                                  ![](https://imgur.com/Oe1fdXA.png)                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| Figure 2: Median number of updates required for Q-learning to learn the value function on the Blind Cliffwalk example, as a function of the total number of transitions (only a single one of which was successful and saw the non-zero reward). Faint lines are min/max values from 10 random initializations. Black is uniform random replay, cyan uses the hindsight-oracle to select transitions, red and blue use prioritized replay (rank-based and proportional respectively). The results differ by multiple orders of magnitude, thus the need for a log-log plot. In both subplots it is evident that replaying experience in the right order makes an enormous difference to the number of updates required. See Appendix B.1 for details. **Left**: Tabular representation, greedy prioritization. **Right**: Linear function approximation, both variants of stochastic prioritization. |

## 3.2. Stochastic Priorization

然而，greedy TD-error prioritization 有幾個問題：

1. 為了避免在整個 replay memory 中掃瞄而帶來的計算代價，TD-error 僅僅更新被
   replay 的 transition。這個帶來的一個後果就是：帶有低 TD-error 的 transition
   在第一次訪問時可能很長時間不會被 replay。
1. 對 noise spikes 非常敏感，bootstrapping 會加劇該現象，估計誤差又會成為另一個
   noise 的來源。
1. 貪婪優先集中於一個小的經驗子集，誤差收縮的很慢，特別是使用函數估計的時候，意
   味著初始高誤差的 transition 被經常 replay，缺乏多樣性使得該系統傾向於
   over-fitting。

為瞭解決上述問題，我們引入了一個隨機採樣的方法，該方法結合了

1. 純粹的貪婪優先
1. 均勻隨機採樣

我們確保被採樣的機率在 transition 優先級上是單調的，與此同時，確保最低優先級的
transition 的機率也是非零。我們定義採樣 transition $i$ 的機率為：

$$
P(i) = \frac{p_i^\alpha}{\Sigma_k p_k^\alpha} \tag{1}
$$

其中，

- $p_i$ 是 transition $i$ 的優先級。指數 $\alpha$ 決定了使用多少的優先級，當
  $\alpha = 0$ 時是均勻的情況。

第一種變形是直接的，等比例的考慮優先級別
，$p_i = |\delta_i| + \epsilon$，$\epsilon$ 是為了避免在 edge-case 時，如果
TD-error 為零就不會再被訪問。

第二種變形是非直接的，基於排行的優先級，$p_i = \frac{1}{\text{rank}(i)}$，其中
$\text{rank}(i)$ 是 replay memory 根據 TD-error 所排行的。

兩個變形都是誤差單調，但是後者更穩健，因為其對離群點不敏感。兩個變形相對均勻的
baseline 都有很大優勢，如 Figure 2. 右側所示。

## 3.4 Annealing the Bias

用隨機更新得來的期望值預測，依賴於這些更新。Prioritized replay 了誤差，因為它以
一種不受控的形式改變了分佈，從而改變了預測會收斂到的 solution（即使 policy 和狀
態分佈都固定）。我們可以用下面的重要性採樣權重來修正該誤差：

$$
w_i = \Bigg(\frac 1 N \cdot \frac{1}{P(i)} \Bigg)^\beta
$$
