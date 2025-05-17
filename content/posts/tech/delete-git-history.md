---
title: 刪除 Git 提交記錄
author: Peng-Yu Chen
date: 2018-02-16
tags:
  - Git
---

# 前言

在使用 Git 時，我們可能會有些「不那麼整齊」的提交記錄（commit history），這時創
建一個新的倉庫（repository），並且將原有資料整份移植過去是個有效的解決方式，但似
乎不太聰明，於是我開始尋找是否有更有效率的方法，好在
[Stack Overflow](https://stackoverflow.com/)
上[這篇文章](https://stackoverflow.com/questions/13716658/how-to-delete-all-commit-history-in-github)給
出了一個可用的解答。

# 刪除提交記錄

若直接刪除 `.git` 資料夾的話，可能會導致你 git 倉庫出現問題。如果想要刪除所有的
提交記錄，且保留目前程式碼的狀態，以下的步驟是安全且可行的：

```bash
# create a temporary branch
git checkout --orphan TEMPORARY_BRANCH

# add all files
git add -A

# commit the changes
git commit -am "Reset Commit History"

# delete the "main" branch
git branch -D main

# rename the current branch to "main"
git branch -m main

# forcely update your "main" branch
git push -f origin main
```

如果想要刪除的是分支，那麼將上述指令中的 `main` 換成預刪除分支的名字即可。
