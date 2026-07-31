+++
date = 2026-07-30T00:00:00-04:00
title = "Three iOS Apps, One Kit: What Was Actually Shared and What Only Looked It"
tags = ["iOS", "Swift", "Git", "App Store Connect"]
categories = ["Engineering"]
+++

I ship three iOS apps out of three repos: **Tefuda**, a poker solitaire board; **Furioke**, a karaoke app for learning Japanese; and **Zestimer**, an interval timer. They were built in that order, and each one started life as a copy-paste of the last.

You know how this goes. The `run.sh` in Tefuda opens with a comment that says, in as many words, "Adapted from the Zestimer run.sh." The two `screenshots.sh` files share eighteen identically-named functions — `log`, `die`, `quiet`, `progress`, `resolve_device`, `verify_localizations`, `capture_scene` — across 1,580 lines that are nearly but not quite the same. Three `fastlane` setups carry the same hard-won comment about why screenshot uploads have to be serialized.

So I extracted the overlap into a shared repo. That part went as expected. The interesting part was discovering how much of the "duplication" was not duplication at all, and how much of the drift was silent.

## The drift nobody chose

Before writing a line of shared code, I diffed the three repos' config. None of these differences was a decision anyone made:

- **`.swiftformat`** was the same intent three times, except Furioke used SwiftFormat's modern hyphenated flag names (`--trim-whitespace`, `--property-types`) while the other two used the deprecated spellings — and Furioke alone set `--redundant-async tests-only`.
- **`.prettierrc`** disagreed three ways about `proseWrap` and `trailingComma`.
- Only **one of the three repos had a pre-commit hook at all**. The other two had `core.hooksPath` unset, so their formatters had never run. Not "ran and found nothing" — never ran.

And then the good one. Tefuda's hook, the one repo that had one:

```sh
for tool in swiftformat prettier; do
  command -v "$tool" >/dev/null || { echo "pre-commit: $tool not found" >&2; exit 1; }
done

swiftformat Tefuda
git add -A .
```

It checks that `prettier` is installed. It never runs it. The comment above those lines says "Both fixers, every time, rather than trusting everyone to remember," and the repo's `AGENTS.md` promises that prose wraps at 80 columns because the hook does it. One fixer ran. For months.

This is the actual argument for extracting shared infrastructure, and it isn't "less code." It's that **one copy gets read**. Three copies get skimmed.

## `Spacing.lg` is 10. Also 16. Also 16.

The plan said to share the design tokens. All three apps have `DesignSystem/Tokens/` with `Palette`, `Radii`, `Spacing`, `Typography`, `Motion`, `Materials`. Same file names, same idea, obviously shareable.

Then I opened them.

| token           | Tefuda | Furioke    | Zestimer |
| --------------- | ------ | ---------- | -------- |
| `Spacing.lg`    | 10     | (`l` = 16) | 16       |
| `Spacing.giant` | 32     | —          | 56       |
| `Radii.md`      | —      | 12         | 8        |

Three apps. Same token names. Different numbers. A shared `Spacing.swift` would have compiled perfectly and silently moved layout in three shipped apps — the kind of change that doesn't show up in a diff review because the diff is _deleting_ three files and adding one.

The fix is to notice that two different things were wearing one name. There is a **ladder** — the list of numbers, which is Tailwind's scale in all three apps and always was — and there are the **names**, which are each app's own vocabulary for which rung means what. The ladder is genuinely shared. The names are genuinely not.

So the kit ships the ladder:

```swift
enum Scale {
  static let s0_5: CGFloat = 2
  static let s1:   CGFloat = 4
  static let s1_5: CGFloat = 6
  static let s2:   CGFloat = 8
  // …
}
```

and each app's `Spacing` names rungs on it, keeping every value it had:

```swift
enum Spacing {
  /// gap-2.5
  static let lg: CGFloat = Scale.s2_5   // 10, exactly as before
  /// p-8
  static let giant: CGFloat = Scale.s8  // 32, exactly as before
}
```

Zero pixels moved. I verified that mechanically rather than by eye — parse both versions, resolve the `Scale.*` references back to numbers, compare:

```text
Spacing.swift: 11 tokens, IDENTICAL
Radii.swift:    7 tokens, IDENTICAL
```

What the apps gained is a shared vocabulary and a place where divergence is visible. What they didn't gain — deliberately — is each other's layout.

The same logic killed a token I had planned to share. Zestimer has an `Elevation` ladder, and it looked like an obvious candidate until I read it:

```swift
/// Z-height of a surface — currently flat by design.
///
/// The ink style has no z-axis: nothing floats, so nothing casts.
```

It's a no-op that returns `self`, and it references Zestimer's own `Palette` and `Radii`. Sharing it would have pushed "no shadows, ever" onto an app that has card and lift shadows on purpose. That's not a shared token; that's one app's art direction wearing a token's clothes. It stayed home.

## Git will not follow a symlinked `.gitignore`

The kit reaches each app repo as a `.kit` submodule, and most of what it shares is a symlink: `.swiftformat`, `.prettierrc`, `.githooks/pre-commit`, `AGENTS.core.md`, the skill file. One source of truth, nothing to keep in sync.

I checked that assumption instead of trusting it, which is how I found the exception. SwiftFormat and Prettier both resolve a symlinked config fine — they just `open()` it, and the kernel does the rest. Git executes a symlinked hook fine too. But:

```console
$ ln -s ../kit/gitignore.base .gitignore
$ git status --porcelain
warning: unable to access '.gitignore': Too many levels of symbolic links
?? .gitignore
?? ignored-by-kit.txt      # <- this was supposed to be ignored
?? kept.txt
```

Git refuses to follow a symlinked `.gitignore`. It's deliberate hardening, and the failure mode is the nasty kind: a warning you'll scroll past, and then **no ignore rules at all**. Build products, raw simulator captures, and — in a repo that keeps App Store credentials on disk — an API key, all quietly eligible for `git add -A`.

So `.gitignore` is the one file the kit copies instead of linking. It goes in between markers:

```gitignore
# === ios-kit base (managed) ===
…
# === end ios-kit base ===

# --- Furioke ---
# everything below the marker is the app's own, and the kit never touches it
```

and `kit doctor` — which the pre-commit hook runs — fails a commit if the managed block has drifted. A copy that can't drift is as good as a link. A copy that can drift silently is how you get three `.swiftformat` files again in a year.

## The harness is shared. The photograph isn't.

The biggest chunk of duplication was the screenshot pipeline. It's also where I was most tempted to over-share.

What every app genuinely does the same way: resolve a simulator name to a UDID on the newest runtime, create one extra simulator per parallel job, boot them, pin the status bar to 9:41, check that the built app is newer than the sources, shard the languages across devices, draw a progress line. That's the harness — 582 lines, written once. The two `screenshots.sh` files it replaced were 646 and 396 lines, and what each repo keeps now is a 155- and a 141-line `scenes.sh` that is entirely about that app's own screens.

What no two apps do the same way, at all:

- **Tefuda** decides a scene is ready by snapping until two consecutive frames are byte-identical. It can do that because the simulator's compositor is deterministic — a screen nobody is animating renders to the same bytes every time — so a scene held until it agrees with itself will come back identical tomorrow.
- **Furioke** waits on a fixed floor, because its scenes are waiting for a Japanese tokenizer's cold furigana pass and a keyboard to raise. It also has to list `ja` as a _secondary_ `-AppleLanguages` entry in every non-Japanese capture, purely to make the Japanese keyboard available for the app to then select — otherwise English raises its slide-to-type intro over the card, and Traditional Chinese raises Zhuyin.
- **Zestimer** compares each fresh capture against the previous run's with a per-pixel tolerance, because its glass materials dither between renders.

Three different definitions of "the screen has settled," each correct for its own app. So the kit's harness calls a `capture_scene` that the _repo_ defines, and knows nothing about scenes, settling, or what a screenshot is of. The contract is four lines in each repo's `scripts/scenes.sh`.

The rule I ended up with: **share the thing that has no opinion.** Device resolution has no opinion. "Is this screen done animating" is nothing but opinion.

## Dropping Ruby for a Go binary

All three repos used fastlane to push App Store metadata and screenshots. Not to upload builds — those go from Xcode's Organizer — just the listing. I'd been eyeing [`asc`](https://github.com/rorkai/App-Store-Connect-CLI), a single Go binary for the App Store Connect API, and the question was whether it could replace fastlane completely rather than sit beside it.

Mostly yes, and the wins were not the ones I expected.

**Screenshots needed no file moves.** `asc screenshots upload --path fastlane/screenshots --device-type IPHONE_65 --replace` fans out over locale subdirectories, which is exactly the layout `deliver` already used. It also deleted two workarounds that had been carefully commented into all three Fastfiles:

```ruby
# ASC 500s the POST that reserves an upload slot when several screenshots
# land on one appScreenshotSet at once. deliver enqueues jobs in
# locale → set → image order, so its worker threads all attack the same
# set — the thread count IS the width of the race.
ENV["DELIVER_NUMBER_OF_THREADS"] = "1"
ENV["SPACESHIP_SCREENSHOT_UPLOAD_TIMEOUT"] = "1"
```

Both gone, because `asc` uploads a set as one job. Two hard-won comments retired is a better outcome than two hard-won comments deduplicated.

**Metadata needed a format conversion.** `asc` keeps canonical JSON (`app-info/<locale>.json`, `version/<version>/<locale>.json`) rather than fastlane's `<locale>/name.txt` tree, and the `migrate` command its docs mention doesn't exist in 3.3.0. So I wrote the converter. The part worth stealing is how I checked it: convert the tracked tree, then `asc metadata pull` the **live** listing and diff the two field by field.

```text
identical=41  differ=4  live-only=0  repo-only=0
```

The four that differed were the localized app names from my two most recent commits — real local work that hadn't been pushed yet. That's about as good a validation as a converter gets: everything matches except the things I already knew didn't.

Then I ran the same check on Tefuda and it came back seven fields apart, in the other direction:

```text
DIFFERS  version/ja.description
    live: …決めるのは三つの手札のどれに置くかだけ。置いた札はもう動かせません。手札が揃ったら…
    repo: …決めるのは三つの手札のどれに置くかだけ。手札が揃ったら換金…
```

The **store** had a sentence the repo didn't — in all five locales. At some point I'd tightened that copy in App Store Connect's web UI and never brought it home, so the tracked tree had been quietly stale for months. Converting it faithfully and pushing it would have reverted live copy to an older draft, in five languages, and the diff would have looked like a successful migration the whole way.

Tefuda's own release notes have said "pull the live listing first — always, even for a one-word edit" for as long as that repo has had a release process. I'd written that rule. I still nearly walked into it, because "convert the tracked tree" _feels_ like a mechanical, lossless operation, and mechanical lossless operations don't feel like they need a source of truth. So Tefuda's new `metadata/` is seeded from the live pull rather than from the repo, plus the five `whatsNew` fields the store legitimately doesn't have — Apple shows no What's New on a first version.

Two apps, the same conversion, opposite answers about which side was stale. The converter was right both times. The only thing that told me which direction to trust was asking the store.

**Two things `asc metadata` doesn't cover**, and this turned out to be a feature. Categories, review information, age ratings and export compliance live in separate commands (`asc categories`, `asc review`, `asc age-rating`, `asc encryption`) rather than in the metadata tree. Under fastlane they rode along with every push, which is how one of these repos learned that an empty `review_information/` directory kills a delivery — _after_ every locale's copy has already uploaded. Set-once-and-forget is the correct shape for a field you set once and forget.

And `asc` plans before it applies:

```text
┌────────┬───────────────────────┬─────────┬────────────────────────────────┬────────────────────────┐
│ change │          key          │ locale  │              from              │           to           │
├────────┼───────────────────────┼─────────┼────────────────────────────────┼────────────────────────┤
│ update │ app-info:ja:name      │ ja      │ Furioke: Sing & Learn Japanese │ Furioke：カラオケで学ぶ日本語 │
│ update │ app-info:zh-Hant:name │ zh-Hant │ Furioke: Sing & Learn Japanese │ Furioke：K歌學日文        │
└────────┴───────────────────────┴─────────┴────────────────────────────────┴────────────────────────┘
```

`deliver` had no equivalent. The first sign a locale was about to be overwritten was that it had been.

One more thing fell out of the migration that I hadn't gone looking for: all three repos kept the _same_ App Store Connect private key, in plaintext, in three gitignored `fastlane/api_key.json` files. `asc auth login` puts one entry in the login keychain and serves all three. Three copies of a private key on disk became zero.

## The migration modifies only what it's migrating

A hook I wrote in the morning nearly caused the problem it exists to prevent.

The kit's pre-commit hook formats and re-stages. My first version did what Tefuda's did — `swiftformat $SOURCE_DIRS`, the whole tree. Then I went to commit the first migration and SwiftFormat wanted to reformat exactly one file, which happened to be in the middle of four uncommitted, half-finished changes that were sitting in that repo when I started.

The hook was about to rewrite somebody's work-in-progress as a side effect of a commit that touched shell scripts. It's obvious once you see it, and completely invisible until you have an unrelated edit in the tree at the wrong moment. The hook now formats only staged files, and skips symlinks so Prettier can't write back through a link into the kit.

Which is a specific case of the general rule I kept bumping into: **a commit may only ever change what it is committing.** The same instinct is why the `.gitignore` managed block stops at a marker, and why `kit sync` moves anything it displaces to `<name>.pre-kit` rather than overwriting it.

## All three, and the one that had to wait a few hours

Tefuda and Furioke went first. Furioke's `scripts/` went from 870 lines to 200, its `AGENTS.md` from 115 to 74. Tefuda's four scripts became four-line shims, and its `AGENTS.md` shed its boilerplate tail while keeping the 600 lines that are actually earned knowledge about that specific board.

Zestimer nearly didn't go at all. When I got to it, `git status` said **475 modified files** — it was mid-redesign, and the files in flight included the very token files the migration rewrites. So I backed the submodule out and left it alone, which felt like the responsible answer.

It was the wrong one, or at least an incomplete one. The redesign wasn't finished and wasn't in a hurry; `git stash` is one command and gives back exactly the tree the migration needs, with the work recoverable by another. "This repo is busy" was a reason to ask, not a reason to stop. Once the tree was clean the migration took twenty minutes.

Zestimer is also the app that justified the harness/photograph split retroactively. It shoots **three** surfaces — iPhone, Apple Watch, and widgets that aren't photographed at all (the app renders them into its own container and a script copies them out). The kit's harness shards _one_ app across languages on _one_ device type. Rather than grow it a watch-shaped hook, its iPhone pass goes through the harness — where sharding actually pays, at ten locales — and the other two stay a repo script. The kit got exactly one new hook out of the whole exercise, `rotate_language`, because Zestimer compares each capture against the previous run's and needed the old set moved aside rather than deleted.

And the last bug the migration found was mine. I'd written `DEVICE_TYPE=IPHONE_65` into all three configs. `IPHONE_65` is 1242×2688; Tefuda's cards are 1320×2868, which is `IPHONE_69`. Two apps were right by accident and one was pointed at a screenshot set its images don't belong to. Zestimer then needed the field to be a _list_, since its 416×496 watch captures sit in the same locale folder as its phone cards and App Store Connect files each by pixel size.

That is three separate mistakes — a hook that formats too much, a token that shouldn't be shared, a display size copied without checking — all of the same species. Each one is what happens when something that _looks_ uniform is assumed to be uniform. Which is the whole subject: the work of sharing code is almost entirely the work of finding out what isn't shared.

The kit is versioned and pinned per repo, so the three still don't have to move together. They just happen to have, this time.
