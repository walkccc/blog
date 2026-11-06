---
name: syndicate
description:
  Draft platform-tuned social posts for a blog post — Threads (zh-TW), X
  (English + Japanese). Use when the user wants to share, promote, or syndicate
  a blog post to social media, or says "/syndicate".
---

# Syndicate a blog post

Turn one blog post into three platform-tuned social drafts, saved to
`social/<slug>.md` for review. The user pastes them manually; nothing is
auto-posted.

## Input

The argument is a path or slug of a post under `content/posts/`. If no argument
is given, use the post with the most recent `date` in its frontmatter.

## Steps

1. Read the post in full. Identify: the core problem, the "aha" insight, and one
   concrete detail (a code trick, a number, a gotcha) that makes the post worth
   clicking. Never invent claims that are not in the post.
2. Derive the canonical URL from the file path:
   `content/posts/<subpath>/<name>.md` →
   `https://blog.pengyuc.com/posts/<subpath>/<name>/` (Hugo slugifies the
   filename: lowercase, spaces → hyphens).
3. Write the three drafts following the platform rules below.
4. Verify character counts with the counter script (see "Counting"), fix any
   draft over its limit, then write `social/<slug>.md` using the output
   template. Include the measured counts.
5. Show the user the drafts and counts in your reply.

## Platform rules

### Threads — 繁體中文（台灣）

- Hard limit 500 characters; target 150–300.
- 台灣用語，不是中國用語：軟體、程式碼、載入、透過、資料（不是软件、代码、加载）。
- 語氣像工程師朋友在聊天：第一行直接講痛點或結論，不要「跟大家分享一篇文章」開場。
- 結構：hook（痛點／意外發現）→ 2–4 句核心洞見（可用短列表）→ 連結。
- 技術名詞保留英文（SwiftUI、zoom transition、snapshot…）。
- Hashtag 最多 1 個，通常 0 個。

### X — English

- Hard limit 280; any URL counts as 23 regardless of length. Target ≤ 260.
- Lead with the problem or a surprising fact — never "New blog post:".
- Concrete beats vague: name the API, the number, the failure mode.
- At most 2 hashtags, and only if genuinely discoverable (#SwiftUI yes, #coding
  no). Zero is fine.

### X — 日本語

- Same 280 weighted limit, but CJK characters count ×2 — effectively ~128
  Japanese characters once the URL (23) is included.
- Rewrite for Japanese tech-Twitter conventions, not a literal
  translation:「〜した話」「〜でハマった」 style hooks work well. Keep register
  consistent (casual だ・である or plain form is typical for engineer accounts).
- Technical terms stay in English.

## Counting

Run this to get all three counts (X counts are weighted: URL = 23, CJK ×2):

```bash
python3 - <<'EOF'
import re, unicodedata

def x_weighted(text):
    text = re.sub(r"https?://\S+", "\x00" * 23, text)
    return sum(2 if unicodedata.east_asian_width(c) in "WF" else 1 for c in text)

threads = """<PASTE THREADS DRAFT>"""
x_en = """<PASTE X EN DRAFT>"""
x_ja = """<PASTE X JA DRAFT>"""
print("threads:", len(threads), "/ 500")
print("x_en:   ", x_weighted(x_en), "/ 280")
print("x_ja:   ", x_weighted(x_ja), "/ 280")
EOF
```

## Output template

Write `social/<slug>.md`:

```markdown
---
post: content/posts/<...>.md
url: https://blog.pengyuc.com/posts/<...>/
drafted: <YYYY-MM-DD>
---

# <Post title>

## Threads (zh-TW) — <n>/500

> <draft>

- [ ] Posted

## X (EN) — <n>/280

> <draft>

- [ ] Posted

## X (JA) — <n>/280 weighted

> <draft>

- [ ] Posted
```

When the user says they posted one, check its box (optionally add the post URL
after the checkbox).
