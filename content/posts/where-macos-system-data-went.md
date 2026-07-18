+++
date = 2026-07-18T00:00:00-04:00
title = "Where 271 GB of \"System Data\" Went on My Mac"
tags = ["macOS", "Docker", "Xcode"]
categories = ["Engineering"]
+++

macOS Storage settings told me I had **271 GB of "System Data."** That number
made no sense. My whole life on this machine is Claude Code, `xcodebuild`, and
`adb install` — I don't hoard 4K movies, I don't run a fleet of VMs, and I
certainly can't point at a quarter-terabyte of anything I'd call "system."

The short version: **"System Data" is a lie of omission.** It's the bucket macOS
drops everything it can't neatly label into, and for a developer that bucket
fills up with build caches you forgot exist. This is the note I wish I'd had
before I started deleting things.

## "System Data" Is a Catch-All, Not a Location

The first instinct is to `rm -rf` something and hope. Resist it. The Storage bar
gives you a _number_, not a _place_. When I actually walked the disk, the real
`/System` was only 54 GB:

```bash
sudo du -xhd 1 /System/Volumes/Data 2>/dev/null | sort -h
```

```text
 11G    /System/Volumes/Data/private
 21G    /System/Volumes/Data/Applications
 21G    /System/Volumes/Data/opt
 31G    /System/Volumes/Data/Library
 54G    /System/Volumes/Data/System
235G    /System/Volumes/Data/Users
374G    /System/Volumes/Data
```

So the 271 GB "System Data" figure isn't `/System` being fat. It's macOS lumping
Docker, Xcode, Simulators, Gradle, Homebrew, and every app's Application Support
folder into one unlabeled pile. None of that is the operating system. It's my
own work, mis-filed.

That reframes the task. It isn't "clean System Data." It's "find what's actually
big, then decide what's safe to delete." Which means measuring first.

## `du` Is the Whole Game

Before touching anything, hunt with `du -h -d 1 ... | sort -h`. It's the same
move at every level: point it at a directory, sort by size, look at the tail.

```bash
# Developer tooling
du -sh ~/Library/Developer/* 2>/dev/null | sort -h
du -sh ~/Library/Developer/Xcode/* 2>/dev/null | sort -h

# The usual big three under ~/Library
du -h -d 1 ~/Library/Containers 2>/dev/null | sort -h | tail -20
du -h -d 1 ~/Library/Caches 2>/dev/null | sort -h | tail -20
du -h -d 1 ~/Library/Application\ Support 2>/dev/null | sort -h | tail -20

# Android
du -h -d 2 ~/.android/avd 2>/dev/null | sort -h | tail
du -h -d 2 ~/Library/Android/sdk 2>/dev/null | sort -h | tail

# When Home is bigger than ~/Library, something else is hiding
du -xhd 1 ~ 2>/dev/null | sort -h
```

The `tail` matters more than the `head` — you only care about the handful of
directories eating tens of gigabytes, not the hundreds sitting at a few MB.

## The Culprits

Once I sorted everything, a very clear ranking fell out. One giant, then a long
tail of dev caches:

| What                     | Size    | Path                                              |
| ------------------------ | ------- | ------------------------------------------------- |
| **Docker**               | 46 GB   | `~/Library/Containers/com.docker.docker`          |
| Homebrew cache           | 14 GB   | `~/Library/Caches/Homebrew`                       |
| Android SDK system image | 8.2 GB  | `~/Library/Android/sdk/system-images`             |
| Android emulator (AVD)   | 8.9 GB  | `~/.android/avd` (3.8 GB of it snapshots)         |
| Xcode DerivedData        | 7.6 GB  | `~/Library/Developer/Xcode/DerivedData`           |
| Gradle cache             | 7.6 GB  | `~/.gradle/caches`                                |
| iOS DeviceSupport        | 2×5.7 G | `~/Library/Developer/Xcode/iOS DeviceSupport`     |
| watchOS DeviceSupport    | 5.3 GB  | `~/Library/Developer/Xcode/watchOS DeviceSupport` |
| Spotify cache            | 3.2 GB  | `~/Library/Caches/com.spotify.client`             |

Almost none of it is precious. It's regenerable: build output, downloaded
dependencies, debug symbols for phones I've since updated, and one Docker
install quietly sitting on 46 GB.

## Cleaning, Safest First

I worked down the list from "definitely regenerates itself" to "check before you
delete."

**Xcode build cache — the safest single cut.** DerivedData is pure build output.
Deleting it just means the next build recompiles.

```bash
rm -rf ~/Library/Developer/Xcode/DerivedData/*
xcrun simctl delete unavailable   # drop simulators for runtimes you no longer have
```

**Old device-support bundles.** Xcode keeps a per-OS-version debug bundle every
time you attach a physical device. I had two iOS builds side by side — I only
still debug against the newer one:

```bash
du -sh ~/Library/Developer/Xcode/iOS\ DeviceSupport/*
rm -rf ~/Library/Developer/Xcode/iOS\ DeviceSupport/"iPhone18,1 26.5.1 (23F81)"
```

**Android caches and emulator snapshots.** Gradle re-downloads what it needs,
and the emulator's Quick Boot snapshots (3.8 GB here) rebuild themselves — just
close the emulator first.

```bash
rm -rf ~/.gradle/caches/*
rm -rf ~/.android/avd/*/snapshots/*
```

Since I mostly `adb install` straight onto a real phone, the whole emulator plus
its system image (~17 GB together) is arguably deletable too — via **Device
Manager → Delete** and **SDK Manager**, not `rm`, so the SDK metadata stays
consistent.

**Homebrew and Spotify.** Both are entirely re-downloadable.

```bash
brew cleanup --prune=all
brew cleanup -s
rm -rf ~/Library/Caches/com.spotify.client/*   # Spotify closed
```

**Docker — the 46 GB elephant.** This one you do _not_ `rm -rf`. Start Docker
Desktop, see what's actually in there, then prune:

```bash
open -a Docker
docker system df -v          # what's using the space
docker builder prune -a      # dead build cache
docker system prune -a       # stopped containers, dangling images, networks
```

Note the deliberate omission of `--volumes`. Build cache and dangling images are
disposable; a named volume might be the only copy of a database. Prune the
former freely, leave the latter alone unless you've checked.

All told this was a comfortable 60–80 GB back with zero risk to anything I
actually needed.

## Rule Out APFS Snapshots

If the numbers still don't add up after clearing caches, the classic hidden
culprit is local Time Machine snapshots silently pinning old data. Check:

```bash
tmutil listlocalsnapshots /
diskutil apfs listSnapshots /
```

Mine came back essentially empty — just one `com.apple.os.update-…` snapshot,
which is the OS-update rollback point. **Leave that one alone.** But if you see
a stack of `com.apple.TimeMachine.*` snapshots, that's your missing space, and
`tmutil deletelocalsnapshots` (or just letting them age out) is the fix.

## The Takeaway

"System Data" is not a thing you clean. It's a label for everything macOS
couldn't be bothered to categorize — and on a developer's machine, that's mostly
build caches, downloaded dependencies, debug symbols, and one Docker install
that grew a beard.

The workflow that matters isn't a list of magic `rm` commands. It's the order of
operations: **measure with `du`, sort by size, identify the few real offenders,
then delete regenerable caches from safest to riskiest.** Do that and the scary
271 GB number turns into a boring afternoon of housekeeping.
