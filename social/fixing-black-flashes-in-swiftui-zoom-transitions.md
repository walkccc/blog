---
post: content/posts/fixing-black-flashes-in-swiftui-zoom-transitions.md
url: https://blog.pengyuc.com/posts/fixing-black-flashes-in-swiftui-zoom-transitions/
drafted: 2026-07-17
---

# Fixing Black Flashes in SwiftUI Zoom Transitions

## Threads (zh-TW) — 401/500

> SwiftUI 的 zoom
> transition 有個很惱人的 bug：dismiss 的瞬間，底下的卡片會閃一格全黑。
>
> Debug 完發現是三個原因疊在一起：
>
> 1. matched
>    source 標在整個 row 上，row 大多是透明的，snapshot 直接透到黑色的 presentation
>    layer
> 2. Liquid Glass 在 dismiss 時 resample 到錯誤的 backdrop
> 3. .presentationBackground 來得不夠早
>
> 解法：source 用有實色的小 thumbnail、transition 附近不要用 glass、host 自己先畫背景。
>
> 完整 debug 筆記：
> https://blog.pengyuc.com/posts/fixing-black-flashes-in-swiftui-zoom-transitions/

- [ ] Posted

## X (EN) — 269/280

> SwiftUI zoom transitions flashing black on dismiss? Three causes stack up: a
> transparent matched source, Liquid Glass resampling the wrong backdrop
> mid-transition, and .presentationBackground applied too late.
>
> Fix pattern + debugging checklist:
> https://blog.pengyuc.com/posts/fixing-black-flashes-in-swiftui-zoom-transitions/

- [ ] Posted

## X (JA) — 209/280 weighted

> SwiftUIのzoom
> transitionがdismiss時に一瞬黒くなるバグ、原因は3つ重なってた：透明なmatched
> source、Liquid
> Glassのbackdrop誤サンプリング、presentationBackgroundのタイミング。デバッグメモ→
> https://blog.pengyuc.com/posts/fixing-black-flashes-in-swiftui-zoom-transitions/

- [ ] Posted
