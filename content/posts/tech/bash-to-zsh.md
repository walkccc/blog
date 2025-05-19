---
title: bash -> zsh
author: Peng-Yu Chen
date: 2018-06-30
tags:
  - zsh
---

# 前言

因為朋友推薦，前陣子開始轉移使用
[oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)，起初覺得用了很久的 bash
已經夠好用了，真的有需要轉移到 zsh 上嗎？但經過使用一個月的心得，我能告訴您
，zsh > bash。

本身是使用 macOS，不確定使用 Windows 的朋友是否能夠如法砲製，zsh 相比 bash 有許
多的優點，像是：

- 不區分大小寫（方便 tab 補齊路徑名）
- 能夠上色，有多種主題
- 能夠顯示 git branch 分支狀態

接下來就來說 zsh 的安裝方法：

1. 安裝 `zsh`
1. 安裝 `oh-my-zsh`
1. 安裝 `zsh-completions`
1. 修改設定

# 安裝流程

## 安裝 zsh

```bash
brew install zsh
```

- 並且將預設的 bash 修改為 zsh：

  ```bash
  chsh -s /usr/local/bin/zsh    # change shell to zsh
  ```

- 若日後要換回原廠預設的 bash：

  ```bash
  chsh -s /bin/bash             # change shell to default bash
  ```

## 安裝 oh-my-zsh

```bash
git clone https://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
```

預設路徑是在 `~/.oh-my-zsh`，若想裝在別處要修改 `~/.zshrc`。

zsh 的設定檔放在 `~/.zshrc` 就像 bash 的設定檔放在 `~/.bashrc` 一樣，而這個檔案
需要我們自己產生，這裡建議使用 oh-my-zsh 的模版比較簡單：

```bash
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
```

## 安裝 zsh-completions

```bash
brew install zsh-completions
```

再這裡還要記得打開 `~/.zshrc` 來加入 zsh-completions 的補全功能：

```bash
code ~/.zshrc   # open ~/.zshrc by VSC
```

加入一行 zsh-completions 對應的路徑：

```bash
# zsh-completions
fpath=(/usr/local/share/zsh-completions $fpath)
```

最後 rebuild zsh 的 `.zcompdump`

```bash
rm -f ~/.zcompdump; compinit
```

## 修改設定

基本上到這步，zsh 該有的功能都已經有了，但我們仍然可以做一些設定讓它更好用！

### 切換主題

主題們在 `~/.oh-my-zsh/themes` 中，切換主題的方式是修改 `~/.zshrc` 的
`ZSH_THEME`，預設是 `robbyrussell`，像我就改成 `agnoster`

```bash
# THEME Config
-ZSH_THEME="robbyrussell"
+ZSH_THEME="agnoster"
```

重新登入看看，zsh 的主題應該已被更換！

agnoster 這個主題需要將 Terminal 的字體換成
[Monoco for Powerline](https://gist.github.com/baopham/1838072)，原廠是沒有的，
他可以顯示 git 狀態及有美麗的顏色主題。

### 啟用插件

插件們在 `~/.oh-my-zsh/plugins` 中，啟用方式和主題類似，修改 `~/.zshrc` 的
`plugins`，例如要啟用 brew，只要修改預設只有 git 成：

```bash
-plugins=(git)
+plugins=(git brew)
```

### 其它

zsh 一樣具備 alias 的功能，只要在 `~/.zshrc` 中加入：

```bash
alias xx="path_name"
```

像我常進一個很長的資料夾，就會加入

```bash
alias cdtb="cd ~/Library/Mobile\ Documents/3L68KQB4HG~com~readdle~CommonDocuments/Documents/textbook"
```

以上就是 zsh 大致上的介紹了！
