+++
date = 2026-07-17T00:01:00-04:00
title = "2026 Is the Cheapest Year Yet to Start Indie Hacking"
tags = ["Indie", "AI"]
categories = ["Essay"]
+++

Between January and July this year, I shipped two apps by myself. If you had
told me that three years ago, I would have assumed you were describing a team.

[Zestimer](https://zestimer.com), started in January, is an interval workout
timer: a library of 212 illustrated exercises, an AI workout builder, a
paste-a-YouTube-link-get-a-circuit importer, an Apple Watch app, widgets, eight
languages, now at v1.3.4. [Furioke](https://furioke.com), started at the end of
May, is a sing-to-learn-Japanese app: it drives your own Spotify, Apple Music,
or YouTube account while showing synced lyrics with furigana and translations,
and turns long-pressed words into spaced-repetition flashcards. iOS plus a full
web app, five languages, on the App Store seven weeks after the first commit.

I still have a full-time engineering job. This post is about why, in 2026, that
sentence and the two above can coexist.

## The Fixed Cost of Building Collapsed

The obvious change is AI pairing. Both of my repos carry an `AGENTS.md` —
architecture rules, design-system constraints, tradeoff principles, all written
down — because most of my pair programming now happens with an AI agent, and
rules that live in a document get followed far more consistently than rules that
live in my head. One person maintaining a 34,000-line iOS app and a 41,000-line
web app would have been a joke a few years ago. Now it is a discipline problem,
not a headcount problem.

But "AI writes code for you" is the overtold half of the story. Three quieter
shifts mattered more to me.

**Infrastructure became Lego.** Zestimer's user data never touches a server I
operate — SwiftData syncs through CloudKit, and the phone, watch, and widgets
share one database. Its entire backend is a few hundred lines of Cloudflare
Worker whose only jobs are hiding API keys and capping AI spend. Furioke needed
a real backend, so it got Supabase and Next.js on Workers — zero machines to
manage from day one. Payments are StoreKit 2 and Stripe. Every one of these used
to be a "later, somehow" project. Now each is a brick you snap on.

**AI went from tool to ingredient.** Zestimer sends Gemini an actual YouTube
video and gets back a structured workout. Furioke uses Claude to translate
lyrics and explain grammar per word. A whole category of product that only
funded teams could build for the last decade is suddenly within one person's
reach.

**Language stopped being a wall.** Zestimer ships in eight languages — over
1,800 localized strings. Furioke launched with App Store pages in five. The old
indie default was "English market or nothing." Today localization is so cheap
that _not_ doing it is the strange choice.

## The Honest Part

To be clear: this is not a "side projects are easy money in 2026" post. There
will be no revenue screenshot, because revenue is not the point of this one.

The bottleneck did not disappear; it moved. The old wall was _can I build this?_
The new wall is _what should I build?_ — which feature to cut, when to ship,
which feedback to take, which to ignore. AI made typing cheap, but it will not
decide for you that a karaoke app should require iOS 26 because the whole
product identity leans on this year's APIs (I made that bet), or that a workout
timer should reach back to iOS 18 because a timer's job is to be on as many
phones as possible (I made the opposite bet, in the same year). Judgment is the
scarce resource now, and judgment only compounds one way: shipping things and
getting corrected by reality.

Distribution is not free either. Posts do not read themselves; apps do not
discover themselves. That is exactly why I am wiring this blog into Threads and
X — the channel is something you grow, not something you get.

## So Why Now

Because the cost of a learning loop has never been lower. The distance from idea
to App Store is short enough that you can make five years' worth of mistakes in
six months — and every one of those mistakes pays out in judgment, which
compounds whether or not the app ever makes a dollar. That return accrues to
you, not to the product.

If you have a day job and an idea you keep not starting, my conclusion is
simple: the toolchain has gotten cheap enough that not starting is now the
expensive choice. In 2026, the hours after work are enough.
