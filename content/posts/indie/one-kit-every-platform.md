+++
date = 2026-07-30T18:00:00-04:00
title = "One Kit, Every Platform: Deleting the Browser That Drew My App Store Screenshots"
tags = ["iOS", "Android", "Swift", "CI", "App Store Connect", "Google Play"]
categories = ["Engineering"]
+++

A few weeks ago I extracted the overlap of three iOS repos into a shared submodule and [wrote about what I found]({{< ref "one-kit-three-ios-apps" >}}) — mostly that the "duplication" wasn't duplication, and the drift was silent.

Then I shipped an Android app, and the kit was called `ios-kit`.

This is what happened next: the kit became platform-neutral, the store screenshots stopped being rendered in a browser, and `fastlane` left the building. The interesting parts were, again, not the parts I planned.

## The thing I couldn't stop apologising for

Here is a comment from the Android repo's publish script, written by me, months before any of this:

```sh
# The Play Console locale folders appshapr renders into — one per caption language
# in its `furioke-android:export --languages` list, and the same five the app itself
# ships. Named here so a locale added there but not here fails loudly below rather
# than shipping a listing that is quietly missing a language.
LOCALES=(en-US ja-JP zh-TW zh-CN ko-KR)
```

And here is one from the iOS repo, three files away:

```sh
# The App Store Connect locale folders AppShapr renders into — one per caption
# language in its `furioke:export --languages` list. These are STORE locales and
# deliberately differ from kit.env's CAPTURE_LANGUAGES: `en` is captured,
# `en-US` is shipped.
STORE_LOCALES=(en-US ja zh-Hant zh-Hans ko)
```

One app. Its languages were spelled out **four times**: the capture codes in the shell config, the store codes in the render script, a third list in the uploader, and a fourth in the caption file that lived in a different repo entirely. Three of the four carried a comment warning that the other three existed.

Those comments are the tell. They're not documentation; they're a guard rail, in prose, around a duplication that didn't have to exist. Nobody could change a language in one place and know they were finished.

A locale is one row now:

```json
{ "capture": "zh-Hant", "appStore": "zh-Hant", "play": "zh-TW" }
```

`capture` drives the app and names the folder a raw capture lands in. The rest are whatever each store insists on calling it. A store the app isn't on has no key, and its publisher never sees the row. The capture harness fans out over `capture`, the renderer writes a folder per row, and each uploader renames on the way out.

That single change removed more lines than anything else I did, and it's the one I'd port to any codebase.

## The browser that wouldn't hold still

Three of the apps composed their App Store cards in a separate Next.js app: storyboards as TypeScript objects, layout in React, export by Playwright driving headless Chromium at each store's pixel size.

It made good-looking cards. It had one fatal property: **two exports of an unchanged storyboard came back different.**

Chromium's rasterisation isn't byte-stable across runs, so the tracked PNGs were permanently dirty and `git status -- screenshots` stopped answering "did any screen change" — which is the question the entire pipeline exists to answer. A store screenshot should be reviewed in a diff. If every screenshot is always in the diff, none of them is.

Every app that used it had independently grown the same workaround. Zestimer's, in Python:

```python
old = numpy.asarray(Image.open(sys.argv[1]).convert("RGB"), dtype=numpy.int16)
new = numpy.asarray(Image.open(sys.argv[2]).convert("RGB"), dtype=numpy.int16)
changed = int((numpy.abs(old - new).sum(axis=2) > 32).sum())
sys.exit(0 if changed < 48 else 1)
```

Tefuda's, in Swift, with a per-channel ceiling instead of a sum. Two hand-tuned tolerances, in two languages, both papering over the renderer rather than fixing it.

So I wrote the renderer. `compose.swift` — 745 lines of CoreGraphics and CoreText, driven by a JSON storyboard that lives in the repo that ships the app.

```sh
$ scripts/kit render --locale en && shasum -a 256 store/screenshots/en-US/*.png > /tmp/a
$ scripts/kit render --locale en && shasum -a 256 store/screenshots/en-US/*.png | diff /tmp/a -
$ echo $?
0
```

Same storyboard, same captures, same bytes. The tolerance gate survives — once, in the kit — but it now catches what it was always for: a scene that was still moving when it was photographed. A run that puts _any_ card back is a bug to chase, not a number to raise.

Three other things fell out of doing it in-process:

**CJK stops being a special case.** CoreText is the only thing on the machine that lays out Japanese and Korean beside Latin without being handed a font per script — but only if you give it an explicit cascade. The implicit fallback for Han characters depends on the system's language order, so "it looks right on my laptop" is not a property of the card. Each face names its own:

```json
"cjk": {
  "ja": "HiraginoSans-W6",
  "zh-Hant": "PingFangTC-Semibold",
  "zh-Hans": "PingFangSC-Semibold",
  "ko": "AppleSDGothicNeo-Bold"
}
```

`YouTube・Spotify・Apple Music 対応。` shapes as one line, Latin in Poppins and the rest in Hiragino, and does it the same way on any Mac.

**The device is drawn, not photographed.** The old renderer composited a photoreal bezel PNG per model per colour. Mine is four numbers — screen aspect, corner radius, bezel width, colour — and a rounded rect. It's exact at any card size, which a PNG never is, and there's no binary asset to keep in step.

It also read as a black rectangle until I added two lines:

```swift
if let rim = model.rim {
  let hairline = max(width * 0.004, 1)
  …
}
```

A hairline just inside the body. That's what a photographed bezel was actually giving me, and it costs nothing.

**A spanning pair lines up exactly.** Two of Furioke's cards carry one pair of tilted phones across both. The old renderer laid out each card separately and hoped the halves met. Mine renders the pair once, on a canvas two cards wide, and each card takes its half — so they line up to the pixel rather than to within a rounding of a rotation.

## What I didn't absorb

Tefuda's cards aren't composed; they're **drawn**. Its title card is a ladder of three poker hands, each one wider and lit harder than the last, with card art ported curve for curve from the app, a 青海波 pattern in the cloth, and a rule about where gold is allowed to appear. That's 1,500 lines of CoreGraphics that happen to output store screenshots.

I could have expressed it as a storyboard. It would have taken a week and produced a worse title card and a worse renderer.

So `kit.json` has an escape hatch:

```json
"render": { "command": "scripts/render-cards.sh" }
```

The kit still owns both ends of that pipeline — the capture, the same-picture gate, the upload — which is most of the value with none of the pretence. It's the same lesson as the `Spacing.lg` problem from last time, wearing a different costume: **a layout and an illustration are two different things sharing one output format.** Forcing one through the other compiles fine and is wrong.

The validation was cheap, and worth doing: after the migration, 36 of Tefuda's 50 cards re-rendered byte-identical. The 14 that didn't were the ones whose raw captures on disk were newer than the last commit — a real screenshot change, not a pipeline regression.

## The Fastfile was longer than what replaced it

The Android app shipped through `fastlane supply`. The Fastfile that configured it: 141 lines. `store/play.sh`, which does the whole job — auth, listing copy, imagery, the AAB, the track — is 182, and about 40 of those are comments explaining Play's behaviour.

The auth is the part people assume is hard:

```sh
header="$(printf '{"alg":"RS256","typ":"JWT"}' | base64url)"
claims="$(printf '{"iss":"%s","scope":"…","aud":"…","exp":%d,"iat":%d}' …)"
signature="$(printf '%s.%s' "$header" "$claims" |
  openssl dgst -sha256 -sign "$pem" -binary | base64url)"
```

A service-account JWT is three base64 segments and one `openssl` call. There is nothing to install; `openssl` is on every machine that has git.

But the real reason to leave wasn't the line count. It's this, from the Fastfile I deleted:

```ruby
# Refuse to upload imagery the pipeline hasn't staged. supply replaces a screenshot
# type wholesale — for every type it finds local files for, it deletes *all* of that
# type on the listing and uploads what's on disk. So a half-staged run — a render
# that died after two cards — doesn't fail, it quietly cuts the store listing down
# to two screenshots.
```

I'd written a guard, in Ruby, against a tool that had no transaction. Play has one — an *edit*: insert, change, commit. Nothing is visible until it commits, and a run that dies halfway leaves the listing untouched. `supply` simply didn't use it that way. Twelve lines of defensive Ruby became a property of the API I was already talking to.

On the App Store side, `asc` had already replaced `deliver` — one Go binary, one keychain entry for the whole account, and it *plans before it applies*. That migration is [the previous post]({{< ref "one-kit-three-ios-apps" >}}).

Both stores, no Ruby, no browser, no npm.

## Making the kit stop being iOS-shaped

The obvious move for an Android app is a second kit. `simctl` and `adb` have nothing to say to each other, after all.

That's true of the last inch and false of everything above it. What the two pipelines actually share, once I measured instead of assuming: the scene × language loop and its sharding, the shutter, the locale table, the "did any screen change" gate, the card composer and every one of its layout rules, the metadata model and its character limits, the house style, the hooks, the skills, the CI workflow.

The shutter is the tell. This function existed twice — once against `simctl`, once against `adb` — with the same paragraph of reasoning in both comments:

> The app never says it has settled, so the shutter asks the screen instead. Two identical frames is a sound stillness signal because both capture paths are DETERMINISTIC: a screen nobody is animating renders to the same bytes frame after frame, launch after launch, run after run.

So instead of a second kit, every device capability became a `platform_*` function, with `platform/{ios,android,web}.sh` each defining all of them. The rule with teeth is that **no command contains `case $PLATFORM`**. Writing one means a capability is missing from the contract — not that this platform is special.

The differences that survived are the real ones. iOS creates a simulator per language and shoots in parallel; Android has one emulator, so `platform_workers` hands back one device and sets `JOBS=1`, and the same sharding code runs serially. No second harness.

And Android has one thing iOS doesn't:

```sh
# Is the system's "isn't responding" dialog on the glass? It has to be asked,
# because every other check says yes: the scene stages, the app says so, the
# screen settles — and it settles *because* the dialog is up and nothing behind
# it is moving.
android_is_anr() { … }
```

An ANR dialog is perfectly still. It passes the two-identical-frames test with flying colours.

I also wrote a `platform/web.sh`, mostly to keep the contract honest. "The device" is a URL, "the app" is a dev server, `platform_screenshot` is headless Chrome, and three functions are no-ops with a line each saying why. It's 94 lines and it proves the interface isn't a phone in disguise.

## The mistake I made twice, in the same week

I added a shared `.editorconfig` to the kit and linked it into every repo. Tidy. Obviously shared. What could an editor config possibly break?

The Android repo's own file, which the link replaced:

```ini
[*.{kt,kts}]
indent_size = 2
ktlint_code_style = ktlint_official

# Composables are PascalCase by convention; ktlint's function-naming rule would
# otherwise reject every screen in the app.
ktlint_function_naming_ignore_when_annotated_with = Composable
```

The kit's said `indent_size = 4` and carried no ktlint rules at all. Linking it would have re-indented every Kotlin file in the repo and made ktlint reject every `@Composable` in the app.

This is *exactly* the `Spacing.lg` bug from the last post — a shared file that compiles everywhere and is silently wrong in one place — and I walked into it again, eight weeks later, with the earlier version written down in the repo's own README. Knowing the shape of a mistake apparently doesn't stop you making it in a new costume; it just makes you recognise it faster.

`.editorconfig` isn't linked any more. It ships in `config/` as a starting point to copy.

The second one was gentler. I keyed the whole metadata tree by capture language, which is beautiful and consistent and dropped a file: Zestimer ships Spanish to **two** App Store listings, `es-ES` and `es-MX`, from one set of screenshots — and their copy is genuinely different. Mexico says *video* and *celular* where Spain says *vídeo* and *móvil*. Somebody had written that, once, carefully.

My converter renamed `es-ES.json` to `es.json` and then printed:

```text
es-MX.json -> es.json already there, dropping the duplicate
```

It wasn't a duplicate. Nothing about the resulting tree would have looked wrong — the file simply wouldn't be there any more, and the next push would have shipped Spain's wording to Mexico.

The fix is small and, in hindsight, obviously right: the tree is keyed by capture language, and **a file named for a store locale wins for that row**. `es.json` is the shared Spanish; `es-MX.json` overrides it. Share what's genuinely the same, name what's genuinely different.

Which is the same sentence as the `.editorconfig` one, and the same sentence as `Spacing.lg`. Three years of this and the job is still, mostly, finding out what isn't shared.

## What it looks like now

Every repo, whatever it's written in:

```text
kit.json            the manifest — platform, locales, scenes, store ids
scripts/kit         the one entry point
scripts/scenes.sh   what a store screenshot IS, for this app
store/cards.json    the storyboard and the copy
store/metadata/     the listing text, one tree, every store
store/screenshots/  the finished cards — tracked
.kit/               the shared half
```

Five commands, identical across iOS, Android and web:

```sh
scripts/kit doctor
scripts/kit capture --skip-build
scripts/kit render
scripts/kit publish copy
scripts/kit publish shots
```

The Android migration alone was **1,040 lines deleted, 652 added**, and most of what was added is a JSON storyboard holding words that used to live in another repo.

Three external dependencies are gone: a browser, a Ruby gem, and a Next.js app that had to be running for me to change a caption. What's left is `bash`, `python3`, `swiftc`, `curl` and `openssl` — five things that were already on the machine.

CI, meanwhile, deliberately doesn't build. It runs `kit doctor`, checks the formatters left nothing to do, and asserts that every card has a caption in every language the manifest declares. Building on a runner is minutes of compute for something the IDE already did on the machine that made the commit; the failures worth catching in CI are the ones a person can't see — a symlink that went stale, a copied token file that drifted, a hook that never ran.

## The part I'd tell past me

The renderer was the big scary piece — a browser replaced by 745 lines of CoreGraphics — and it took an afternoon, because compositing an image onto a background under some text is not hard. What took the rest of the week was the small stuff: finding four spellings of one locale, noticing that a metadata file had gone missing, realising that a shared editor config isn't shared.

**The hard part of consolidation is never writing the shared thing. It's deciding what "the same" means**, over and over, in cases where the answer looks obvious and isn't.
