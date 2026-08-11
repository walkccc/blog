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

Every time I ship a new one, the same instinct shows up: _this product should have its own everything._ Its own X account, its own support inbox, its own logins to Stripe and Supabase and App Store Connect. It feels like what a real company would do.

It's half right, and the half that's wrong is expensive. Here's the rule I settled on:

> **Products get masks, not accounts.**
> One identity behind the curtain, as many faces as you like in front of it.

The two halves point in _opposite directions_. Out front, consolidate onto **the person** — you. Out back, consolidate onto **the studio** — an entity that outlives your enthusiasm for any one app. Getting that backwards is how solo developers end up with six inboxes and no readers.

### Front of House: Market From Your Own Account

The tempting move is to spin up `@zestimer` and `@playtefuda` and start posting product updates. Don't — not first.

**A new brand account starts at zero twice over.** Zero followers is the obvious one. The one that hurts is zero _signal_: a freshly registered account whose entire history is promotional links is exactly the shape of a spam account, and the ranking systems on X, Threads, and Reddit treat it accordingly. You post into an empty room for months and conclude that marketing doesn't work.

Your personal account isn't in that hole. It has history, real replies, a follow graph, and — this is the part that compounds — permission to be interesting about something other than the product. Nobody subscribes to "v1.3.4 is out." People will absolutely read "here's the five-line diff gate that finally told me whether a screenshot changed." The product shows up inside the story, and the story is the thing worth following: longer conversion path, an order of magnitude more reach.

**It's also the only version you'll actually keep doing.** One person cannot ship code, answer support, and feed three brand accounts across three platforms. One account survives a bad week at the day job; nine don't.

So the split: the **personal account** is the primary channel — build logs, the bug that ate a Saturday, the occasional number — with a bio that reads `Maker of Zestimer, Furioke & Tefuda`. **Product handles** get registered immediately, on every platform I might ever want, then left alone with a bio and a link pointing back at me. That's squatting insurance, and it costs one evening.

A brand account earns a seat when it has a job that _isn't_ storytelling: status and outage notices, support with a public paper trail, or a community that has started talking to each other rather than to me. Tefuda will get there first, because games generate that kind of traffic. Until the job exists, the account is a chore with no reader.

The objection I had to think about: doesn't tying everything to my personal account destroy the asset if I ever sell an app? No. The transferable asset was never the follower count — it's the domain, the Workspace, the App Store record, the users, the revenue. Distribution through a personal account is rented attention, but it's rented to _me_, and it moves to the next product for free.

### Back of House: One Workspace, Many Domains

Front of house is personal. Back of house is the exact opposite — nothing important should sit in `myname@gmail.com`.

The failure mode here isn't dramatic, it's just slow: one Google account per product. Three products means three logins, three password vaults, three places a verification code might land, and three chances to be locked out of a billing email on the one weekend it matters.

The fix is one line: **a product needs an address, not an account.** Google Workspace charges per _user_, not per domain, so a single seat can legitimately own mail for every domain you'll ever buy.

The actual clicks, since this is the part every guide waves at:

1. **Add the domain.** Admin Console → Account → **Domains** → _Manage domains_ → **Add a domain**. Choose **Secondary domain**, not domain alias — see the fork below.
2. **Verify it.** Google hands you a TXT record; publish it at the domain's apex in Cloudflare, then hit _Verify_. If your registrar and DNS are both Cloudflare this takes about a minute.
3. **Point MX at Google.** One record on the apex: `smtp.google.com`, priority `1`. (The old five-host `aspmx.l.google.com` set still works, but new domains should use the single record.) Delete whatever parking MX the registrar left behind, or mail silently goes there instead.
4. **Add the alias.** Directory → **Users** → your account → _User information_ → **Email aliases** → add `support@newproduct.com`. No new seat, no extra bill.
5. **Reply as the product.** Gmail → Settings → Accounts → _Send mail as_ → add the address with the name `NewProduct Support`, check **Treat as an alias** (it's a real alias on the same account, so no SMTP server needed), then set _When replying to a message_ → **Reply from the same address the message was sent to**.

The fork in step 1 matters more than it looks. A **domain alias** mirrors every existing username onto the new domain automatically — less typing, but `support@` on one product becomes the same mailbox as `support@` on every other. A **secondary domain** gives each product its own namespace, which is what you want the moment you have more than one. Either way, a domain can belong to only one Workspace at a time, so decide before you point MX anywhere.

Once that architecture is set, launching a fourth product is a form to fill out, not a decision to make:

| Step | Action                                                    |
| ---- | --------------------------------------------------------- |
| 1    | Buy the domain, DNS on Cloudflare                         |
| 2    | Add to Workspace as a secondary domain, verify, set MX    |
| 3    | Add `support@newproduct.com` as an alias on your one user |
| 4    | DKIM + SPF + DMARC for that domain                        |
| 5    | Gmail: Send mail as → treat as alias → reply-from-same    |
| 6    | Gmail filter → `[NewProduct]` label                       |
| 7    | Register the social handles, bio points at your account   |
| 8    | Announce it from your personal account, as a story        |

Three of those rows are where it actually goes wrong:

- **DKIM is per domain (step 4).** Adding a secondary domain does _not_ authenticate it — Google generates a key per domain, and skipping this is what decides whether your replies land in spam. Admin Console → Apps → Google Workspace → **Gmail** → _Authenticate email_ → pick the domain → **Generate new record** → publish the `google._domainkey` TXT → **Start authentication**. Give each domain its own SPF (`v=spf1 include:_spf.google.com ~all`) and `_dmarc` record too, starting at `p=none` with a reporting address and tightening after a week of reports.
- **App mail leaves from a subdomain, not the root.** Password resets and receipts should go through Resend, Postmark, or Cloudflare Email, sending from `mail.zestimer.com`. Reputation is tracked per sending domain, so a bounce spike on automated mail stays quarantined away from the address you answer humans from.
- **Third-party services register under the studio.** Apple Developer, Stripe, the bank, the App Store Connect account holder — all on `magicparklabs.com`, never personal Gmail. Changing the Apple ID on a live developer account later is genuinely painful. For per-product tooling logins, plus-tagging is enough (`pengyuc+zestimer-supabase@magicparklabs.com`), but treat it as _filing_, not as a security boundary.

One ceiling worth knowing up front: 30 aliases per user, which sounds infinite until you reflexively create `support@`, `hello@`, and `press@` on every domain. Pick one public address per product and one for the studio. And if you don't have a Workspace at all, Cloudflare Email Routing is the $0 version of this — same central inbox, with the tradeoff that you'll need an SMTP relay to reply as the product.

### Why the Two Halves Point Opposite Ways

Market as a person, operate as a company. It looks inconsistent, but it's the same principle applied to two different lifespans.

Attention accrues to people. A follower earned by a story about a rendering bug stays with you when the app that had the bug is sunset; a follower of `@zestimer` does not. Assets shouldn't work that way — domains, billing, developer accounts, and support history need to survive your interest in any given app, be handed to a collaborator, or be sold, and none of that works if they live in a personal Gmail with your recovery phone number attached.

One inbox, many masks. The masks are cheap, the inbox is the asset, and the person wearing them is the distribution channel.

---

<a id="zh"></a>

## 中文

我現在手上有三個產品——[Zestimer](https://zestimer.com)、[Furioke](https://furioke.com)、[Tefuda](https://playtefuda.com)——再加上一個工作室網域 `magicparklabs.com`，底下掛著一個 Google Workspace。四個網域、一個人、只有下班後的時間。

每次上線一個新產品，同一個直覺就會冒出來：**這個產品應該要有自己的一切**。自己的 X 帳號、自己的客服信箱、自己的 Stripe 和 Supabase 和 App Store Connect 登入。很像一間真的公司會做的事。

它對了一半，而錯的那一半很貴。我最後定下來的規則是：

> **產品要的是面具，不是帳號**。
> 幕後只有一個身分，幕前你想戴幾張臉都行。

這兩半指向**相反的方向**。台前，集中到**人**身上——也就是你。幕後，集中到**工作室**身上——一個能活得比你對任何單一 App 的熱情還久的主體。把這兩件事搞反，就是獨立開發者最後擁有六個收件匣、零個讀者的原因。

### 台前：用你自己的帳號做行銷

最誘人的做法是開一個 `@zestimer`、一個 `@playtefuda`，然後開始發產品更新。別這樣——至少不要一開始就這樣。

**一個新的品牌帳號是「零」了兩次**。零追蹤者是明顯的那個。真正會痛的是零**訊號**：一個剛註冊、整段歷史都是推廣連結的帳號，長得就跟垃圾帳號一模一樣，而 X、Threads、Reddit 的排序系統也就這樣對待它。你會對著一個沒人的房間發文好幾個月，然後得出「行銷沒有用」的結論。

你的本帳不在那個坑裡。它有歷史、有真實的互動、有追蹤關係圖，而且——這才是會複利的部分——它有資格去聊產品以外的東西。沒有人會訂閱一個一直說「v1.3.4 上線了」的帳號，但他們絕對會讀「這五行 diff 閘門終於讓我知道截圖到底有沒有變」。產品出現在故事裡面，而值得追蹤的是那個故事：轉換路徑比較長，觸及高一個數量級。

**而且這也是你唯一真的能持續做下去的版本**。一個人沒辦法同時寫程式、回客服，還餵飽三個平台上的三個品牌帳號。一個帳號撐得過正職一個很爛的星期，九個撐不住。

所以我的分工是：**本帳**是主要管道——開發日誌、吃掉一整個週六的那個 bug、偶爾幾個數字——bio 寫 `Maker of Zestimer, Furioke & Tefuda`。**產品 handle** 馬上註冊，所有我可能會想要的平台都註冊，然後放著不管，只留一段 bio 和一個指回我本帳的連結。這是防搶註的保險，成本是一個晚上。

品牌帳號要開始佔一個位子，得先有一份**不是說故事**的工作：狀態與故障公告、需要公開紀錄的客服，或是一個成員開始彼此對話（而不是只對著我說話）的社群。Tefuda 會比另外兩個先走到那一步，因為遊戲本來就會產生那種流量。在那份工作出現之前，品牌帳號只是一件沒有讀者的雜事。

我確實想過一個反駁：把一切綁在本帳上，如果哪天我想賣掉某個 App，不就把資產毀了嗎？不會。可以轉手的資產從來就不是追蹤數，而是網域、Workspace、App Store 紀錄、使用者、營收。透過本帳做分發的確是「租來的注意力」，但它租給的是**我**，而且免費跟著搬到下一個產品。

### 幕後：一個 Workspace，很多個網域

台前是個人的。幕後剛好相反——任何重要的東西都不該躺在 `myname@gmail.com` 裡。

這裡的失敗模式不戲劇化，只是慢慢地爛掉：一個產品開一個 Google 帳號。三個產品就是三組登入、三個密碼庫、三個驗證碼可能寄達的地方，以及三次在最要命的那個週末被鎖在帳單信外面的機會。

解法只有一句：**產品需要的是一個地址，不是一個帳號**。Google Workspace 是按**使用者**收費，不是按網域，所以一個席次就能名正言順地擁有你未來會買的每一個網域的郵件。

實際上要點哪裡——這是每篇教學都含糊帶過的部分：

1. **加網域**。Admin Console → 帳戶 → **網域** → _管理網域_ → **新增網域**。選**附加網域**（secondary domain），不要選網域別名，理由見下面那段。
2. **驗證**。Google 會給你一筆 TXT 記錄，發布在網域根層（apex），然後按_驗證_。如果註冊商和 DNS 都在 Cloudflare，這一步大概一分鐘。
3. **把 MX 指到 Google**。根層一筆就好：`smtp.google.com`，優先權 `1`。（舊的五筆 `aspmx.l.google.com` 還是能用，但新網域直接用單筆的那個。）記得把註冊商留下的停放 MX 刪掉，不然信會安安靜靜地跑去那裡。
4. **加別名**。目錄 → **使用者** → 你的帳號 → _使用者資訊_ → **電子郵件別名** → 加上 `support@newproduct.com`。不用新席次，不會多收錢。
5. **用產品的身分回信**。Gmail → 設定 → 帳戶 → _以其他地址發送郵件_ → 加入該地址，名稱寫 `NewProduct Support`，勾選**視為別名**（它是同一個帳號上的真別名，所以不需要另外接 SMTP），再把「回覆郵件時」設成**從接收信件的相同地址回覆**。

第 1 步那個分岔比看起來重要。**網域別名**（domain alias）會自動把現有的每個使用者名稱鏡射到新網域上——少打很多字，但某個產品的 `support@` 會變成跟其他所有產品的 `support@` 同一個信箱。**附加網域**讓每個產品保有自己的命名空間，只要你手上不只一個產品，你要的就是這個。無論選哪種：一個網域同時只能屬於一個 Workspace，所以在把 MX 指出去之前先決定好。

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

其中三列是真的會出錯的地方：

- **DKIM 是每個網域各一把（第 4 步）**。加了附加網域**不等於**它通過驗證——Google 是每個網域各生一把金鑰，而跳過這步就決定了你的回信會不會進垃圾桶。Admin Console → 應用程式 → Google Workspace → **Gmail** → _驗證電子郵件_ → 選那個網域 → **產生新記錄** → 發布 `google._domainkey` 的 TXT → **開始驗證**。SPF（`v=spf1 include:_spf.google.com ~all`）和 `_dmarc` 也要每個網域各設一份，從 `p=none` 加一個報告收件地址開始，看一週報告之後再收緊。
- **App 寄的信走子網域，不要走根網域**。密碼重設、收據這些自動信，該走 Resend、Postmark 或 Cloudflare Email，而且從 `mail.zestimer.com` 寄。信譽是以「寄件網域」為單位追蹤的，所以自動信件上的一波退信，會被隔離在你用來回覆真人的那個地址之外。
- **第三方服務一律掛在工作室名下**。Apple Developer、Stripe、銀行、App Store Connect 的帳號持有人——全部用 `magicparklabs.com`，絕不用個人 Gmail。事後要換掉一個已上線開發者帳號的 Apple ID 是真的很痛苦。至於各產品自己的工具登入，plus tagging 就夠了（`pengyuc+zestimer-supabase@magicparklabs.com`），但請把它當成**歸檔**，不是安全邊界。

有一個上限值得先知道：每個使用者 30 個別名。聽起來像無限，直到你反射性地在每個網域都開了 `support@`、`hello@`、`press@`。每個產品挑一個對外地址，工作室再一個就夠了。如果你根本沒有 Workspace，這一切的免費版是 Cloudflare Email Routing——一樣的集中收件匣，代價是你需要接一個 SMTP relay 才能用產品地址回信。

### 為什麼這兩半指向相反的方向

用個人的身分行銷、用公司的身分營運。看起來很不一致，但那其實是同一個原則套用在兩種不同的壽命上。

注意力累積在人身上。一個因為你講了某個渲染 bug 的故事而來的追蹤者，會在那個 App 收攤之後繼續留在你身邊；`@zestimer` 的追蹤者不會。資產不能這樣——網域、帳單、開發者帳號、客服歷史，需要活得比你對任何一個 App 的興趣還久，需要能交給合作者，或是能賣掉，而如果這些東西住在一個綁著你救援手機號碼的個人 Gmail 裡，那全部都做不到。

一個收件匣，很多張面具。面具很便宜，收件匣才是資產，而戴著面具的那個人，就是分發管道。
