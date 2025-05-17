---
title: Anaconda 完美解決 Python 2 和 Python 3 並存問題
author: Peng-Yu Chen
date: 2019-04-01
tags:
  - Anaconda
  - Python
toc: true
---

# 前言

[Python 3](https://www.python.org/download/releases/3.0/) 已經逐漸取代 Python
2，成為普遍開發者所使用的語言，但最囧的是，很多老舊的專案、系統仍然運行著由
Python 2 所編成的環境中，有時為了在舊版本（Python 2）中開發，必需要有一個很好的
環境控制方式。

## Anaconda

引用維基百科上有關
[Anaconda](<https://en.wikipedia.org/wiki/Anaconda_(Python_distribution)>) 的敘
述：

> Anaconda 是一種 Python 語言的免費增值開源發行版，用於進行大規模數據處理，預測
> 分析，和科學計算，致力於簡化包的管理和部署。Anaconda 使用軟體包管理系統 Conda
> 進行包管理。

使用 Anaconda 可以幫我解決以下兩個大問題：

- 提供 package management：功能類似 pip
- 提供 virtual environment：解決了 Python 多版本並存問題

你可以在[這裡](https://www.anaconda.com/download/#macos)下載 macOS 的 Anaconda
最新版本，其它作業系統也能在分頁當中找到。

## 更新套件

```bash
conda update --all
```

## 建立環境

```bash
# Create the environment
conda create --name python3 python=3.7
conda create --name python2 python=2.7

# Activate the environment
source activate python3     # Linux/macOS
activate python3            # Windows
```

更多的指令，可以查看

```bash
conda -h
```

## 管理 packages

conda 的 package management 是對 pip 功能的擴充，如果已經啟動了某個 Python 環境
，便可以在該環境開始安裝第三方的 package，例如：

```bash
conda install numpy  # Install numpy package
conda list           # List all installed packages
conda update numpy   # Update numpy package
conda remove numpy   # Remove numpy package
```

## 設定預設版本

macOS High Sierra 原廠就預設自帶 Python 2 版本，位置在
`/System/Library/Frameworks/Python.framework/Versions/2.7`，我希望預設版本是
Python 3

```bash
export PATH="/usr/local/bin:$PATH"            # Default path setting
export PATH="/Users/Jay/anaconda3/bin:$PATH"  # Anaconda 3
```

若將 Anaconda 的路徑放在下面，代表 Anaconda 會覆蓋掉預設路徑 Python 2 的環境，也
就是若你灌的版本（例如：3.7.3)，當你在 Terminal 執行：

```bash
$ python
```

就會直接進到 Anaconda 底下的 Python 3.7.3 的環境了！其它的設定：如 `pip` 等也都
是。
