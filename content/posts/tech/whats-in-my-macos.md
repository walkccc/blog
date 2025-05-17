---
title: What's in my macOS?
date: 2025-05-17
tags:
  - macOS
---

After completely reinstalling macOS, here's how I like to set up a clean,
powerful, and personalized development environment.

## 🧑‍💻 Personal Setup

1. **Create a temporary Admin user**
   - Name it `Temp`
1. **Login with Temp**, then:
   - Rename your home folder from `jay` to `Jay`
   - Open **System Settings > Users & Groups**
     - Change **Account Name**: `jay` → `Jay`
     - Change **Home Directory**: `/Users/jay` → `/Users/Jay`
1. Log out and log in as `Jay`
1. Delete `Temp` user

## ⚡️ Workflow Boosters

- **Install [Magnet](https://apps.apple.com/us/app/magnet/id441258766?mt=12)**
  from the App Store (window manager)
- **Increase keyboard repeat speed**:
  ```bash
  defaults write NSGlobalDomain KeyRepeat -int 1
  defaults write NSGlobalDomain InitialKeyRepeat -int 20
  ```
- [Set a password shorter than 4 characters](https://www.reddit.com/r/MacOS/comments/8z2wo8/can_i_set_a_password_less_than_4_characters/)
  (optional)

## 🍺 Homebrew & Terminal Setup

1. **Install [Homebrew](https://brew.sh)**
   ```bash
   /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
   ```
1. **Install iTerm2 and [Zim](https://zimfw.sh)**

   ```bash
   # Install iTerm2 via Homebrew!
   brew install --cask iterm2

   # Install Zim
   curl -fsSL https://raw.githubusercontent.com/zimfw/install/master/install.zsh | zsh

   # Add powerlevel10k
   echo 'zmodule romkatv/powerlevel10k' >> ~/.zimrc

   # Install powerlevel10k
   zimfw install
   # Now, `powerlevel10k` folder appears in `/Users/Jay/.zim/modules`

   # Customize powerlevel10k's icon in ~/.p10k.zsh
   # Change
   #     `typeset -g POWERLEVEL9K_VCS_BRANCH_ICON=`
   # to
   #     `typeset -g POWERLEVEL9K_VCS_BRANCH_ICON='\uF126 '`
   ```

## 🔍 Finder Customization

```bash
# Show full path in Finder
defaults write com.apple.finder _FXShowPosixPathInTitle -bool true; killall Finder

# Hide full path in Finder
defaults write com.apple.finder _FXShowPosixPathInTitle -bool false; killall Finder
```

## ⚙️ Dotfiles

Clone and install [`walkccc`'s dotfiles](https://github.com/walkccc/dotfiles):

```bash
cd ~
curl https://codeload.github.com/walkccc/dotfiles/tar.gz/main | tar -xz --strip=1 dotfiles-main
rm README.md LICENSE .gitignore
```

## 🧩 Safari + Xcode

Install Xcode and fix the `xcode-select` path:

```bash
sudo xcode-select -s /Applications/Xcode.app
xcrun safari-web-extension-converter ~/Repositories/leetcode-search-by-question-id
```

## 🐍 Python Environment (Anaconda)

1. Install Anaconda:
   ```bash
   brew install --cask anaconda
   ```
1. Initialize Conda:
   ```bash
   /usr/local/anaconda3/bin/conda init zsh
   ```
1. Restart Terminal → Conda config appears in `~/.zshrc`

## 🟩 Node Environment (NVM)

1. Install NVM:

   ```bash
   brew install nvm
   ```

2. Add to `~/.zshrc`:

   ```bash
   export NVM_DIR="$HOME/.nvm"
   [ -s "/usr/local/opt/nvm/nvm.sh" ] && . "/usr/local/opt/nvm/nvm.sh"
   [ -s "/usr/local/opt/nvm/etc/bash_completion.d/nvm" ] && . "/usr/local/opt/nvm/etc/bash_completion.d/nvm"
   ```

3. Fix insecure directory warning:

   ```bash
   compaudit | xargs chmod g-w
   ```

4. Install/Use Node:
   ```bash
   nvm use system
   nvm run system --version
   nvm install 14.15.3
   nvm install 15.5.0
   ```

## 🚀 Raycast + Extensions Setup

1. **Install Raycast**

   ```bash
   brew install --cask raycast
   ```

1. Set up **Quicklinks**
1. Enable **Clipboard History**
1. Link **Scripts** downloaded from dotfiles to `~/.config/raycast/commands`
