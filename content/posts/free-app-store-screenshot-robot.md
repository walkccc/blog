+++
date = 2026-07-21T00:00:00-04:00
title = "The Free Screenshot Robot: App Store Assets in Ten Languages, Zero Clicks"
tags = ["iOS", "Automation", "AI", "Indie"]
categories = ["Engineering"]
+++

*[English](#en) · [中文](#zh)*

<a id="en"></a>

## English

App Store screenshots are the chore nobody warns you about. A finished iOS app
needs a set per language, a set per device family, and a fresh set every time
you nudge a color. For [Zestimer](https://zestimer.com) that is ten languages ×
about ten scenes × iPhone-plus-Watch — call it a hundred and fifty images —
and the moment I touched the design system, all of them were wrong.

So I did what any tired solo developer does: I built a robot. The whole pipeline
now runs from one command, ships to App Store Connect in ten languages, and
costs nothing to reproduce. This post is the blueprint — every stage, the one
genuinely clever trick, and the free parts list, so you can lift it wholesale.

### The Shape of It

A screenshot pipeline is four jobs pretending to be one:

1. **Capture** — get clean pixels of each screen, in each language.
2. **Render** — drop those pixels into framed, captioned marketing cards.
3. **Stabilize** — know, reliably, whether anything actually changed.
4. **Upload** — push the final set to App Store Connect.

Most people wire these together with UI-test recordings and a pile of manual
Figma work. I wanted none of that. Every stage below is a plain script, and one
Claude Code _skill_ drives all four so I never have to remember the order.

### Stage 1: Capture Inside the App, Not the UI

The standard advice is to drive `XCUITest` — launch the app, tap through to each
screen, snapshot. I find UI tests for screenshots miserable: they're slow,
flaky, and they break every time a button moves.

The trick that made this pleasant is to make the screenshots a _feature of the
app_, reachable by a launch argument. Zestimer ships a `-ScreenshotMode` that
swaps in an in-memory SwiftData store, seeds deterministic demo data, and reads
a `-ScreenshotScene` argument to jump straight to one surface. No tapping, no
navigation, no waiting for animations. Each screen is one launch:

```sh
xcrun simctl launch --terminate-running-process "$UDID" com.your.app \
  -ScreenshotMode -ScreenshotScene activeWorkout -ScreenshotLanguage japanese
xcrun simctl io "$UDID" screenshot --type=png out/ja/active.png
```

Loop that over `{scene} × {language}` and the whole set falls out in a couple of
minutes. A full pass for me is a hundred launches, and because the data is
seeded (not live), the Japanese card and the German card show the _same_
workout — which is exactly what you want when the store reviewer compares
locales. The one rule that saved me: an unknown `-ScreenshotLanguage` should **trap** —
not silently fall back to English, or you'll ship an English card in a Korean
listing and never notice.

Everything here is free and Apple-native: `simctl` for boot, appearance,
status-bar override (pin it to 9:41), and capture. No third-party runner.

### Stage 2: Render the Cards — Screenshots Are Ads

Raw simulator captures are documentation. The store wants _advertisements_: a
device frame, a headline that sells one idea, a background that's on-brand. That
framing is its own tool, and it's the one stage I used to pay for.

Here's the important update. There's now a free Claude Code skill,
`app-store-screenshots`, that scaffolds a complete Next.js editor into your
project: live preview at true resolution, drag-to-reorder, inline caption
editing, per-locale text, light/dark variants, cross-screen "panoramic"
composition, and one-click bulk export at every Apple and Google resolution
via `html-to-image`. It writes an `app-store-screenshots.json` you can commit, so
your card layouts live in git alongside the app.

That means the expensive-looking part of this pipeline — the framed marketing
cards — is now a `npm run` away, for free, running entirely on your machine. My
own renderer feeds captions per language and exports one folder per locale; the
skill's template does the same job out of the box.

### Stage 3: Make Git the Source of Truth

This is the trick I'm proudest of, and it's five lines of Python.

The problem: the rendered cards are committed to git, but a headless Chromium
render isn't byte-stable. Glass materials and anti-aliasing dither by a pixel or
two between runs. If you commit every render, `git status` is always dirty and
you can never answer the only question that matters: _did a screenshot actually
change?_

The fix is a diff gate. After each render, compare the new PNG against the
committed one. If they differ only within rendering noise, **keep the old
bytes**:

```python
old = numpy.asarray(Image.open(previous).convert("RGB"), dtype=numpy.int16)
new = numpy.asarray(Image.open(fresh).convert("RGB"), dtype=numpy.int16)
# Dither lands in the low tens per pixel; a real change (a redrawn digit,
# moved text) lands in the hundreds. 32 splits them cleanly.
changed = int((numpy.abs(old - new).sum(axis=2) > 32).sum())
if changed < 48:
    shutil.copy(previous, fresh)   # unchanged: restore committed bytes
```

Now a clean `git status` on the screenshots folder _is_ the answer: nothing
changed, skip the upload. A dirty status is a real diff you can eyeball before
it ships. The whole "did I need to recapture?" decision collapses into `git
diff` — no manifest, no hashes, no state file. The version control you already
have becomes the change-detection system you didn't want to build.

### Stage 4: Upload With Fastlane

The last mile is [fastlane](https://fastlane.tools) `deliver`, which is free and
open source. It uploads a folder of screenshots per locale to App Store Connect
over the API. Two details worth stealing:

- **deliver assigns each image to a device family by its pixel dimensions.** So
  the 416-px-wide Apple Watch captures and the 1242-px iPhone cards can live in
  the _same_ locale folder — deliver sorts them onto the right device
  automatically. Numeric filename prefixes (`1-list`, `2-active`) set the
  on-store order.
- **One capture set can feed two listings.** Spanish ships as `es-ES`
  and `es-MX` on the store, but the screenshots are identical — so I render `es-ES`
  once and mirror the folder to `es-MX` before upload.

Split the pipeline into three scripts — `generate` (capture + render + stage),
`publish-screenshots`, `publish-metadata` — so the slow, serialized upload is
its own step you only run when Stage 3 says something changed.

### The Skill That Runs the Whole Thing

Four scripts is still four things to remember, in order, with the right
arguments. The layer that makes it feel like a robot is a Claude Code **skill**.

Two primitives do the work, and they're worth distinguishing:

- An **`AGENTS.md`** (or `CLAUDE.md`) is the _harness_ — the standing rules your
  AI pair follows on every task: architecture, design tokens, tone, the ban on
  compiling the app unless it's a release. It's the constitution.
- A **skill** is a _packaged procedure_ — a Markdown file with front matter that
  the agent invokes on demand. It's a runbook you write once and trigger with a
  slash command.

My `/release` skill is the runbook for shipping. Its front matter tells the
agent when to fire; its body is the checklist:

```markdown
---
name: release
description: Ship the App Store release end to end — draft What's New in every
  locale from the git log, bump the version, sync the site, capture and upload
  screenshots when a screen changed, push metadata. Use on "/release".
---

1. Diff `last-release..HEAD`, keep only user-visible changes.
2. Draft What's New in every locale (a real rewrite, never a literal
   translation).
3. Bump the version.
4. If a rendered screen changed, run generate → publish-screenshots.
5. Push metadata. Report which path you took — never claim an upload
   succeeded unless the command did.
```

When I type `/release 1.4`, the agent reads the git log since the last release,
writes the release notes in eleven locales, bumps the version, decides from the
diff whether any _rendered_ screen changed, and — only if it did — runs the
capture-render-upload chain in the background. It hands me back exactly two
things Apple gates behind a signed build: the Xcode archive and the final
Submit. The judgment ("does this diff touch a screen?", "is this note honest?")
lives in the skill's prose; the mechanics live in the scripts it calls.

That's the real unlock. The scripts made the work _possible_; the skill
made it _delegable_.

### Copy This, For Free

Here's the full parts list. Every line is free or open source:

| Stage | Tool | Cost |
| --- | --- | --- |
| Capture | `simctl` + an in-app `-ScreenshotMode` | free (Apple) |
| Render | `app-store-screenshots` Claude skill | free, local |
| Stabilize | ~10 lines of Python + NumPy/Pillow | free |
| Upload | fastlane `deliver` | free (open source) |
| Orchestrate | a Claude Code skill + `AGENTS.md` | free |

The only piece that ever cost money — the marketing-card renderer — is now the
free skill. So the honest summary is: **the entire pipeline is reproducible for
zero dollars.** Add `-ScreenshotMode` to your app, scaffold the renderer with
the skill, drop in the diff gate, point fastlane at the folders, and write
a `/release` runbook that ties them together.

### The Takeaway

The screenshots were never the hard part. The hard part was the _loop_ — that
every design change silently invalidated a hundred images, and the cost of
finding out was opening App Store Connect and squinting. Automating the capture
saved minutes. Making git the source of truth saved the loop. And writing the
whole thing down as a skill meant the robot could run it, not just me.

The best tools a solo developer builds aren't features. They're the machines
that let one person keep shipping like a team — and, increasingly, they're free.

---

<a id="zh"></a>

## 中文

App Store 截圖是沒人事先警告你的雜事。一個做完的 iOS App，每種語言要一組、每個裝置
家族要一組，而且每次只是微調一個顏色，就得重做一次。以 [Zestimer](https://zestimer.com)
來說，那是十種語言 × 大約十個畫面 × iPhone 加 Apple Watch——大概一百五十張圖——而我
只要一動到設計系統，這一百五十張就全部過時了。

於是我做了任何一個累壞的獨立開發者都會做的事：寫了一隻機器人。現在整條流程一個指令
跑完，用十種語言上傳到 App Store Connect，而且要重現它不用花半毛錢。這篇文章就是
藍圖——每個階段、那個真正聰明的小技巧，還有一份全免費的清單，讓你可以整套搬走。

### 全貌

一條截圖流程，其實是四件事假裝成一件：

1. **擷取（Capture）**——把每個畫面、每種語言，拍成乾淨的像素。
2. **渲染（Render）**——把那些像素放進有裝置外框、有標語的行銷卡。
3. **穩定（Stabilize）**——可靠地知道「到底有沒有東西真的變了」。
4. **上傳（Upload）**——把最終成品推上 App Store Connect。

多數人會用 UI 測試錄影加上一堆手動 Figma 把這些接起來。我一樣都不想要。下面每個
階段都是一支單純的腳本，再由一個 Claude Code **skill** 統一驅動這四步，我就永遠不用
記順序。

### 第一步：在 App 內截圖，而不是驅動 UI

標準做法是驅動 `XCUITest`——啟動 App、一路點進每個畫面、截圖。我覺得用 UI 測試來
截圖很痛苦：又慢、又不穩，而且只要有顆按鈕移動就會壞掉。

讓這件事變得愉快的關鍵，是把截圖做成 **App 本身的一個功能**，用啟動參數就能到達。
Zestimer 內建一個 `-ScreenshotMode`：它換上一個記憶體內的 SwiftData store、塞入
固定不變的示範資料，再讀 `-ScreenshotScene` 參數直接跳到某個畫面。不用點、不用
導覽、不用等動畫。每個畫面就是一次啟動：

```sh
xcrun simctl launch --terminate-running-process "$UDID" com.your.app \
  -ScreenshotMode -ScreenshotScene activeWorkout -ScreenshotLanguage japanese
xcrun simctl io "$UDID" screenshot --type=png out/ja/active.png
```

把它在 `{畫面} × {語言}` 上跑一圈，整組圖幾分鐘就出來了。我的完整一輪是一百次啟動，
而因為資料是「種」進去的（不是真實資料），日文卡和德文卡顯示的是**同一組**訓練——
這正是審查人員比對各語系時你會想要的效果。有一條規則救了我：碰到未知的
`-ScreenshotLanguage` 應該直接 **trap（崩掉）**，而不是默默退回英文，否則你會在韓文
listing 裡放一張英文卡，卻永遠不會發現。

這裡的一切都免費、而且是 Apple 原生：用 `simctl` 開機、切外觀、覆寫狀態列（把時間
釘在 9:41）、截圖。沒有任何第三方跑腿工具。

### 第二步：把截圖做成廣告卡

模擬器的原始截圖是「說明書」。商店要的是**廣告**：一個裝置外框、一句只賣一個賣點的
標題、一個符合品牌調性的背景。這層「包裝」本身是一個獨立的工具，也是我以前唯一要
付費的階段。

這裡有個重要更新。現在有一個免費的 Claude Code skill：`app-store-screenshots`，
它會在你的專案裡骨架化（scaffold）出一套完整的 Next.js 編輯器：真實解析度的即時
預覽、拖曳排序、行內編輯標語、每個語系各自的文案、明暗色變體、跨畫面的「全景式」
排版，還有透過 `html-to-image` 一鍵匯出所有 Apple 與 Google 要求的尺寸。它會寫出
一份你可以 commit 的 `app-store-screenshots.json`，於是你的卡片排版就跟 App 一起
存在 git 裡。

換句話說，這條流程裡看起來最貴的部分——那些有外框、有標語的行銷卡——現在只差一個
`npm run`，免費，而且完全跑在你自己的機器上。我自己的渲染器會依語言餵入標語、每個
語系匯出成一個資料夾；這個 skill 的模板開箱即用，做的是同一件事。

### 第三步：讓 Git 成為真相來源

這是我最得意的技巧，而它只有五行 Python。

問題是：渲染出來的卡片會 commit 進 git，但無頭 Chromium 的渲染並不是每個位元組都
穩定的。玻璃材質和抗鋸齒會在每次執行之間抖動個一兩像素。如果你把每次渲染都 commit，
`git status` 就永遠是髒的，你也就永遠回答不了那個唯一重要的問題：**到底有沒有哪張
截圖真的變了？**

解法是一道 diff 閘門。每次渲染後，把新的 PNG 跟已 commit 的那張比對。如果差異只落在
渲染雜訊的範圍內，就**保留舊的位元組**：

```python
old = numpy.asarray(Image.open(previous).convert("RGB"), dtype=numpy.int16)
new = numpy.asarray(Image.open(fresh).convert("RGB"), dtype=numpy.int16)
# 抖動每像素落在十幾的量級；真正的改動（重畫的數字、位移的文字）落在數百。
# 用 32 就能把兩者乾淨地切開。
changed = int((numpy.abs(old - new).sum(axis=2) > 32).sum())
if changed < 48:
    shutil.copy(previous, fresh)   # 沒變：還原成已 commit 的位元組
```

於是截圖資料夾一個乾淨的 `git status` **本身就是答案**：什麼都沒變，跳過上傳。髒的
狀態就是一個你可以在上架前親眼看過的真實 diff。整個「我到底需不需要重拍？」的判斷，
坍縮成一句 `git diff`——不用清單、不用 hash、不用狀態檔。你早就有的版本控制，變成了
那個你本來不想自己造的變更偵測系統。

### 第四步：用 Fastlane 上傳

最後一哩是 [fastlane](https://fastlane.tools) 的 `deliver`，它免費、開源。它會透過
API 把每個語系一個資料夾的截圖上傳到 App Store Connect。有兩個細節值得偷：

- **deliver 是依像素尺寸把每張圖分派到裝置家族的。** 所以 416 像素寬的 Apple Watch
  截圖，跟 1242 像素的 iPhone 卡，可以放在**同一個**語系資料夾裡——deliver 會自動把
  它們分到正確的裝置上。檔名前面的數字（`1-list`、`2-active`）決定商店上的排序。
- **一組擷取可以餵兩個 listing。** 西班牙文在商店上是 `es-ES` 和 `es-MX` 兩個
  listing，但截圖完全一樣——所以我只渲染 `es-ES` 一次，上傳前把資料夾鏡像到
  `es-MX`。

把流程拆成三支腳本——`generate`（擷取 + 渲染 + 就位）、`publish-screenshots`、
`publish-metadata`——讓那個又慢又必須序列化的上傳自成一步，只有在第三步說「有東西
變了」時才跑。

### 驅動整條流程的那個 Skill

四支腳本，仍然是四件要記的事，還得照順序、帶對參數。真正讓它「像一隻機器人」的那
一層，是一個 Claude Code **skill**。

有兩個基本元件在做事，值得分清楚：

- **`AGENTS.md`**（或 `CLAUDE.md`）是**框架（harness）**——你的 AI 夥伴在每個任務上
  都遵守的常駐規則：架構、設計 token、語氣、以及「除非是發版否則禁止編譯 App」這種
  禁令。它是憲法。
- **skill** 是**打包好的流程**——一份帶 front matter 的 Markdown，讓 agent 隨叫隨用。
  它是一份你寫一次、用一個斜線指令觸發的作業手冊（runbook）。

我的 `/release` skill 就是發版的作業手冊。它的 front matter 告訴 agent 什麼時候該
啟動；它的本體就是那份檢查清單：

```markdown
---
name: release
description: 端到端完成 App Store 發版——從 git log 為每個語系草擬 What's New、
  升版號、同步網站、當畫面有變時擷取並上傳截圖、推送 metadata。用於 "/release"。
---

1. Diff `last-release..HEAD`，只留下使用者看得到的改動。
2. 為每個語系草擬 What's New（要真正改寫，絕不逐字翻譯）。
3. 升版號。
4. 若有被渲染的畫面變了，跑 generate → publish-screenshots。
5. 推送 metadata。回報你走了哪條路——指令沒成功就絕不宣稱上傳成功。
```

當我打 `/release 1.4`，agent 會讀取上次發版以來的 git log、用十一種語系寫好發版
說明、升版號、從 diff 判斷有沒有任何**被渲染的**畫面變了，而且——只有在真的變了
時——才在背景跑「擷取—渲染—上傳」這條鏈。它最後只把 Apple 卡在「簽名建置」後面的
兩件事交還給我：Xcode 封存（archive），以及最後按下 Submit。判斷（「這個 diff 有沒有
動到畫面？」「這則說明誠不誠實？」）活在 skill 的文字裡；機械操作活在它呼叫的腳本裡。

這才是真正的解鎖。腳本讓工作**變得可能**；skill 讓工作**變得可以交付出去**。

### 免費複製這一切

這是完整的零件清單。每一行都免費或開源：

| 階段 | 工具 | 費用 |
| --- | --- | --- |
| 擷取 | `simctl` + App 內建 `-ScreenshotMode` | 免費（Apple） |
| 渲染 | `app-store-screenshots` Claude skill | 免費、在地 |
| 穩定 | 約 10 行 Python + NumPy/Pillow | 免費 |
| 上傳 | fastlane `deliver` | 免費（開源） |
| 編排 | 一個 Claude Code skill + `AGENTS.md` | 免費 |

唯一一個曾經要花錢的環節——行銷卡渲染器——現在就是那個免費的 skill。所以誠實的
結論是：**整條流程可以用零元重現。** 幫你的 App 加上 `-ScreenshotMode`、用 skill
骨架化出渲染器、放進那道 diff 閘門、把 fastlane 指向那些資料夾，再寫一份 `/release`
作業手冊把它們串起來。

### 結語

截圖從來不是難的部分。難的是那個**迴圈**——每一次設計改動都會默默讓一百張圖失效，
而要發現這件事的代價，是打開 App Store Connect 瞇著眼睛比對。把擷取自動化，省的是
幾分鐘；讓 git 成為真相來源，救的是那個迴圈。而把整件事寫成一個 skill，意味著機器人
可以自己跑它，而不只是我。

獨立開發者做出的最好工具，往往不是功能，而是那些讓一個人能像一支團隊一樣持續出貨的
機器——而且，越來越常見的是，它們是免費的。
