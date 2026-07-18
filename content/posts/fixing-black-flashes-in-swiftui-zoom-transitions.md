+++
date = 2026-07-02T00:00:00-04:00
title = "Fixing Black Flashes in SwiftUI Zoom Transitions"
tags = ["SwiftUI", "iOS", "Animation"]
categories = ["Engineering"]
+++

SwiftUI's zoom navigation transition can look wonderful when it works: a card or
thumbnail stays visually connected to the detail screen, and dismissal feels
like the screen knows exactly where it came from.

The failure mode is just as memorable. During the first or last frame of the
transition, the source card suddenly turns black. Then, a beat later, it
resolves back to the app background or card color.

This post is the debugging note I wish I had started with.

## The Symptom

I had a workout app with a few related transitions:

- An exercise thumbnail opens a full-screen exercise detail view.
- A workout history row opens a full-screen workout recap.
- Both use SwiftUI's zoom transition through `matchedTransitionSource` and
  `navigationTransition(.zoom(...))`.

Opening usually looked fine. Dismissing was worse: the card underneath the
detail view appeared as an all-black rectangle for a frame before turning back
into the intended surface color.

Adding `.presentationBackground(appBackground)` helped in a few places, but it
did not fully fix dismissal. That was the clue: the black frame was not only the
presented cover's background.

## What Was Actually Happening

There were three overlapping causes.

First, the matched source was too large and too transparent. I had marked an
entire row as the zoom source:

```swift
SessionRow(session: session)
  .cardTransitionSource(id: session.id, in: namespace)
```

Rows are often mostly transparent. They contain text, thumbnails, spacing, and
whatever card or list background happens to sit behind them. During a
transition, SwiftUI snapshots that source. If the source has no stable fill of
its own, the snapshot can expose whatever the presentation system has underneath
at that instant.

Sometimes that underneath layer is black.

Second, Liquid Glass content surfaces can resample the wrong backdrop during a
full-screen cover transition. A glass card may look correct at rest, but while a
zooming cover is being dismissed, its backdrop can momentarily be the
presentation host instead of the app page.

Third, `.presentationBackground(...)` is not always early enough. The view
receiving `.navigationTransition(.zoom(...))` may be snapshotted before
presentation chrome and backgrounds have fully settled. The transition host
itself needs to paint stable pixels.

## The Fix Pattern

The reliable pattern was:

1. Use a small, painted source for the matched transition.
2. Avoid glass for content cards that sit behind zoom dismissals.
3. Paint the presented transition host, not only the presentation background.

### 1. Make the Thumbnail the Source

Instead of marking a whole row as the source, attach the transition to the
thumbnail tile:

```swift
ExerciseThumbnailView(...)
  .padding(Spacing.xs)
  .frame(width: 52, height: 52)
  .background(Palette.secondary.opacity(Opacity.faint))
  .clipShape(RoundedRectangle(cornerRadius: Radii.xs, style: .continuous))
  .matchedTransitionSource(id: id, in: namespace) { source in
    source.clipShape(RoundedRectangle(cornerRadius: Radii.xs, style: .continuous))
  }
```

The important part is not the exact styling. The important part is that the
matched source is a view with its own non-transparent fill.

For my workout rows, the detail now grows from the exercise thumbnail rather
than from an entire text row. For history rows, the recap grows from the
workout's leading exercise thumbnail.

### 2. Keep Behind-the-Cover Cards Opaque

If a card is visible underneath a zooming full-screen cover, be careful with
glass:

```swift
CircuitCard(...)
  .surface(radius: Radii.md, glass: false)
```

This keeps the design-system radius, elevation, highlight, and color tokens, but
forces the card onto the opaque content-surface branch. No backdrop sampling
means there is no black presentation layer to sample during dismissal.

This is the tradeoff I missed at first: Liquid Glass can be right for
steady-state UI and wrong for transition-adjacent content.

### 3. Paint the Destination Before the Zoom Modifier

The presented view should paint the app background at the level that receives
the transition:

```swift
fullScreenCover(item: $selectedExercise) { exercise in
  NavigationStack {
    ExerciseDetailView(exercise: exercise)
  }
  .background(Palette.background.ignoresSafeArea())
  .navigationTransition(.zoom(sourceID: exercise.id, in: namespace))
  .presentationBackground(Palette.background)
}
```

The `presentationBackground` still matters, but the extra `.background(...)` on
the transition host prevents the transition snapshot from falling through to the
system's default black.

For a generic card expansion helper, you can do the same thing at the call site:

```swift
.cardExpansion(item: $selectedSession, in: namespace) { session in
  WorkoutSessionView(session: session)
    .background(Palette.background.ignoresSafeArea())
    .presentationBackground(Palette.background)
}
```

The key is modifier order. The stable background must be part of the destination
view before the transition is applied.

## A Useful Debugging Rule

When a SwiftUI zoom transition flashes black, ask:

- Is the matched source transparent?
- Is the source bigger than the visual object the user thinks they tapped?
- Is any visible card behind the cover using glass or material?
- Does the view receiving `.navigationTransition` paint its own background?
- Is the presentation background applied only inside a child view?

Most of my failed attempts only answered one of those questions. The persistent
fix answered all of them.

## The Takeaway

Zoom transitions are snapshot-driven. Snapshots punish ambiguity.

Give the source a real fill. Give the destination a real background. Do not ask
glass to sample a backdrop while the presentation stack is changing underneath
it.

Once those pieces are explicit, the transition feels boring again, which is
exactly what a transition bug fix should feel like.
