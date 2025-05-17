---
title: Hexo Building
date: 2018-02-14
tags:
  - Hexo
  - Static Site Generator
---

# 前言

近日有感沒有花些心思去整理平時的一些思緒、札記等等，於是決定要開始著手寫部落格。
在網路上查了許多資料後，決定使用 [Hexo](https://hexo.io) 所提供的服務，裡面也有
許多主題可供挑選，而我使用的這個是
[NexT](https://github.com/theme-next/hexo-theme-next)，一款由中國人所設計，非常
精緻典雅的主題。

# 使用 [Hexo](https://hexo.io) 來建立個人網站

網誌中的第一篇文章，我想要談談我在使用 [Hexo](https://hexo.io) 建立個人網站時並
且發布到 [GitHub Pages](https://pages.github.com) 時所遇到的問題，以及我是怎麼解
決的。

# 環境需求

要開始使用 [Hexo](https://hexo.io)，必需先確保你的電腦有以下的插件：

1. [Git](https://zh.wikipedia.org/wiki/Git)
2. [Node.js](https://nodejs.org/en/)

# 安裝 [Hexo](https://hexo.io)

我們可以藉由 [Node.js](https://nodejs.org/en/) 輕鬆的安裝
[Hexo](https://hexo.io)：

```bash
npm install hexo-cli -g
```

# 初始化倉庫（repository）

你必需先在你的 github 帳戶底下建立一個新的倉庫，名字必需是
`<username>.github.io` 或是 `<username>.github.io/<project>`，否則你將無法成功發
佈文章。

# 初始化 Hexo 資料夾

再來，藉由以下指令初始化 Hexo 資料夾：

```bash
hexo init <project>     # create a Hexo project
cd <project>            # change directory
npm install             # update something
```

`<project>` 的名稱有兩個選擇：

1. 和 GitHub 上的名稱（`walkccc.github.io`）一樣
2. 任意名稱

# 安裝佈署器

不像其它靜態頁面生產器，[Hexo](https://hexo.io) 提供了官方的佈署器（deployer），
我們可由下列指令安裝之：

```bash
npm install hexo-deployer-git --save
```

# 安裝美麗的主題

我刪除了 [Hexo](https://hexo.io) 預設的主題 landscape，並且安裝
[NexT](https://github.com/theme-next/hexo-theme-next) 主題。

```bash
# delete the default theme
rm -rf themes/landscape

# clone the theme next
git clone https://github.com/theme-next/hexo-theme-next themes/next

# delete the .git files in the subdirectory to prevent unwanted conflicts
rm -rf themes/next/.git*
```

# 連結本機資料夾與 GitHub 上的倉庫

跟著 GitHub 上新建新倉庫的導覽（這裡會做一些修改）：

```bash
git init
git add .
git commit -m 'init'
git remote add origin https://github.com/<username>/<project>.git
git push -u master
```

# 修改設定

在資料夾的根目錄，打開 `/_config.yml` 並且修改如以下（在這裡我用我的帳號來做示範
）

```
title: Jay's Blog
author: Jay Chen
url: http://walkccc.github.io/blog
root: /blog/
theme: next
deploy:
  type: git
  repo: https://github.com/walkccc/blog.git
  branch: gh-pages
```

確保你的 branch 屬性填寫的是 `master`（非常重要！）GitHub 不允許使用分支（如
：gh-pages）發布網頁在 `<username>.github.io`（但如果你是發布在例如
：`<username>.github.io/blog` 下便可以），兄弟我就是不知道這點，所以在這耗費了許
久時間 ⋯⋯ 囧。

# 準備佈署網頁！

```bash
hexo clean      # hexo clean the folder first (highly recommended)
hexo generate   # generate the "public" files (Alias: hexo g)
hexo deploy     # deploy it!                  (Alias: hexo d)
```

後面兩行生成（generate）和佈署（deploy）的指令也可寫成一行：

```bash
hexo deploy -g
```

更多相關的指令可以

```bash
hexo --help
```

文章所使用的 .md 檔位置是在 `/source/_posts` 中，排序的方式會根據 `date` 來決定
。稍候片刻，你的網頁將會被發佈在：`https://<username>.github.io/` 或是
`https://<username>.github.io/<project>` 上！

恭喜你，但先別急，讓我們來做點優化。

# 增加數學式支援

NexT 主題很貼心，預設裡就能支援數學式了，但還是需要使用者做一些調整，在
`/themes/next/_config.yml` 中 enable 你的 math 設定為 true，並修改 CDN 位置：

```yml
math:
  enable: true
  cdn: //cdn.bootcss.com/mathjax/2.7.1/latest.js?config=TeX-AMS-MML_HTMLorMML
```

有一點要特別注意的是 `per_page` 屬性在這我是維持原本的 `true`，這樣的話我只在我
想要加入數學式的頁面才載入 javascript，例如像這頁的標題就會長的像：

```
---
title: 你好，世界！
date: 2018-02-14
mathjax: true
---
```

來試試簡單的幾行數學式：

$$\frac{-b \pm \sqrt{b^2 - 4ac}}{2a}$$

$$
y =
\begin{cases}
\min\{c, a, t\} & \mbox{ if $y < 0$} \\
\max\{d, o, g\} & \mbox{ if $y \ge 0$}
\end{cases}
$$

在這我發現了在換行時，`\\`, `{}` 等跳脫字元（escape character），會和 html 語法
相衝，所以我們要做點處理。

# 數學式渲染跳脫字元處理

引用我學長
的[文章](https://hsins.github.io/2018/01/05/Built-Personal-Website-with-Hexo/)：

Hexo 預設使用 `hexo-renderer-marked` 引擎進行渲染，但對於底線、反斜線、中括號定
義了轉義，容易與 MathJax 數學公式渲染時所處理的字符造成衝突，建議可以變更渲染引
擎為下列選項中其中一個：

- hexo-renderer-kramed
- hexo-renderer-pandoc

```bash
npm uninstall hexo-renderer-marked --save     # uninstall the hexo-renderer-marked
npm install hexo-renderer-kramed --save       # install the hexo-renderer-kramed
```

除此之外，還需要改變行內公式的轉義設定。修改
`./node_modules/kramed/lib/rules/inline.js`：

```
// escape part
- escape: /^\\([\\`*{}\[\]()#$+\-.!_>])/,
+ escape: /^\\([`*\[\]()#$+\-.!_>])/,

// em part
- em: /^\b_((?:__|[\s\S])+?)_\b|^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
+ em: /^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,`
```

記得要

```bash
hexo clean
```

否則數學式還是會無法渲染。

# 備份本機上的資料夾

在我功佈署我們的網頁後，GitHub 上有的只有網頁所需要的檔案，但如果哪天換了電腦，
想要在別台電腦上修改，事情就會變得非常麻煩。

我們可以運用 GitHub 的分支（branch）功能來備份根目錄資料夾。

1. 可以在 GitHub 上專頁上新增一個叫 sources 的分支 
   ![](https://i.imgur.com/16sXJlB.png)
2. 或是在根目錄透過 command line 指令：

```bash
git checkout -b sources   # create the branch on your local machine and switch in this branch
git push origin sources   # push the new branch: sources on github
git add .                 # add all files in root folder
git commit -m 'first backup sources'
git push --set-upstream origin sources
```

大功告成，開始寫部落格吧！
