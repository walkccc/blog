---
title: Digit DP
author: Peng-Yu Chen
date: 2022-08-27
tags:
  - Algorithms
math: true
---

第一篇記錄算法的文章，來介紹下新學到的算法：Digit DP。

## 暴力解

題目模版一般會長的像這樣：給定區間 $[L, R]$ 和一個函數
$f(x) \to \\{\texttt{True}, \texttt{False}\\}$，問在 $[L, R]$ 之間的正整數，滿足
$f(x)$ 的數量有多少？

現在我們可以帶入一個真實的例子，在 $[L = 19, R = 9999]$ 區間，每一位數的總合為
$k = 23$ 的數字共有幾個？

暴力解用 C++ 代碼演示如下：

```cpp
bool DigitSum(int num) {
  int sum = 0;
  while (num > 0) {
    sum += num % 10;
    num /= 10;
  }
  return sum;
}

int GetSatisfiedCount(int l, int r, int k) {
  int satisfied_count = 0;
  for (int num = l; num <= r; ++num)
    if (DigitSum(num) == k)
      ++satisfied_count;
  return satisfied_count;
}

constexpr int start = 19;
constexpr int end = 9'999;
constexpr int k = 23;
std::cout << GetSatisfiedCount(start, end, k);
```

而這樣的時間複雜度為：

$$
O(r - l) \cdot O(|\texttt{f}|) = O(r - l) \cdot \log(\texttt{num}),
$$

若區間大小非常大，例如 $[0, 10^9]$，暴力解不太現實。

## 重新思考

那麼，該如何優化呢？首先定義：

> $S(R, k)$：在 $[0, R]$ 滿足 $f(x) = k$ 的數量，其中 $f(x)$ 為 $x$ 的數字總合。

可以推出，$S(R) - S(L - 1)$ 為在 $[L, R]$ 滿足 $f(x) \to \texttt{True}$ 的數量。
並且觀察因為 $R = 9999$，任何滿足條件的值最多只會有 $n = 4$ 位數。現在我們試著將
題目變成子問題：若在第一位填上 `i _ _ _`，會發現在剩下的 $n - 1 = 3$ 位數中，剩
下 $k - i$ 的餘額。

現在我們定義 $dp(n, k)$：在 $n$ 位數當中，滿足 $f$ 的數量，可得：

$$
dp(n, k) = \sum_{i = 0}^9 dp(n - 1, k - i).
$$

並且定義終止條件：$dp(0, 0) = 1$。

但這出現了一個問題，若 $R$ 不等於 $10^n - 1$ 的值，那麼不能考慮所有 $n$ 為數的數
字，舉例來說，若 $R = 3456$，則我們不能考慮所有 4 位數的字，更具體的說，任何在
$[3457, 9999]$ 區間滿足條件的數字都不應被計算進去，必需加入適當的條件判斷。

我們可以加入一個 `isTight` 的布林值，判斷當前這一位數應不應該遵守開區間的邊界規
範，重新定義 $dp(n, k, \texttt{isTight})$，對於題目 $S(R, k) = S(3456, k)$，我們
可以針對在首位數填上不同數字分成以下三種情形討論：

- 填上 $< 3$ 的數字，那麼剩下的 3 位數不再需要遵守邊界規範，可以自由的填上
  $[0, 999]$ 而不會超過 $3456$。
- 填上 $3$，那麼剩下的三位數還是必需遵守邊界規範 $[0, 456]$。
- 顯然地，我們不能在第一位數填上 $> 3$ 的任何數字。

$$
\begin{aligned}
S(R, k)
  &= S(3456, k) \\\\
  &= dp(4, k, \texttt{isTight = True}) \\\\
  &= \sum_{i = 0}^2 dp(4 - 1, k - i, \texttt{isTight = False}) \\\\
  &+ dp(4 - 1, k - 3, \texttt{isTight = True}).
\end{aligned}
$$
