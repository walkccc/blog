+++
date = 2026-07-17T00:00:00-04:00
title = "Two Apps, One Philosophy: the Stacks Behind Zestimer and Furioke"
tags = ["Swift", "SwiftUI", "Next.js", "Indie"]
categories = ["Engineering"]
+++

This year I shipped two indie apps: [Zestimer](https://zestimer.com), an
interval workout timer I started in January, and [Furioke](https://furioke.com),
a karaoke lyrics companion for learning Japanese that I started at the end of
May. Different products, same person, and — it turns out — mostly the same
opinions.

This post is about the stack and architecture choices behind both: what I
deliberately kept identical, what I deliberately made different, and the
decisions I would repeat.

## The Apps, Briefly

Zestimer is a native iOS HIIT/interval timer: build circuits, run them
full-screen with voice cues and haptics, track streaks. It ships 212 illustrated
exercises, an AI workout builder, a YouTube-to-workout importer, widgets, and a
standalone Apple Watch app.

Furioke turns songs you already stream — on Spotify, Apple Music, or YouTube —
into Japanese lessons: synced lyrics with furigana over kanji, toggleable
romaji, translations, and long-press-to-save vocabulary that comes back as
spaced-repetition flashcards.

## Language Choices

The split is boring on purpose: **Swift for anything that runs on an Apple
device, TypeScript for everything else.**

Both apps are fully native SwiftUI. For Zestimer this was non-negotiable: a
timer lives or dies on audio latency, background behavior, and haptics, and I
also wanted a watch app and widgets, which cross-platform frameworks still treat
as an afterthought. For Furioke the pull was different: the entire product
identity leans on new system APIs, and native is the only place you get them on
day one.

Everything server-side is TypeScript. Zestimer's backend is a small Cloudflare
Worker. Furioke's backend is a full Next.js app deployed to Cloudflare Workers
via OpenNext, which doubles as the web app and the API for iOS. One brain, two
languages, no third.

## What Is Deliberately the Same

**SwiftUI without view models.** Neither app has an MVVM layer. SwiftData models
are the source of truth, views observe them, `@State` is for ephemeral UI, and
domain logic lives in model extensions or small focused services. Every time I
have been tempted to add an architecture layer, deleting the temptation was the
better call.

**Design tokens over ad-hoc styling.** Both apps route all styling through a
token layer — palette, spacing, radii, typography, motion. It is the reason a
one-person project can survive a redesign.

**Ship-fast rules, written down.** Both repos commit straight to `main`. No side
branches, no PRs, no CI gate; the quality bar is a local hook and the discipline
in each repo's `AGENTS.md` — which exists because my pair programmer these days
is an AI agent, and rules that live in a doc get followed more consistently than
rules that live in my head.

**No test targets, by choice.** Controversial, but honest: at this scale the
scarcest resource is iteration speed, and the failure modes I actually hit are
design mistakes, not regressions a unit test would have caught. I revisit this
tradeoff every time an app grows; so far, shipping has won.

## What Is Deliberately Different

The backends could not be more different, and that is the part I would defend
hardest: **the backend should be as big as the product's trust problem, and no
bigger.**

Zestimer's user data never touches a server I run. SwiftData syncs through
CloudKit; the phone, the watch, and the widgets share one App Group store, and
there is no WatchConnectivity code at all — workouts appear on the watch because
the database syncs, not because I move messages around. The only server
component is a ~700-line Cloudflare Worker whose entire job is to keep API keys
out of the binary and cap AI spend: App Check attestation on every request, and
KV-backed daily quotas — per-user and account-wide — standing between the
internet and my Gemini bill.

Furioke needed the opposite. Cross-platform accounts, a community layer, lyrics
and translations cached server-side, and entitlements that can arrive from
StoreKit, Apple server notifications, Google Play, or Stripe — that is a real
backend, so it got a real one: Supabase for Postgres and auth, Next.js API
routes for everything else, one unified entitlement endpoint so every client
asks the same question the same way.

## Three Decisions I Would Repeat

**Derive time, never accumulate it.** Zestimer's timer never counts ticks. It
anchors a start date and recomputes elapsed time from the wall clock on every
tick, so drift cannot accumulate, and phase cues fire on boundary crossings so a
jittery tick cannot skip a beep. The web timer on zestimer.com reimplements the
same engine in TypeScript — a throttled background tab resynchronizes to true
time the moment it wakes. Same problem, same fix, two languages.

**Don't host audio; orchestrate it.** Furioke never streams a single song.
Playback is delegated to the user's own Spotify, Apple Music, or YouTube account
behind one provider protocol, and feature code branches on a _capability_ —
"does this provider require visible video?" — never on the provider's name. That
one abstraction is simultaneously the licensing strategy, the cost model, and
the reason a solo developer can offer a catalog of everything.

**Run the same code where consistency matters.** Furigana segmentation must be
identical on web and iOS, so instead of porting the tokenizer, the iOS app runs
the same kuromoji JavaScript inside JavaScriptCore, fed by the same dictionary
from bundled resources. The flashcard scheduler is ported line by line between
TypeScript and Swift with a comment binding the two. Boring beats clever when
the alternative is two implementations drifting apart.

## The OS-Floor Bet

One choice splits the two apps cleanly. Zestimer targets iOS 18 — a timer should
reach as many phones as possible. Furioke requires iOS 26 — it leans on this
year's SwiftUI, SwiftData, and Liquid Glass, and supporting older systems would
have doubled the design surface for an audience that mostly is not there yet.

As a solo developer you only get so much novelty budget. Spend it where the
product actually differentiates, and let everything else be boring.

## The Takeaway

Two apps taught me the same lesson from opposite directions: almost every
architectural question resolves to _what is the smallest thing that keeps the
product trustworthy?_ CloudKit when the data is personal, Postgres when the data
is shared. No view models until a view earns one. No second implementation of
anything that must agree with itself.

The stack is not the product. The stack is whatever lets one person keep
shipping the product.
