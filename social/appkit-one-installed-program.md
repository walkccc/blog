---
post: content/posts/indie/appkit-one-installed-program.md
url: https://blog.pengyuc.com/posts/indie/appkit-one-installed-program/
drafted: 2026-08-06
---

# appkit: One Installed Program, and the Xcode Windows It Deletes

A CLI is a hard sell on Threads — the audience is not iOS engineers, and a bash harness for App Store screenshots has an addressable crowd in the low hundreds here. So the tool is not the subject in any of these: the pain is, and appkit is the answer at the end.

**Post them as a series, in this order, a few days apart.** #2 leads because it is the one with a picture; a single post about a tool reads as a catalogue, three posts about the same problem accumulate.

## Threads (zh-TW) #2 — 262/500 · post FIRST

**配圖（必要）**：`static/images/appkit-contact-sheet.png` — 143 張卡、11 個語系的總覽表。這則的全部威力都在那張圖，沒有圖不要發。

> Zestimer 有 11 種語言、143 張 App Store 截圖。我每動一次設計，它們就同時全部過期。
>
> 以前要知道哪張真的變了，得上 App Store Connect 一張一張瞇著眼睛比對。
>
> 現在是一張圖：一列一個語言，一欄一張卡，git 說變了的框紅色，新增的框綠色。
>
> 而格子上的那個洞，就是某個語言少了一張——那是整組截圖最常見的出錯方式，也是一張一張看時最難發現的。
>
> https://blog.pengyuc.com/posts/indie/appkit-one-installed-program/

- [ ] Posted

## Threads (zh-TW) #1 — 407/500 · post SECOND

**配圖**：終端機跑完那五行的畫面。

> 以前上架一次 app，我大概要在 Xcode 跟 App Store Connect 之間切十幾次視窗。
>
> Archive、等它轉圈、Organizer、Distribute、點過四個問你上次也答過的畫面，再開瀏覽器重新整理 TestFlight 等 Processing。
>
> 沒有一步是難的。每一步都是人坐在那邊等機器。
>
> 現在整個上架是五行：
>
> ```
> appkit make screenshots
> appkit review screenshots
> appkit upload metadata 1.3.0
> appkit upload screenshots 1.3.0
> appkit ship --submit
> ```
>
> 十一種語言、143 張截圖，同樣的輸入永遠同樣的 bytes。
>
> https://blog.pengyuc.com/posts/indie/appkit-one-installed-program/

- [ ] Posted

## Threads (zh-TW) #3 — 219/500 · post THIRD

無配圖。這則是給工程圈的，共鳴來自那條界線本身。

> 做這個工具最花時間的不是寫功能，是決定什麼不准放進去。
>
> 我三個 app 裡有三個 Spacing.lg，值都不一樣。
>
> 所以設計 token 我只共用「檔案要怎麼分」，數字一個都不共用。共用了值，就是幫我自己一次改壞三個已經上架的 App。
>
> 工具共用形狀，不共用內容。這條線我守得比寫任何功能都久。
>
> https://blog.pengyuc.com/posts/indie/appkit-one-installed-program/

- [ ] Posted

## X (EN) — 238/280

Attach the contact sheet.

> `git status` says whether a store screenshot changed, never which one.
>
> So the pipeline draws itself a contact sheet: 143 cards, 11 locales, red = changed, green = new. A hole in the grid is a missing translation.
>
> https://blog.pengyuc.com/posts/indie/appkit-one-installed-program/

- [ ] Posted

## X (JA) — 249/280 weighted

> App Storeのスクショ、git statusは「変わったか」しか答えない。「どれが」は答えられない。
>
> なので143枚・11言語を1枚のcontact sheetに並べて、変更を赤・新規を緑で囲む方式にした。格子の穴＝その言語だけ画像が無い、が一発で分かる。
>
> https://blog.pengyuc.com/posts/indie/appkit-one-installed-program/

- [ ] Posted
