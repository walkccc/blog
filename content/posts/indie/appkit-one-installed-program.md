+++
date = 2026-08-02T12:00:00-04:00
title = "appkit: One Installed Program, and the Xcode Windows It Deletes"
tags = ["iOS", "Android", "Swift", "Homebrew", "App Store Connect", "Google Play"]
categories = ["Engineering"]
+++

_[English](#en) · [中文](#zh)_

<a id="en"></a>

## English

The honest origin story is smaller than the engineering: I got tired of remembering `asc` flags.

Not the hard kind of tired — I could look them up. The kind where every release starts with re-deriving a workflow I had already invented twice, because it lived as shell history in three different repos and none of the three agreed with each other. I wanted one thing that was **deterministic** — the same input produces the same bytes, every time, on every machine — and **configurable** — the parts that are genuinely one app's own (a bundle id, a scene name, a caption) live in that app's repo and nowhere else. Everything in between should be a program I install once and never think about again.

That program is `appkit`:

```sh
brew tap walkccc/appkit https://github.com/walkccc/appkit
brew install --HEAD appkit
```

One install, every repo. `appkit doctor` runs from any subdirectory of any app, the way `git status` does, because it walks up looking for `appkit.json` the same way git walks up looking for `.git`.

### The three things a repo owns

Everything else is appkit's:

```
appkit.json           platform, locales, scenes, store ids, version
scripts/scenes.sh     what a store screenshot IS for this app
store/                the storyboard, the words, the finished cards
```

No app repo contains a device resolver, a JWT signer, a CoreGraphics compositor, or a copy of `.prettierrc` that quietly drifted from its siblings. It contains a manifest, one file that says what a screenshot means for this specific app, and the outputs.

The commands are `verb noun`, spelled out on purpose:

```sh
appkit new ios             # scaffold a repo, from nothing
appkit make screenshots    # capture + render, and stop
appkit review screenshots  # every card in one picture
appkit upload metadata     # the listing text, onto the store
appkit ship                # the binary, onto the store
```

Not `appkit ns`, not `--publish-shots`. I stopped needing to remember anything the moment the vocabulary became "what am I trying to do" instead of "what did I name this six months ago."

### Scaffolding a bundle id into a pipeline

Here's the part that used to eat an afternoon. A new app starts in Xcode — File ▸ New ▸ Project, pick a name, pick a bundle id. The moment that project exists, it needs a screenshot pipeline, a listing-text tree in five languages, a `.swiftformat` that matches every other repo, a pre-commit hook, and a manifest tying all of it together. None of that is specific to _this_ app. All of it used to get copied out of whichever sibling repo I had open — which is how a settle floor tuned for one app's animations, or a locale table with a language this app doesn't ship, ends up in a fresh project that has nothing to do with either.

Now:

```sh
appkit new ios --name Tefuda --bundle com.magicparklabs.Tefuda \
  --locales "en ja zh-Hant zh-Hans ko"
```

That writes `appkit.json`, `scripts/scenes.sh`, a `store/cards.json` storyboard whose every caption already speaks all five declared languages, a metadata skeleton, and then runs `appkit sync`. It refuses to overwrite anything that already exists unless you pass `--force`, so running it again to add the Android sibling is safe rather than destructive.

What it deliberately does **not** do is make the Xcode project. appkit never opens, reads, or edits an `.xcodeproj`. Scaffolding the config around an app and building the app are different jobs, and the line between them is one appkit is careful never to cross.

### The `if DEBUG` that makes a screenshot reproducible

`appkit capture` photographs the app by launching it like this, once per scene per language:

```sh
Tefuda -ScreenshotMode -ScreenshotScene board -AppleLanguages "(ja)"
```

That's the entire contract. The app reads those launch arguments, puts itself on exactly that screen with exactly that data, and holds still — the `if DEBUG` branch every app on appkit carries, compiled out of Release entirely. appkit doesn't know what a "board" scene is; `scripts/scenes.sh` in the app's own repo owns that. It owns everything around it: resolving a device, booting one simulator per language and running them in parallel, and the shutter.

The shutter is the piece I'd port to any project, screenshots or not: it doesn't sleep for a guessed number of seconds and hope. It waits for two byte-identical frames.

```sh
appkit capture             device → .screenshots/<language>/<scene>.png
appkit render              those  → store/screenshots/<locale>/NN-name.png
appkit review screenshots  those  → one sheet, every locale at once
```

`render` is a Swift binary — CoreGraphics and CoreText, driven by the JSON storyboard — not a headless browser. Compose the same captures against the same storyboard twice and you get the same bytes twice, which is what makes `git status -- store/screenshots` an actual answer to "did any screen change." That's the whole point of deterministic: a pipeline that produces slightly different pixels every run isn't reviewable, it's noise you learn to ignore, and the day you ignore a real change is the day a stale screen ships.

`review` is the newest of the three and the one I should have written first. `git status` says _whether_ a card changed; it cannot say **which**, because a PNG diff in a terminal is a line telling you the bytes differ. So it draws a contact sheet — one row per locale, one column per card, red where git says a card changed and green where it's new. A hole in the grid is a locale short of a scene, which is the most common way a set goes wrong and the hardest thing to see one file at a time.

If you want the pipeline as a blueprint to copy rather than as engineering, that's [The Free Screenshot Robot](/posts/indie/free-app-store-screenshot-robot/) — same five stages, no appkit required.

`appkit doctor` catches the other silent failure: every iOS card here is 1242×2688, which files under Apple's `IPHONE_65`. `asc screenshots upload --device-type` is a filter, not a label — a set whose pixels don't match any declared type uploads **zero files and reports success**. Doctor checks the storyboard's own dimensions against `appkit.json`'s declared types on every run, because I have, more than once, scaffolded `IPHONE_69` next to a 1242×2688 storyboard and not found out until a listing quietly kept its old screenshots.

### `appkit ship`: everything between ⌘B and TestFlight

This is the one I built the rest of appkit to get to.

The old way: ⌘B, then Product ▸ Archive, and wait for Xcode's window to finish doing whatever it does. Then the Organizer opens, and you click Distribute App, and pick App Store Connect, and click through four more screens that ask questions you answered identically the last twenty times, and wait again while it uploads. Then you tab over to a browser, open App Store Connect, click into TestFlight, and refresh the build list every couple of minutes until "Processing" turns into something you can attach to a version. None of that is hard. All of it is a person sitting in front of a spinner, because the tool wants to be watched.

```sh
appkit ship --dry-run    # print the plan, touch nothing
appkit ship              # archive, upload, wait for processing, attach
appkit ship --submit     # …and send it for review
```

Underneath it hands the project to `asc publish appstore --wait`, which blocks until Connect finishes processing the binary, then attaches it. One process, no context switch, no polling by hand. And the build number is resolved **against what Connect already holds**, not guessed locally — because a number picked on my machine gets rejected as a duplicate only _after_ the archive finishes, which is the slow half of the whole operation. Finding that out at the start instead of the end is most of what "the tool respects my time" means in practice.

The split between `appkit ship` and `appkit ship --submit` is deliberate, not decorative. Attaching a build to a version is reversible in the Connect UI — you can swap it before anyone sees it. Submitting for review is not something you take back by clicking around, so it costs one extra word, typed on purpose, rather than being the default behavior of the command that builds.

Android has no Organizer to route around — `appkit ship` on that platform puts the AAB straight on a track, same verb, same manifest.

### Not memorizing `asc`, deliberately

The metadata side has the same shape. I don't type into App Store Connect's text fields by hand, and I don't hold `asc metadata` syntax in my head either:

```sh
appkit pull metadata     # what the store actually shows, right now
appkit check metadata    # every field's length, every language's coverage
appkit upload metadata   # send it, every locale, in one run
```

`check` counts characters, not bytes, because every CJK locale fails a byte count that `en-US` sails through, and an app on both stores is checked against the tighter of the two limits before anything goes out. `pull` exists because the tracked tree is only the source of truth if it's kept current: someone tightens a subtitle from the Connect dashboard during a review, that edit never comes back to the repo, and the next upload silently overwrites it with the older tracked copy. Pulling first is how "what should the listing say" and "what does the listing currently say" stay the same question.

### The command is the mechanism, the skill is the judgement

Not everything that repeats is a script. Some of it is a decision that depends on context a script doesn't have — whether a settle floor reaches past an animation's _start_, whether this is the release where `--submit` is safe, whether a caption should be borrowed verbatim from the app's own UI strings. That's what the skills are, linked once per machine into `~/.claude/skills`:

| Skill | Judgement it carries |
| --- | --- |
| `/appkit-scaffold` | what a fresh repo needs after `appkit new` writes files |
| `/appkit-screenshots` | which of the pipeline stages actually broke, and why |
| `/appkit-metadata` | store limits, keyword rules, when to pull before editing |
| `/appkit-release` | the release order, and which steps are reversible |

The command writes files and moves bytes. The skill is what I used to hold in my head and re-derive every release — now it's read by whatever's helping me ship that day, and it doesn't degrade between releases the way my own memory of "what did I do last time" reliably does.

### What's still deliberately by hand

appkit has a short, sharp list of things it refuses to do, and the list is as important as the commands. It never runs `xcodebuild` or `gradlew` outside of `appkit run`, `appkit ship`, and `appkit capture --build`. It never creates the Xcode or Android Studio project. It never makes the Google Play app record, because Play has no API for that.

And the values that make one app look like itself — `Spacing.lg`, a brand palette, a title card somebody actually illustrated instead of composed — are explicitly not shipped from here; only the _shape_ of where they live is. Three apps had a `Spacing.lg` meaning three different values. Sharing the file would have moved layout in shipped apps; sharing the paragraph could not.

The bar for what belongs in appkit at all is one sentence: is this true of every app, or only true of the one currently open in front of me. Most of the engineering effort, honestly, has gone into deciding what stays out.

### What it comes down to

One binary. One manifest per repo. A handful of skills for the parts that need judgement instead of a script. And a release that now looks like this:

```sh
appkit make screenshots
appkit review screenshots
appkit upload metadata 1.3.0
appkit upload screenshots 1.3.0
appkit ship --submit
```

Five lines, none of which I have to remember the internals of, all of which do exactly the same thing on the next app as they did on this one. That was always the ask — not a faster Xcode, just fewer things I have to hold in my own head between releases.

---

<a id="zh"></a>

## 中文

誠實的起源比工程本身小得多：我受夠了記 `asc` 的參數。

不是那種很難的累。參數我查得到。是那種每次發版都要重新推導一次「我上次到底怎麼做的」的累——因為那套流程只存在三個 repo 的 shell history 裡，而且三份互相不一致。

我要的東西只有兩個性質：**決定性**——同樣的輸入，在任何一台機器上、任何一次執行，都產生同樣的 bytes；以及**可設定**——真正屬於某一個 App 的東西（bundle id、畫面名稱、標語）只活在那個 App 的 repo 裡，其他地方都不准有。中間所有的東西，都應該是一個我裝一次、然後再也不用想起來的程式。

那個程式就是 `appkit`：

```sh
brew tap walkccc/appkit https://github.com/walkccc/appkit
brew install --HEAD appkit
```

裝一次，所有 repo 通用。`appkit doctor` 可以從任何 App 的任何子目錄執行，就像 `git status` 一樣——因為它是往上找 `appkit.json`，就像 git 往上找 `.git`。

### 一個 repo 只擁有三樣東西

其他全部都是 appkit 的：

```
appkit.json           平台、語系、畫面、商店 id、版本
scripts/scenes.sh     對這個 App 來說，一張商店截圖「是什麼」
store/                分鏡表、文案、成品卡片
```

沒有任何一個 App repo 裡有裝置解析器、JWT 簽章器、CoreGraphics 合成器，或是一份早就悄悄跟兄弟 repo 走鐘的 `.prettierrc`。它只有一份 manifest、一個說明「截圖對這個 App 而言是什麼」的檔案，以及產出。

指令一律是 `動詞 名詞`，而且刻意寫得很長：

```sh
appkit new ios             # 從零骨架化一個 repo
appkit make screenshots    # 擷取 + 渲染，然後停
appkit review screenshots  # 所有卡片，一張圖看完
appkit upload metadata     # 文案，上傳到商店
appkit ship                # 二進位檔，上傳到商店
```

不是 `appkit ns`，也不是 `--publish-shots`。當這套詞彙從「我六個月前把它取叫什麼」變成「我現在想做什麼」的那一刻，我就不再需要記任何東西了。

### 把一個 bundle id 骨架化成一條流程

以前這件事會吃掉一個下午。一個新 App 從 Xcode 開始——File ▸ New ▸ Project，取個名字，選個 bundle id。那個專案存在的瞬間，它就需要一條截圖流程、一棵五種語言的文案樹、一份跟其他 repo 一致的 `.swiftformat`、一個 pre-commit hook，還有一份把這些全綁在一起的 manifest。這些沒有一項是**這個** App 特有的。而它們以前全都是從我剛好開著的某個兄弟 repo 複製過來的——這就是為什麼一個為了別的 App 的動畫調出來的 settle floor，或是一張含著這個 App 根本沒上的語言的語系表，會出現在一個跟它們毫無關係的新專案裡。

現在是：

```sh
appkit new ios --name Tefuda --bundle com.magicparklabs.Tefuda \
  --locales "en ja zh-Hant zh-Hans ko"
```

它會寫出 `appkit.json`、`scripts/scenes.sh`、一份每句標語都已經會講五種語言的 `store/cards.json` 分鏡表、一棵 metadata 骨架，然後跑 `appkit sync`。除非你加 `--force`，否則它拒絕覆蓋任何已存在的檔案——所以再跑一次來加 Android 那邊是安全的，而不是破壞性的。

它刻意**不做**的事，是建立 Xcode 專案。appkit 從不打開、讀取或修改任何一個 `.xcodeproj`。把設定骨架化，跟建置 App，是兩件不同的工作，而這條界線是 appkit 小心翼翼從不跨過的。

### 讓截圖可以重現的那個 `if DEBUG`

`appkit capture` 拍 App 的方式，是每個畫面、每種語言各啟動它一次：

```sh
Tefuda -ScreenshotMode -ScreenshotScene board -AppleLanguages "(ja)"
```

整份契約就這樣。App 讀這些啟動參數，把自己放到那個畫面、帶著那份資料，然後靜止不動——這是每個裝了 appkit 的 App 都帶著的 `if DEBUG` 分支，在 Release 裡整段被編譯掉。appkit 不知道 `board` 這個畫面是什麼；那是 App 自己 repo 裡的 `scripts/scenes.sh` 的事。它擁有的是周圍所有東西：解析裝置、每種語言開一台模擬器並行跑，還有快門。

快門是我會搬去任何專案的那一塊，跟截圖無關：它不會 sleep 一個猜出來的秒數然後祈禱，它等的是**連續兩張 byte 完全相同的畫格**。

```sh
appkit capture             裝置   → .screenshots/<語言>/<畫面>.png
appkit render              那些   → store/screenshots/<語系>/NN-名稱.png
appkit review screenshots  那些   → 一張總覽表，所有語系一次看完
```

`render` 是一個 Swift 執行檔——CoreGraphics 加 CoreText，由 JSON 分鏡表驅動——不是無頭瀏覽器。同一組擷取配同一份分鏡表跑兩次，你會得到兩次一樣的 bytes，而這正是 `git status -- store/screenshots` 能真正回答「有沒有畫面變了」的原因。這就是「決定性」的全部意義：一條每次都吐出稍微不同像素的流程，不是可以審查的，它是你會學著忽略的雜訊——而你忽略一個真實改動的那天，就是一張過期畫面上架的那天。

`review` 是三者中最新的一個，也是我早該先寫的那一個。`git status` 說的是「有沒有」卡片變了；它說不出**是哪一張**，因為 PNG 在終端機裡的 diff 就是一行「這兩份 bytes 不一樣」。所以它會畫一張總覽表——一列一個語系，一欄一張卡，git 說變了的框紅色，新增的框綠色。格子上的一個洞，就是某個語系少了一個畫面，而那是整組圖最常見的出錯方式，也是一張一張看時最難發現的。

如果你要的是一份可以照抄的藍圖而不是工程細節，那是 [免費的截圖機器人](/posts/indie/free-app-store-screenshot-robot/)——同樣五個階段，不需要 appkit。

`appkit doctor` 抓的是另一個無聲的失敗：這裡每張 iOS 卡都是 1242×2688，歸在 Apple 的 `IPHONE_65` 底下。而 `asc screenshots upload --device-type` 是一個**過濾器**，不是標籤——一組像素對不上任何已宣告類型的圖，會上傳**零個檔案，然後回報成功**。doctor 每次都會拿分鏡表自己的尺寸去對 `appkit.json` 宣告的類型，因為我不只一次在一份 1242×2688 的分鏡表旁邊骨架化出 `IPHONE_69`，然後直到某個 listing 安靜地留著舊截圖，才發現這件事。

### `appkit ship`：⌘B 跟 TestFlight 之間的所有東西

這是我為了抵達它，才做出 appkit 其他部分的那一個。

舊的做法：⌘B，然後 Product ▸ Archive，等 Xcode 的視窗做完它在做的那些事。接著 Organizer 打開，你按 Distribute App，選 App Store Connect，再點過四個問你「上二十次都答一樣」的畫面，然後再等它上傳。然後你切到瀏覽器，打開 App Store Connect，點進 TestFlight，每隔幾分鐘重新整理一次建置清單，直到「處理中」變成一個你可以掛到版本上的東西。這裡面沒有一步是難的。全部都是一個人坐在轉圈圈前面，因為那個工具想被人看著。

```sh
appkit ship --dry-run    # 印出計畫，什麼都不碰
appkit ship              # 封存、上傳、等處理完、掛上去
appkit ship --submit     # ⋯⋯然後送審
```

底層它把專案交給 `asc publish appstore --wait`，`--wait` 會一路擋到 Connect 處理完二進位檔、再把它掛上去。一個行程，不用切換情境，不用手動輪詢。而且 build number 是 **對著 Connect 已經有的東西**解出來的，不是在本機猜的——因為一個在我機器上挑的號碼，會在封存**完成之後**才被判定重複，而封存是整個操作裡最慢的那一半。在開頭而不是結尾發現這件事，就是「這個工具尊重我的時間」在實務上的大部分意思。

`appkit ship` 跟 `appkit ship --submit` 之間的分界是刻意的，不是裝飾。把一個建置掛到版本上，在 Connect 介面裡是可逆的——在任何人看到之前你都可以換掉。送審不是你點一點就能收回的事，所以它要多打一個字，刻意地打，而不是變成「那個負責建置的指令」的預設行為。

Android 沒有 Organizer 要繞過——`appkit ship` 在那個平台上就是把 AAB 直接放上某個 track，同一個動詞，同一份 manifest。

### 刻意不去記 `asc`

metadata 那一側是同樣的形狀。我不用手在 App Store Connect 的欄位裡打字，也不把 `asc metadata` 的語法記在腦子裡：

```sh
appkit pull metadata     # 商店現在實際顯示的是什麼
appkit check metadata    # 每個欄位的長度、每個語言的覆蓋率
appkit upload metadata   # 送出去，所有語系，一次跑完
```

`check` 數的是字元不是位元組，因為每個 CJK 語系都會在一個 `en-US` 輕鬆通過的位元組檢查上失敗；而一個同時在兩個商店上的 App，會先對著兩者中比較嚴的那個限制檢查，然後才送任何東西出去。`pull` 存在的理由是：被追蹤的那棵樹，只有在它保持最新時才是真相來源。有人在審查期間從 Connect 後台改緊了一句副標，那個編輯永遠不會回到 repo，而下一次上傳會用比較舊的追蹤版本安靜地覆蓋掉它。先 pull，是讓「listing 應該說什麼」跟「listing 現在說什麼」保持是同一個問題的方法。

### 指令是機制，skill 是判斷

不是每一件重複的事都該變成腳本。有些是取決於腳本沒有的上下文的判斷：一個 settle floor 有沒有蓋過某個動畫的**起手**、這一版跑 `--submit` 安不安全、某句標語該不該直接沿用 App 自己的 UI 字串。這就是那些 skill，每台機器連結一次到 `~/.claude/skills`：

| Skill                 | 它承載的判斷                                  |
| --------------------- | --------------------------------------------- |
| `/appkit-scaffold`    | `appkit new` 寫完檔案後，一個新 repo 還缺什麼 |
| `/appkit-screenshots` | 壞掉的到底是流程的哪一階段，以及為什麼        |
| `/appkit-metadata`    | 商店限制、關鍵字規則、什麼時候該先 pull       |
| `/appkit-release`     | 發版順序，以及哪些步驟是可逆的                |

指令寫檔案、搬 bytes。skill 是我以前得放在腦子裡、每次發版重新推導一次的東西——現在它由那天幫我出貨的東西讀，而且不會像我對「上次我到底做了什麼」的記憶一樣，在兩次發版之間衰退。

### 刻意留在手動的部分

appkit 有一份很短、很銳利的「它拒絕做的事」清單，而這份清單跟那些指令一樣重要。它不會在 `appkit run`、`appkit ship`、`appkit capture --build` 以外的地方跑 `xcodebuild` 或 `gradlew`。它不會建立 Xcode 或 Android Studio 專案。它不會建立 Google Play 的 App 紀錄，因為 Play 根本沒有那個 API。

而那些讓一個 App 看起來像它自己的**值**——`Spacing.lg`、品牌色盤、一張真的有人畫出來而不是合成出來的標題卡——明確地不從這裡出貨；只有它們該住在哪裡的**形狀**才是共用的。我有三個 App 各有一個 `Spacing.lg`，三個值都不一樣。共用那個檔案，會動到已經上架的 App 的排版；共用那段文字，不可能。

什麼東西夠格進 appkit，標準只有一句話：這件事是不是對每一個 App 都成立，還是只對我現在正開著的這一個成立。老實說，這裡大部分的工程心力，都花在決定什麼不要進來。

### 講到底

一個執行檔。每個 repo 一份 manifest。少數幾個 skill，負責那些需要判斷而不是腳本的部分。以及一個現在長這樣的發版：

```sh
appkit make screenshots
appkit review screenshots
appkit upload metadata 1.3.0
appkit upload screenshots 1.3.0
appkit ship --submit
```

五行，沒有一行的內部我需要記得，而且每一行在下一個 App 上做的事，跟在這一個上一模一樣。這一直都是我要的——不是一個更快的 Xcode，只是兩次發版之間，少一些我得自己放在腦子裡的東西。
