---
title: Git Rebase 妙用
author: Peng-Yu Chen
date: 2022-08-17
tags:
  - Git
math: true
---

# 前言

Git 能夠有效地幫助你為一份專案寫下對應的日誌。良好、明確的提交歷史（Commit
History）能讓日後回來追蹤代碼時，事半功倍，因此筆者認為學習有效地使用 Git，尤其
是 [rebase](https://git-scm.com/docs/git-rebase) 這個功能是尤為重要的。

在終端機中輸入：

```bash
git rebase --help
```

就能看到許多 `git rebase` 的用法，本文將會專注在筆者最常使用的幾個指令中，並談談
如何高效地整理提交歷史。

# 手把手範例

- 範例會帶讀者實際的操作 `git rebase` 的用法，並且所有指令設皆能複製貼上。
- 本範例假設讀者有一定的 `git` 基礎。

## 正文開始

首先，建立我們的測試工作環境 `git-rebase-demo`，以下所有操作都會在此路徑下完成：

```bash
mkdir ~/git-rebase-demo && cd $_
git init
```

新增帶有錯別字的 `A.txt` 並提交（commit）`A.txt`：

```bash
echo 'I am fil A.' > A.txt
git add A.txt
git commit -m 'Create A.txt'
```

新增檔案 `B.txt` 並提交 `B.txt`：

```bash
echo 'I am file B.' > B.txt
git add B.txt
git commit -m 'Create B.txt'
```

查看歷史記錄：

```bash
git log --oneline
```

```
def4567 (HEAD -> main) Create B.txt
abc1234 Create A.txt
```

以下會用 `def4567` 和 `abc1234` 指代，讀者請自行帶換。

現在我們希望能修改 `A.txt` 中的錯別字，又不希望因此增加一個節點，就只是因為一時
疏忽。其中一個方法是透過 `git checkout abc1234` 修改，而筆者將介紹一種更簡單的方
式。

_注意：此方法只適用新的提交節點（`def4567`）不依賴舊的提交節點（`abc1234`）的情
況。_

首先，修正錯別字：

```bash
sed -i 's/fil/file/' A.txt     # Linux
```

```bash
sed -i '' 's/fil/file/' A.txt  # macOS
```

關鍵一步，先不管順序對錯，在此提交修改：

```bash
git add A.txt
git commit -m 'Put anything here'
```

重頭戲，進步 `git rebase` 介面：

```bash
git rebase -i --root
```

你會看到

```
pick abc1234 Create A.txt
pick def4567 Create B.txt
pick xyz0001 Put anything here
```

在 [vim](https://www.vim.org) 底下，重新調整提交順序，並將 `xyz0001` 的 `pick`
改為 `fixup`（也可縮寫成 `f`），這麼一來 `xyz0001` 就會和 `abc1234` 合併產生一個
新的節點產生新的哈希值，因為修改了舊的節點，`def4567` 的節點也會自動生成一份新的
哈希值。

```
pick abc1234 Create A.txt
fixup xyz0001 Put anything here
pick def4567 Create B.txt
```

`ZZ` 退出 vim，並再次查看提交歷史：

```bash
git log --oneline
```

```
6a674ea (HEAD -> main) Create B.txt
5a584af Create A.txt
```

可以看到我們優雅地將提交歷史變乾淨了！

# 結語

`git rebase` 的功能非常強大，已經是筆者生活與工作中不可或缺的重要工具，本文演示
了一個簡單的例子，除了 `fixup` 外，其它還有像 `reword`, `edit` 等等方便的功能，
就暫時不在這篇文章中提及了！
