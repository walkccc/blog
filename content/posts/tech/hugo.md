---
title: 用 Hugo 搭建，並用 GitHub Actions 佈署你的個人博客！
author: Peng-Yu Chen
date: 2022-08-16
tags:
  - Hugo
---

# 遇見 Hugo

今天來談談 [Hugo](https://gohugo.io) 環境設定時所遇到的一些問題及解決辦法。

# 預備環境

要開始使用 [Hugo](https://gohugo.io)，若您的環境跟我一樣是 macOS 的話，可以使用
套件管理工具 [Homebrew](https://brew.sh/index_zh-tw.html)：在終端機輸入以下指令
後，便會開始下載需要的套件。

```bash
brew install hugo
```

# 設定本地端

## 初始化並創建 Git 倉庫

```bash
REPO_NAME=blog
hugo new site $REPO_NAME -f yaml
cd $REPO_NAME
git init
git add .
git commit -m "$ hugo new site blog -f yaml"
```

## 使用 Git submodules 新增主題

```bash
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
git submodule update --init --recursive
git add .
git commit -m "Add PaperMod submodule"
```

## 修改預設 config.yaml 檔

複製貼上
[`config.yaml`](https://github.com/walkccc/blog/blob/main/config.yaml)。

```bash
git add .
git commit -m "Update default config.yaml"
```

# 新增彙總（Archives）

```bash
echo "---
title: 彙總
layout: archives
url: /archives/
summary: archives
---" > content/archives.md
```

# 設定 GitHub Actions

複製貼上
[`.github/workflows/main.yaml`](https://github.com/walkccc/blog/blob/main/.github/workflows/main.yaml)

```bash
git add .
git commit -m "Add GitHub Actions"
```

# 設定 GitHub

1. 新建一個 repository 名為 `blog`。

1. 推送代碼到 GitHub 上。

   ```bash
   git remote add origin https://github.com/<GITHUB_USERNAME>/blog.git
   git push -u origin main
   ```

這時稍等片刻，就能在你的域名看到你的網站了！
