---
title: macOS Developer Tips
date: 2025-05-17
tags:
  - macOS
---

These are handy tools and tweaks that aren't required for a clean install, but
can improve workflow or handle specific tasks.

## 🔧 Git Utilities

- [Revert repo to a previous commit](https://stackoverflow.com/questions/4114095/how-to-revert-a-git-repository-to-a-previous-commit)

- [Rebase without changing timestamps](https://stackoverflow.com/questions/2973996/git-rebase-without-changing-commit-timestamps):
  ```bash
  git rebase -i --root
  # Change pick → edit
  GIT_COMMITTER_DATE="YYYY-MM-DDTHH:MM:SS" git commit --amend --date="YYYY-MM-DDTHH:MM:SS"
  git rebase --committer-date-is-author-date <NEW_SHA>
  ```

## 🛠 Other Custom Tweaks

- **Remove `.DS_Store` files**:

  ```bash

  defaults write com.apple.desktopservices DSDontWriteNetworkStores -bool TRUE
  sudo find / -name .DS_Store -exec rm {} +
  ```

- **Make Dock appear instantly**:
  ```bash
  defaults write com.apple.Dock autohide-delay -float 0.0001; killall Dock
  # To reset:
  defaults delete com.apple.Dock autohide-delay; killall Dock
  ```

## 🐍 Python Environment (Anaconda)

- Create and use an environment:

  ```bash
  conda create --name MKDOCS python=3.9
  conda activate MKDOCS
  pip install mkdocs mkdocs-material
  ```

- Remove environment:
  ```bash
  conda remove --name MKDOCS --all
  conda deactivate
  ```
