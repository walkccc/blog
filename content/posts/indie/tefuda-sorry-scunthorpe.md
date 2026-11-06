+++
date = 2026-07-24T12:00:00-04:00
title = "[Tefuda] Sorry, Scunthorpe: 200 Lines Between a Text Box and the Internet"
tags = ["TypeScript", "Unicode", "Cloudflare", "Indie"]
categories = ["Engineering"]
+++

_[English](#en) · [中文](#zh)_

<a id="en"></a>

## English

[Tefuda](https://playtefuda.com) has exactly one field a player can type free text into: sixteen characters, under the results panel, next to a button that says Post. Everything else on screen is a card.

One text box. It cost two hundred lines, five languages, and an apology to a town in Lincolnshire.

### The Cap on the Input Is Not the Cap

A name is the one thing a player writes that everybody else reads — the board prints it on the row, in the replay, and on the preview card of a shared run.

The first trap is free of charge: HTML's `maxLength` counts UTF-16 units, not characters. Paste in twelve emoji and it cheerfully calls that twenty-four, then cuts the name through the middle of a surrogate pair. You store half a character; the board draws a replacement glyph. So the clip spreads instead of slicing:

```ts
export const MAX_NAME = 16;
const STRIPPED = /[\u0000-\u001F\u007F-\u009F\u200B-\u200F\u202A-\u202E]/g;

export function clipName(raw: string): string {
  return [...raw.replace(STRIPPED, "")].slice(0, MAX_NAME).join("");
}
```

`STRIPPED` is the interesting half. Control characters, zero-width characters and the bidi overrides at U+202A–U+202E are all invisible in the field, and none of them stay inside the name once it is on the board: an unterminated right-to-left override reverses what gets printed _after_ it — the score, the rank, whatever the row had planned. A sixteen-character name is allowed to re-typeset its neighbours.

Hence the rule at the top of the file: **a cap enforced on one side only is not a cap.** `maxLength` stops a person, not a `POST`, so the form and the Worker import the same module. The form is the courtesy; the server is the rule.

### Two Lists, Not One

The obvious build is one array of bad words and `.includes()`. It is also exactly how you tell a player named Cassandra she can't play. So there are two.

**`ANYWHERE`** holds terms long enough, or ugly enough, that no ordinary name contains one by accident; they match as substrings, after the gaps are closed. **`WORDS`** holds the short ambiguous ones — `ass`, `coon`, `spic`, `dick`, `jap` — slurs on their own and ordinary syllables inside `Cassandra`, `raccoon`, `spicy`, `Dickens`, `Japan`; those match whole words only.

That split is the entire design; everything after it is Unicode. One asymmetry: CJK terms only ever go in the first list, because a word boundary a regex can see does not exist in Chinese or Japanese.

### The Fold

Then come the classics: SHOUTING, Ｆｕｌｌｗｉｄｔｈ, a dot between every letter, a 3 where an e should be.

```ts
function fold(raw: string): string {
  return [
    ...raw
      .normalize("NFKD")
      .replace(/\p{M}/gu, "")
      .normalize("NFC")
      .toLowerCase(),
  ]
    .map((ch) => LEET[ch] ?? ch)
    .join("");
}
```

NFKD folds fullwidth forms and ligatures down to ASCII and splits accents off their letters, so stripping `\p{M}` turns `fück` into `fuck`. Then lowercase, then the leet table. Then, before the `ANYWHERE` pass, everything that isn't a letter or a digit is deleted outright, which makes `f.u.c.k`, `f-u-c-k` and `f u c k` one word.

The filter itself is nine lines. The other two hundred are vocabulary and reasons:

```ts
export function isBlockedName(raw: string): boolean {
  const folded = fold(raw);
  const closed = folded.replace(BETWEEN, "");
  return (
    ANYWHERE.some((term) => closed.includes(term)) ||
    folded.split(BETWEEN).some((word) => WORDS.has(word))
  );
}
```

**And now my favourite bug in the project.** Run NFKD on `씨발` and you do not get `씨발`. You get 씨발 — jamo, five code points that render like the two syllables you started with and match nothing in a list written in syllables. Every Korean term I had typed out was decoration.

The fix is one extra step: decompose, strip the marks, then **compose again**. NFC puts Korean back into syllables and leaves the folded Latin where it landed, because `ｆ` has no business recomposing into anything. Without the test that caught it, I would still not know.

### Sorry, Scunthorpe

Scunthorpe is a town in Lincolnshire whose name contains four unfortunate letters. AOL locked its residents out of the internet over it in 1996, roughly every filter since has re-enacted the moment, and the whole genre of mistake is named after the place.

Mine blocks it too, on purpose, and there is a test that says so out loud:

```ts
it("takes the Scunthorpe side of the trade, deliberately", () => {
  // A false positive costs one player one retry; a false negative puts a slur
  // on a public board. So the long unambiguous terms match anywhere, and this
  // town pays for it. Recorded as a decision, not discovered as a bug.
  expect(isBlockedName("Scunthorpe")).toBe(true);
});
```

In source, a deliberate trade-off and an unnoticed defect look identical — until somebody writes the assertion that tells them apart. Cassandra, meanwhile, pays nothing, because `ass` is on the other list. That is the entire reason there are two.

### `a s s` Gets Through, and That's the Ceiling

There's a test for that too, and it asserts the failure:

```ts
it("can be beaten on purpose, and that is the known ceiling", () => {
  expect(isBlockedName("a s s")).toBe(false);
});
```

Spacing is precisely what makes those three words rather than one, and blocking it means going back to blocking every name that merely contains the letters.

So the honest description is a floor, not moderation. It knows five languages the way a list knows anything, and what it is for is the name typed to see what happens — which is nearly all of them. Anyone who sets out to beat it will, and the backstop stays what it always was: every row can be deleted. That's a better place to land than a filter that thinks it's a wall.

### The Smug Part: A Filter That Shipped Without Shipping

A blocked name comes back as `bad_name` — the same error code an empty name gets, not one of its own. That isn't squeamishness, though a screen reading "your name was rejected as a slur" is worse in every direction than one reading "that name won't work". It's that `bad_name` is already read by every client, including an iOS app that has been on the App Store since before the list existed. A code of its own means editing five dictionaries and taking an App Review round trip, in order to say a sentence nobody should have to read twice.

So the filter took effect on a native app without the native app being updated. It lives in the Worker, at the only place a name actually arrives:

```ts
function postedName(raw: unknown): string | null {
  if (typeof raw !== "string") return null;
  const name = cleanName(raw);
  return name.length > 0 && !isBlockedName(name) ? name : null;
}
```

Side effect I like even more: because only the Worker calls `isBlockedName`, the vocabulary never reaches a browser. The static export carries `clipName` and not one word of the list — grep the built site for any term on it and you get nothing back. Nobody's kid gets to View Source and find a JavaScript array of a hundred slurs, which is a shipping accident with a long and public history.

### What Generalizes

1. **Split the list by ambiguity, not by severity.** Length is a decent proxy: long terms are safe to match anywhere, short ones need a word boundary.
2. **Normalize twice.** NFKD to fold, NFC to give Hangul back its syllables.
3. **Write the false positive down as a test.** An intentional trade and an unnoticed bug are the same bytes until somebody asserts one of them.
4. **Enforce it where the `POST` lands.** Whatever the form does is a courtesy to the person typing.
5. **Pick an error code your shipped clients already understand.** That's what lets a rule change without a release.
6. **Keep the delete button.** Against somebody actually trying, it's the only part that works.

One text field, sixteen characters, two hundred lines — and the one I'd defend longest is the line that apologises to Scunthorpe.

---

<a id="zh"></a>

## 中文

[Tefuda](https://playtefuda.com) 從頭到尾只有一個能讓玩家自由打字的欄位：十六個字，在結算面板下面、「上榜」按鈕旁邊。畫面上其他東西全都是牌。

一個輸入框。它花了兩百行、五種語言，還欠英國林肯郡一個小鎮一句道歉。

### 輸入框上的上限不是上限

名字是玩家唯一自己打的東西，也是所有人都會看到的東西：榜上那一列、重播畫面、別人分享一局時的預覽卡，三個地方都會印出來。

第一個陷阱是免費附贈的：HTML 的 `maxLength` 數的是 UTF-16 單位，不是字。貼進去十二個 emoji，它會很開心地算成二十四個，然後從某個代理對（surrogate pair）的正中間把名字切斷。存進去的是半個字元，畫在榜上的是一個問號方塊。所以裁切用展開，不用 `slice`：

```ts
export const MAX_NAME = 16;
const STRIPPED = /[\u0000-\u001F\u007F-\u009F\u200B-\u200F\u202A-\u202E]/g;

export function clipName(raw: string): string {
  return [...raw.replace(STRIPPED, "")].slice(0, MAX_NAME).join("");
}
```

比較有趣的是 `STRIPPED` 那一半。控制字元、零寬字元，還有 U+202A–U+202E 那幾個雙向文字覆寫（bidi override），在輸入框裡全都隱形，而且一旦上了榜就不會乖乖待在名字裡面：一個沒有關掉的「由右至左覆寫」，會把它**後面**印出來的東西整段翻過來——分數、名次，那一列原本打算顯示的一切。一個十六個字的名字，有權力重新排版它的鄰居。

所以檔案最上面就寫著一條規則：**只在一邊執行的上限不是上限。** `maxLength` 擋得住人，擋不住一個 `POST`，於是表單和 Worker import 的是同一個模組。表單是禮貌，伺服器才是規矩。

### 兩張清單，不是一張

最直覺的做法是一個髒話陣列配 `.includes()`。而你會用它擋掉第一個叫 Cassandra 的玩家。所以清單有兩張。

**`ANYWHERE`** 收的是那種夠長、夠難聽，正常名字不可能不小心撞到的詞；比對前先把空隙關起來，再整串找過去。**`WORDS`** 放的是短而有歧義的那些——`ass`、`coon`、`spic`、`dick`、`jap`——它們單獨出現是髒話，出現在 `Cassandra`、`raccoon`、`spicy`、`Dickens`、`Japan` 裡面則是再正常不過的音節，所以只跟整個詞比對。

這一刀就是整個設計，剩下的都是 Unicode 的事。另外，中日文的詞只會進第一張清單，因為正規表示式看得見的那種詞邊界，在中文和日文裡根本不存在。

### 把花招攤平

接著是那幾招經典：全部大寫、全形、每個字母中間點一個點、用 3 代替 e。

```ts
function fold(raw: string): string {
  return [
    ...raw
      .normalize("NFKD")
      .replace(/\p{M}/gu, "")
      .normalize("NFC")
      .toLowerCase(),
  ]
    .map((ch) => LEET[ch] ?? ch)
    .join("");
}
```

NFKD 會把全形和連字折回 ASCII，也會把重音符號從字母上拆下來，所以把 `\p{M}` 拔掉之後，`fück` 就變回 `fuck`。接著轉小寫，再套上火星文對照表。然後在跑 `ANYWHERE` 之前，把所有不是字母也不是數字的東西整個刪掉——於是 `f.u.c.k`、`f-u-c-k`、`f u c k` 全都是同一個字。

過濾器本體是九行。剩下的兩百行是詞彙和理由：

```ts
export function isBlockedName(raw: string): boolean {
  const folded = fold(raw);
  const closed = folded.replace(BETWEEN, "");
  return (
    ANYWHERE.some((term) => closed.includes(term)) ||
    folded.split(BETWEEN).some((word) => WORDS.has(word))
  );
}
```

**接下來是這個專案裡我最喜歡的一個 bug。** 對 `씨발` 跑 NFKD，你拿到的不是 `씨발`，而是 씨발——韓文字母（jamo），五個碼位，畫出來很像你原本那兩個音節，但跟一張用音節寫成的清單完全對不上。我打進去的每一個韓文詞都只是裝飾。

修法只多一步：分解、拔掉記號，然後**再合成回去**。NFC 會把韓文拼回音節，同時讓已經折下來的拉丁字母留在原地，因為 `ｆ` 沒有理由再變回全形。要不是有那個測試抓到，我到現在都還不會知道。

### 對不起，Scunthorpe

Scunthorpe 是林肯郡的一個小鎮，鎮名裡不幸地夾著四個字母。1996 年 AOL 因此把全鎮居民擋在網路外面，之後大概每一套過濾器都重演過同一齣，久到這類錯誤現在乾脆以這個地名命名。

我的也擋，而且是故意的，還有一個測試把話講明：

```ts
it("takes the Scunthorpe side of the trade, deliberately", () => {
  // A false positive costs one player one retry; a false negative puts a slur
  // on a public board. So the long unambiguous terms match anywhere, and this
  // town pays for it. Recorded as a decision, not discovered as a bug.
  expect(isBlockedName("Scunthorpe")).toBe(true);
});
```

在原始碼裡，刻意的取捨和沒人發現的缺陷長得一模一樣，直到有人寫下那行把兩者分開的斷言。至於 Cassandra，她什麼都不用付，因為 `ass` 在另一張清單上。這就是清單要有兩張的全部理由。

### `a s s` 過得去，而那就是天花板

這件事也有測試，而且斷言的是「擋不住」：

```ts
it("can be beaten on purpose, and that is the known ceiling", () => {
  expect(isBlockedName("a s s")).toBe(false);
});
```

那幾個空白就是讓它變成三個字的原因。要擋它，就得回頭去擋任何含有這幾個字母的名字。

所以誠實的說法是：這是一道下限，不是內容審核。它「懂」五種語言的方式，就是一張清單懂事情的方式，而它真正要處理的，是那種打進去看看會怎樣的名字——那幾乎就是全部。存心要繞過的人一定繞得過去，而最後那道防線從頭到尾沒變過：每一列都刪得掉。這樣的定位，比一個自以為是牆的過濾器實在多了。

### 最爽的部分：一個沒有出版本就上線的過濾器

被擋下來的名字回的是 `bad_name`——跟「名字是空的」同一個錯誤碼，沒有自己的代號。這不是不好意思講白話，雖然一個寫著「你的名字因為含有歧視字眼被拒絕」的畫面，怎麼看都比「這個名字不能用」更糟。真正的理由是：`bad_name` 是每個客戶端本來就讀得懂的碼，包括一個在這張清單存在之前就已經上架 App Store 的 iOS App。給它一個新的錯誤碼，等於要改五本字典、跑一次 App Review 來回，只為了講一句沒人應該讀第二次的話。

於是原生 App 一個字都沒改，這個過濾器就對它生效了。它住在 Worker 裡——名字真正會抵達的地方，也只有那裡：

```ts
function postedName(raw: unknown): string | null {
  if (typeof raw !== "string") return null;
  const name = cleanName(raw);
  return name.length > 0 && !isBlockedName(name) ? name : null;
}
```

還有一個我更喜歡的副作用：因為只有 Worker 會呼叫 `isBlockedName`，那批詞彙從來沒有進過瀏覽器。靜態輸出裡有 `clipName`，卻沒有清單上的任何一個字——對建好的網站 grep 上面任何一個詞，什麼都搜不到。沒有哪個小孩按「檢視原始碼」會撈到一個裝著一百個歧視字眼的 JavaScript 陣列，而這種意外，公開翻車過不只一次。

### 幾條可以直接抄的

1. **清單按「有沒有歧義」分，不要按「有多髒」分。** 長度是不錯的判準：長的詞在任何位置比對都安全，短的詞得抓整個詞。
2. **正規化做兩次。** NFKD 折疊，NFC 把音節還給韓文；少了第二次，你的韓文清單只是裝飾。
3. **誤擋要寫成測試。** 沒有那行斷言，刻意的取捨和沒人發現的 bug 長得一模一樣。
4. **在 `POST` 落地的地方擋。** 表單那一層只是對正在打字的人客氣。
5. **錯誤碼挑客戶端已經認得的。** 這是規則能不發版就改掉的原因。
6. **刪除鍵留著。** 面對存心的人，真正有用的只有它。

一個輸入框，十六個字，兩百行——而其中我會捍衛最久的那一行，是跟 Scunthorpe 道歉的那一行。
