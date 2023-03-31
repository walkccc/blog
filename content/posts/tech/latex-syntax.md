---
title: LaTeX 語法整理
author: Peng-Yu Chen
date: 2018-02-17
tags:
  - LaTeX
math: true
---

# 前言

使用 [LaTeX](https://www.latex-project.org) 這款排版工具也有 1 年多了，有鑑於在
查語法時常重複查詢造成效率下降，加上朋友的建議，於是就來整理份常用的 LaTeX 語法
吧！

# 基本

|       程式碼        |         LaTeX         |       程式碼        |         LaTeX         |
| :-----------------: | :-------------------: | :-----------------: | :-------------------: |
|         a_1         |         $a_1$         |      x^{2018}       |      $x^{2018}$       |
|    e^{-\alpha t}    |    $e^{-\alpha t}$    |      a^3\_{ij}      |      $a^3_{ij}$       |
| e^{x^2} \ne {e^x}^2 | $e^{x^2} \ne {e^x}^2$ | 94 \times 87 = 8178 | $94 \times 87 = 8178$ |

# 根號

|   程式碼    |     LaTeX     |        程式碼        |         LaTeX          |
| :---------: | :-----------: | :------------------: | :--------------------: |
|   \sqrt x   |   $\sqrt x$   | \sqrt{x^2 + \sqrt y} | $\sqrt{x^2 + \sqrt y}$ |
| \sqrt[3]{2} | $\sqrt[3]{2}$ |   \surd[x^2 + y^2]   |   $\surd[x^2 + y^2]$   |

# 線、向量

|      程式碼      |       LaTeX        |       程式碼        |         LaTeX         |
| :--------------: | :----------------: | :-----------------: | :-------------------: |
| \overline{m + n} | $\overline{m + n}$ |  \underline{m + n}  |  $\underline{m + n}$  |
|      \vec a      |      $\vec a$      | \overrightarrow{AB} | $\overrightarrow{AB}$ |

# 其它

|                程式碼                 |                 LaTeX                  |                  程式碼                   |                    LaTeX                    |
| :-----------------------------------: | :------------------------------------: | :---------------------------------------: | :-----------------------------------------: |
| \underbrace{a + b + \cdots + z}\_{26} | $\underbrace{a + b + \cdots + z}_{26}$ | \int_0^{\frac{\pi}{2}} \cos\theta d\theta | $\int_0^{\frac{\pi}{2}} \cos\theta d\theta$ |
|   \frac{x^2}{1 + x + \cdots + x^n}    |   $\frac{x^2}{1 + x + \cdots + x^n}$   |            x^{\frac{2}{k + 1}}            |            $x^{\frac{2}{k + 1}}$            |
|           \sum\_{i = 1}^{n}           |           $\sum_{i = 1}^{n}$           |              \prod\_\epsilon              |              $\prod_\epsilon$               |

# 數學符號

## 數學模式上標

|   程式碼   |    LaTeX     |    程式碼    |     LaTeX      |
| :--------: | :----------: | :----------: | :------------: |
|   \hat a   |   $\hat a$   |   \tilde a   |   $\tilde a$   |
| \widehat a | $\widehat a$ | \widetilde a | $\widetilde a$ |
|  \acute a  |  $\acute a$  |   \grave a   |   $\grave a$   |
|   \dot a   |   $\dot a$   |   \ddot a    |   $\ddot a$    |
|  \check a  |  $\check a$  |   \breve a   |   $\breve a$   |
|   \bar a   |   $\bar a$   |    \vec a    |    $\vec a$    |

## 小寫希臘字母

|   程式碼    |     LaTeX     |  程式碼   |    LaTeX    |
| :---------: | :-----------: | :-------: | :---------: |
|   \alpha    |   $\alpha$    |    \xi    |    $\xi$    |
|    \beta    |    $\beta$    |     o     |     $o$     |
|   \gamma    |   $\gamma$    |    \pi    |    $\pi$    |
|   \delta    |   $\delta$    |  \varpi   |  $\varpi$   |
|  \epsilon   |  $\epsilon$   |   \rho    |   $\rho$    |
| \varepsilon | $\varepsilon$ |  \varrho  |  $\varrho$  |
|    \zeta    |    $\zeta$    |  \sigma   |  $\sigma$   |
|    \eta     |    $\eta$     | \varsigma | $\varsigma$ |
|   \theta    |   $\theta$    |   \tau    |   $\tau$    |
|  \vartheta  |  $\vartheta$  | \upsilon  | $\upsilon$  |
|    \iota    |    $\iota$    |   \phi    |   $\phi$    |
|   \kappa    |   $\kappa$    |  \varphi  |  $\varphi$  |
|   \lambda   |   $\lambda$   |   \chi    |   $\chi$    |
|     \mu     |     $\mu$     |   \psi    |   $\psi$    |
|     \nu     |     $\nu$     |  \omega   |  $\omega$   |

## 大寫希臘字母

| 程式碼  |   LaTeX   |  程式碼  |   LaTeX    |
| :-----: | :-------: | :------: | :--------: |
| \Gamma  | $\Gamma$  |  \Sigma  |  $\Sigma$  |
| \Delta  | $\Delta$  | \Upsilon | $\Upsilon$ |
| \Theta  | $\Theta$  |   \Phi   |   $\Phi$   |
| \Lambda | $\Lambda$ |   \Psi   |   $\Psi$   |
|   \Xi   |   $\Xi$   |  \Omega  |  $\Omega$  |
|   \Pi   |   $\Pi$   |
