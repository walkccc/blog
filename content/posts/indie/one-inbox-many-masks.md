+++
date = 2026-07-30T00:00:00-04:00
title = "One Inbox, Many Masks: Running Several Products as One Person"
tags = ["Indie", "Marketing", "Ops"]
categories = ["Essay"]
+++

_[English](#en) · [中文](#zh)_

<a id="en"></a>

## English

I now have three products — [Zestimer](https://zestimer.com), [Furioke](https://furioke.com), and [Tefuda](https://playtefuda.com) — plus a studio domain, `magicparklabs.com`, with a Google Workspace behind it. Four domains, one person, evenings only.

Every time I ship a new one, the same instinct shows up: _this product should have its own everything._ Its own X account, its own Threads handle, its own support inbox, its own logins to Stripe and Supabase and App Store Connect. It feels tidy. It feels like what a real company would do.

It's half right, and the half that's wrong is expensive. Here's the rule I settled on:

> **Products get masks, not accounts.**
> One identity behind the curtain, as many faces as you like in front of it.

The interesting part is that the two halves point in _opposite directions_. Out front, consolidate onto **the person** — you. Out back, consolidate onto **the studio** — a legal-ish entity that outlives your enthusiasm for any one app. Getting that backwards is how solo developers end up with six inboxes and no readers.

### Front of House: Market From Your Own Account

The tempting move is to spin up `@zestimer` and `@playtefuda` and start posting product updates. Don't — not first.

**A new brand account starts at zero twice over.** Zero followers is the obvious one. The one that actually hurts is zero _signal_: a freshly registered account whose entire history is promotional links is exactly the shape of a spam account, and the ranking systems on X, Threads, and Reddit treat it accordingly. You will post into a room with nobody in it, for months, and conclude that marketing doesn't work.

Your personal account isn't in that hole. It has history, real replies, a follow graph, and — this is the part that compounds — permission to be interesting about something other than the product.

**People follow builders, not changelogs.** Nobody subscribes to a feed of "v1.3.4 is out." They will absolutely read "here's the five-line diff gate that finally told me whether a screenshot changed," or "I picked iOS 26 for one app and iOS 18 for another in the same year, and here's the argument." The product shows up inside the story, and the story is the thing worth following. The conversion path is longer and the reach is an order of magnitude better.

**It's also the only version you'll actually keep doing.** One person cannot ship code, answer support, and feed three brand accounts across three platforms. Whatever you design has to survive a bad week at your day job. One account survives; nine don't.

So the split I use:

- **Personal account** — the primary channel. Build logs, the bug that ate a Saturday, short clips of a screen I'm proud of, the occasional number. Bio says `Maker of Zestimer, Furioke & Tefuda` with the links.
- **Product handles** — registered immediately, on every platform I might ever want, and then left mostly alone. This is squatting insurance, and it costs one evening. A bio, a link, a pointer back at me.

**A brand account earns its keep when it has a job that isn't storytelling.** That job is usually one of three: status and outage notices, player or user support with a public paper trail, or a community that has started talking to _each other_ rather than to me. Tefuda will get there before the other two, because games generate that kind of traffic. Until the job exists, the account is a chore with no reader.

The objection I had to think about: doesn't tying everything to my personal account destroy the asset if I ever want to sell an app? No — and the reason is the second half of this post. The transferable asset was never the follower count. It's the domain, the Workspace, the App Store record, the users, the revenue. Distribution through a personal account is rented attention, but it's rented to _me_, and it moves to the next product for free. That's the trade: the audience doesn't transfer with the app, and the app doesn't need it to.

### Back of House: One Workspace, Many Domains

Front of house is personal. Back of house should be the exact opposite — nothing important should be sitting in `myname@gmail.com`.

The failure mode here isn't dramatic, it's just slow: one Google account per product. Three products means three logins, three password vaults, three places a verification code might land, and three chances to be locked out of a billing email on the one weekend it matters. Multiply by every product you'll ship in the next five years.

The fix is that **a product needs an address, not an account.** Google Workspace hands you that for free, because you pay per _user_, not per domain.

**1. Add each product domain as a secondary domain.** Admin Console → Account → Domains → Manage domains → Add a domain → _Secondary domain_. Verify it, point the MX records, done. Your Workspace now legitimately owns mail for `zestimer.com` and `playtefuda.com` without adding a seat.

There's also a _domain alias_, which mirrors every existing username onto the new domain automatically. It's less typing, but it means `support@` on one product is the same address as `support@` on every other — fine functionally, sloppy in practice. Secondary domains keep each product's namespace its own. Either way: a domain can belong to only one Workspace at a time, so decide before you point MX anywhere.

**2. Add the product addresses as aliases on your one user.** Directory → Users → your account → User information → Email aliases. Add `support@zestimer.com`, `hello@playtefuda.com`, `contact@magicparklabs.com`. No new seats, no extra bill. Everything lands in the one inbox you already have open.

The ceiling is 30 aliases per user, which sounds infinite until you reflexively create `support@`, `hello@`, `contact@`, `legal@`, and `press@` on every domain. Pick one public address per product and one for the studio.

**3. Reply as the product.** Gmail → Settings → Accounts → _Send mail as_ → add `support@zestimer.com` with the name `Zestimer Support`, and check **Treat as an alias**. Because it's a real alias on the same Workspace account, you don't need a separate SMTP server — Google will send it. Then set _When replying to a message_ → **Reply from the same address the message was sent to**, and you will never again answer a Zestimer user from your personal name by accident.

**4. Do the DKIM step for every domain.** This is the one everyone skips and it's the one that decides whether your replies land in spam. Adding a secondary domain does _not_ authenticate it — Google generates a DKIM key **per domain**. Admin Console → Apps → Google Workspace → Gmail → Authenticate email → select the domain → generate → publish the `google._domainkey` TXT record → start authentication. Then give each domain its own SPF (`v=spf1 include:_spf.google.com ~all`) and a `_dmarc` record, starting at `p=none` with a reporting address and tightening once you've seen a week of reports.

**5. Send app mail from a subdomain, not the root.** Password resets, receipts, and anything your product sends automatically should not go through Gmail. Use Resend, Postmark, or Cloudflare Email — and send from `mail.zestimer.com` rather than `zestimer.com`. Reputation is tracked per sending domain, so a bad campaign or a bounce spike on automated mail stays quarantined away from the root domain you use to answer humans.

**6. Register third-party services under the studio.** Apple Developer, Stripe, the bank, the App Store Connect account holder — all of it on `magicparklabs.com`, never on your personal Gmail. Changing the Apple ID on a live developer account later is genuinely painful, and "which email owns this?" is a question you want to answer once, forever.

For per-product tooling logins (a Supabase project, a Cloudflare account, an API dashboard), plus-tagging is enough: `pengyuc+zestimer-supabase@magicparklabs.com`. Two caveats worth knowing — a minority of services still reject `+` in an email field, and the tag is trivially strippable, so treat it as _filing_, not as a security boundary. Where a service refuses it, spend a real alias.

**7. Label on arrival.** One Gmail filter per domain: `to:zestimer.com` → label `[Zestimer]`. Do this the day you add the domain, not the day the inbox gets confusing.

If you don't have a Workspace, the free version of all of this is Cloudflare Email Routing — same forwarding, same central inbox, $0 — with the tradeoff that you'll need an SMTP relay to reply as the product address.

### The New-Product Checklist

Once the architecture is set, launching a fourth product is a form to fill out, not a decision to make:

| Step | Action                                                     |
| ---- | ---------------------------------------------------------- |
| 1    | Buy the domain, DNS on Cloudflare                          |
| 2    | Add it to Workspace as a secondary domain, verify, set MX  |
| 3    | Add `support@newproduct.com` as an alias on your user      |
| 4    | DKIM + SPF + DMARC for that domain                         |
| 5    | Gmail: Send mail as, treat as alias, reply-from-same       |
| 6    | Gmail filter → `[NewProduct]` label                        |
| 7    | Register the social handles, bio points at your account    |
| 8    | Announce it from your personal account, as a story         |

Eight steps, one evening, and it's identical every time. That's the whole point: **each new product should cost you configuration, not accounts.**

### Why the Two Halves Point Opposite Ways

It looks inconsistent — market as a person, operate as a company — but it's the same principle applied to two different lifespans.

Attention accrues to people. A follower earned by a story about a rendering bug stays with you when the app that had the bug is sunset; a follower of `@zestimer` does not. So the public layer should be the thing with the longest half-life you own, which is you.

Assets should not. Domains, billing, developer accounts, and support history need to survive your interest in any given app, be handed to a collaborator, or be sold — and none of that works if it lives in a personal Gmail with your recovery phone number attached. So the private layer should be the thing that can outlive you, which is the studio.

One inbox, many masks. The masks are cheap, the inbox is the asset, and the person wearing them is the distribution channel.

---

<a id="zh"></a>

## 中文

我現在手上有三個產品——[Zestimer](https://zestimer.com)、[Furioke](https://furioke.com)、[Tefuda](https://playtefuda.com)——再加上一個工作室網域 `magicparklabs.com`，底下掛著一個 Google Workspace。四個網域、一個人、只有下班後的時間。

每次上線一個新產品，同一個直覺就會冒出來：**這個產品應該要有自己的一切**。自己的 X 帳號、自己的 Threads handle、自己的客服信箱、自己的 Stripe 和 Supabase 和 App Store Connect 登入。看起來很整齊，很像一間真的公司會做的事。

它對了一半，而錯的那一半很貴。我最後定下來的規則是：

> **產品要的是面具，不是帳號**。
> 幕後只有一個身分，幕前你想戴幾張臉都行。

有意思的地方在於，這兩半指向**相反的方向**。台前，集中到**人**身上——也就是你。幕後，集中到**工作室**身上——一個能活得比你對任何單一 App 的熱情還久的主體。把這兩件事搞反，就是獨立開發者最後擁有六個收件匣、零個讀者的原因。

### 台前：用你自己的帳號做行銷

最誘人的做法是開一個 `@zestimer`、一個 `@playtefuda`，然後開始發產品更新。別這樣——至少不要一開始就這樣。

**一個新的品牌帳號是「零」了兩次**。零追蹤者是明顯的那個。真正會痛的是零**訊號**：一個剛註冊、整段歷史都是推廣連結的帳號，長得就跟垃圾帳號一模一樣，而 X、Threads、Reddit 的排序系統也就這樣對待它。你會對著一個沒人的房間發文好幾個月，然後得出「行銷沒有用」的結論。

你的本帳不在那個坑裡。它有歷史、有真實的互動、有追蹤關係圖，而且——這才是會複利的部分——它有資格去聊產品以外的東西。

**大家追蹤的是「做東西的人」，不是 changelog**。沒有人會訂閱一個一直說「v1.3.4 上線了」的帳號。但他們絕對會讀「這五行 diff 閘門終於讓我知道截圖到底有沒有變」，或是「同一年裡我一個 App 選 iOS 26、另一個選 iOS 18，理由是這樣」。產品出現在故事裡面，而值得追蹤的是那個故事。轉換路徑比較長，但觸及高一個數量級。

**而且這也是你唯一真的能持續做下去的版本**。一個人沒辦法同時寫程式、回客服，還餵飽三個平台上的三個品牌帳號。你設計出來的東西必須能撐過正職一個很爛的星期。一個帳號撐得住，九個撐不住。

所以我的分工是：

- **本帳**——主要管道。開發日誌、吃掉一整個週六的那個 bug、我自己滿意的畫面短片、偶爾幾個數字。Bio 寫 `Maker of Zestimer, Furioke & Tefuda` 加連結。
- **產品 handle**——馬上註冊，所有我可能會想要的平台都註冊，然後幾乎放著不管。這是防搶註的保險，成本是一個晚上。一段 bio、一個連結、一個指回我本帳的指標。

**品牌帳號要開始有價值，得先有一份「不是說故事」的工作**。那份工作通常是三種之一：狀態與故障公告、需要公開紀錄的用戶客服，或是一個成員開始彼此對話（而不是只對著我說話）的社群。Tefuda 會比另外兩個先走到那一步，因為遊戲本來就會產生那種流量。在那份工作出現之前，品牌帳號只是一件沒有讀者的雜事。

我確實想過一個反駁：把一切綁在本帳上，如果哪天我想賣掉某個 App，不就把資產毀了嗎？不會——理由就在這篇的下半部。可以轉手的資產從來就不是追蹤數，而是網域、Workspace、App Store 紀錄、使用者、營收。透過本帳做分發的確是「租來的注意力」，但它租給的是**我**，而且免費跟著搬到下一個產品。這就是交易條件：受眾不會跟著 App 走，而 App 也不需要它。

### 幕後：一個 Workspace，很多個網域

台前是個人的。幕後應該剛好相反——任何重要的東西都不該躺在 `myname@gmail.com` 裡。

這裡的失敗模式不戲劇化，只是慢慢地爛掉：一個產品開一個 Google 帳號。三個產品就是三組登入、三個密碼庫、三個驗證碼可能寄達的地方，以及三次在最要命的那個週末被鎖在帳單信外面的機會。再乘上你未來五年會做的每一個產品。

解法是：**產品需要的是一個地址，不是一個帳號**。Google Workspace 免費給你這件事，因為它是按**使用者**收費，不是按網域。

**一、把每個產品網域加成附加網域（secondary domain）**。Admin Console → 帳戶 → 網域 → 管理網域 → 新增網域 → **附加網域**。驗證、指好 MX，完成。你的 Workspace 現在名正言順地擁有 `zestimer.com` 和 `playtefuda.com` 的郵件權限，而且沒有多一個席次。

另外還有一種叫**網域別名**（domain alias），它會自動把現有的每個使用者名稱鏡射到新網域上。少打很多字，但代價是某個產品的 `support@` 跟其他所有產品的 `support@` 是同一個地址——功能上沒問題，實務上很邋遢。附加網域讓每個產品保有自己的命名空間。無論選哪種：一個網域同時只能屬於一個 Workspace，所以在把 MX 指出去之前先決定好。

**二、把產品地址加成你那唯一使用者的別名**。目錄 → 使用者 → 你的帳號 → 使用者資訊 → 電子郵件別名。加上 `support@zestimer.com`、`hello@playtefuda.com`、`contact@magicparklabs.com`。不用新席次、不會多收錢。全部落進你本來就開著的那一個收件匣。

上限是每個使用者 30 個別名。聽起來像無限，直到你反射性地在每個網域都開了 `support@`、`hello@`、`contact@`、`legal@`、`press@`。每個產品挑一個對外地址，工作室再一個就夠了。

**三、用產品的身分回信**。Gmail → 設定 → 帳戶 → **以其他地址發送郵件** → 加入 `support@zestimer.com`，名稱寫 `Zestimer Support`，勾選**視為別名**。因為它是同一個 Workspace 帳號上的真別名，你不需要另外接 SMTP——Google 會直接幫你寄。接著把「回覆郵件時」設成**從接收信件的相同地址回覆**，從此你再也不會不小心用個人名字回一封 Zestimer 用戶的信。

**四、每個網域都要做 DKIM**。這是所有人都會跳過、卻決定你的回信會不會進垃圾桶的那一步。加了附加網域**不等於**它通過驗證——Google 是**每個網域**各生一把 DKIM 金鑰。Admin Console → 應用程式 → Google Workspace → Gmail → 驗證電子郵件 → 選那個網域 → 產生 → 發布 `google._domainkey` 的 TXT 記錄 → 開始驗證。然後給每個網域各自的 SPF（`v=spf1 include:_spf.google.com ~all`）和一筆 `_dmarc` 記錄，從 `p=none` 加一個報告收件地址開始，看一週報告之後再收緊。

**五、App 寄的信走子網域，不要走根網域**。密碼重設、收據、任何你的產品自動發出去的東西，都不該經過 Gmail。用 Resend、Postmark 或 Cloudflare Email——而且從 `mail.zestimer.com` 寄，不是 `zestimer.com`。信譽是以「寄件網域」為單位追蹤的，所以自動信件上的一次爛行銷或一波退信，會被隔離在你用來回覆真人的那個根網域之外。

**六、第三方服務一律掛在工作室名下**。Apple Developer、Stripe、銀行、App Store Connect 的帳號持有人——全部用 `magicparklabs.com`，絕不用個人 Gmail。事後要換掉一個已上線開發者帳號的 Apple ID 是真的很痛苦，而「這個東西是哪個 email 擁有的？」是一個你會想一次回答、永久有效的問題。

至於各產品自己的工具登入（某個 Supabase 專案、某個 Cloudflare 帳號、某個 API 後台），plus tagging 就夠了：`pengyuc+zestimer-supabase@magicparklabs.com`。有兩個值得知道的但書——少數服務仍然拒收 email 欄位裡的 `+`，而且那個 tag 隨手就能被剝掉，所以請把它當成**歸檔**，不是安全邊界。碰到不收的服務，就花掉一個真的別名。

**七、進來就貼標籤**。每個網域一條 Gmail 篩選器：`to:zestimer.com` → 標籤 `[Zestimer]`。在你加網域的那一天就做，不要等到收件匣開始亂了才做。

如果你沒有 Workspace，這一切的免費版是 Cloudflare Email Routing——一樣的轉寄、一樣的集中收件匣、零元——代價是你需要接一個 SMTP relay 才能用產品地址回信。

### 新產品檢查清單

架構定下來以後，上線第四個產品就變成填表格，而不是做決定：

| 步驟 | 動作                                             |
| ---- | ------------------------------------------------ |
| 1    | 買網域，DNS 託管到 Cloudflare                    |
| 2    | 加進 Workspace 成為附加網域，驗證、設定 MX       |
| 3    | 在你的使用者底下加 `support@newproduct.com` 別名 |
| 4    | 幫該網域設好 DKIM + SPF + DMARC                  |
| 5    | Gmail：以其他地址發送、視為別名、同址回覆        |
| 6    | Gmail 篩選器 → `[NewProduct]` 標籤               |
| 7    | 註冊社群 handle，bio 指回你的本帳                |
| 8    | 用本帳、以說故事的方式宣布它                     |

八步、一個晚上，而且每次都一模一樣。這就是重點：**每多一個產品，你付出的應該是設定，不是帳號**。

### 為什麼這兩半指向相反的方向

看起來很不一致——用個人的身分行銷、用公司的身分營運——但那其實是同一個原則套用在兩種不同的壽命上。

注意力累積在人身上。一個因為你講了某個渲染 bug 的故事而來的追蹤者，會在那個 App 收攤之後繼續留在你身邊；`@zestimer` 的追蹤者不會。所以對外那一層，應該綁在你擁有的東西裡半衰期最長的那個上面，也就是你自己。

資產不該這樣。網域、帳單、開發者帳號、客服歷史，需要活得比你對任何一個 App 的興趣還久，需要能交給合作者，或是能賣掉——而如果這些東西住在一個綁著你救援手機號碼的個人 Gmail 裡，那全部都做不到。所以對內那一層，應該綁在能活得比你久的那個主體上，也就是工作室。

一個收件匣，很多張面具。面具很便宜，收件匣才是資產，而戴著面具的那個人，就是分發管道。
