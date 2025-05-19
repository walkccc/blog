---
title: What's in my macOS?
date: 2025-05-19
tags:
  - macOS
---

After completely reinstalling macOS, here's how I like to set up a clean,
powerful, and personalized development environment.

## 1. Initial macOS Configuration

### 🧑‍💻 User Account Setup

1. **Create a temporary Admin user**
   - Name it `Temp`
1. **Login with Temp**, then:
   - Rename your home folder from `jay` to `Jay`
   - Open **System Settings > Users & Groups**
     - Change **Account Name**: `jay` → `Jay`
     - Change **Home Directory**: `/Users/jay` → `/Users/Jay`
1. Log out and log in as `Jay`
1. Delete `Temp` user

### ⚡️ System & Workflow Enhancements

- **Install [Magnet](https://apps.apple.com/us/app/magnet/id441258766?mt=12)**
  from the App Store (window manager)
- **Increase keyboard repeat speed**:
  ```bash
  defaults write NSGlobalDomain KeyRepeat -int 1
  defaults write NSGlobalDomain InitialKeyRepeat -int 20
  ```
  _(You may need to log out and log back in for these changes to take full
  effect.)_
- (Optional)
  [Set a password shorter than 4 characters](https://www.reddit.com/r/MacOS/comments/8z2wo8/can_i_set_a_password_less_than_4_characters/)

## 2. Essential Command-Line Tools

### 🍺 [Homebrew](https://brew.sh)

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

_(Follow the on-screen instructions, which may include adding Homebrew to your
PATH.)_

**Install essential packages:**

```bash
brew install --cask iterm2
brew install --cask visual-studio-code
brew install --cask cursor
brew install --cask anaconda
brew install --cask raycast
brew install gh
```

### Zsh and [Zim](https://zimfw.sh) Framework

**Configure Zsh Environment for Zim:** Add the following lines to `~/.zshenv`.
This tells Zim where to store its configuration files, adhering to the XDG Base
Directory specification.

```bash
echo 'export ZDOTDIR=$HOME/.config/zsh' >> ~/.zshenv
echo 'export ZIM_HOME=$ZDOTDIR/.zim' >> ~/.zshenv
```

_(If `~/.zshenv` doesn't exist, these commands will create it.)_

**Install Zim:**

```bash
curl -fsSL https://raw.githubusercontent.com/zimfw/install/master/install.zsh | zsh
```

## 3. Dotfiles & Terminal Customization

> Personalize your shell environment with custom configurations.

### ⚙️ Install Dotfiles

Clone and set up your preferred dotfiles. This example uses
[walkccc's dotfiles](https://github.com/walkccc/dotfiles).

```bash
cd ~
curl https://codeload.github.com/walkccc/dotfiles/tar.gz/main | tar -xz --strip=1 dotfiles-main
rm README.md LICENSE .gitignore
```

### ✨ Apply Changes & Theme

Restart your terminal (iTerm2) to apply the new Zsh and Zim configurations. This
will also likely trigger the installation of themes like
[Powerlevel10k](https://github.com/romkatv/powerlevel10k) if it's part of your
dotfiles or Zim setup. Follow any on-screen prompts from Powerlevel10k to
configure your prompt.

## 4. Development Environment Setup

### 🐍 Python (via Anaconda)

**Initialize Conda for Zsh:**

```bash
/opt/homebrew/anaconda3/bin/conda init zsh
```

_(The path might vary slightly depending on your Homebrew prefix if not default.
`/opt/homebrew/anaconda3/bin/conda init zsh` for Apple Silicon usually)_

**Restart Terminal:** After restarting, Conda's configuration should be active
in `$ZDOTDIR/.zshrc`.

### 🟩 Node (via Node Version Switcher - NVS)

**Install [NVS](https://github.com/jasongin/nvs)** This method installs NVS
following the
[XDG Base Directory](https://wiki.archlinux.org/title/XDG_Base_Directory)
specification for `NVS_HOME`.

```bash
export NVS_HOME="$HOME/.local/share/nvs"
git clone https://github.com/jasongin/nvs "$NVS_HOME"
. "$NVS_HOME/nvs.sh" install
```

**Add NVS to your PATH:** To make `nvs` available in every new shell session,
add the following to your Zsh configuration file.

```bash
# The following line should already be present in your Zsh configuration file:
# $ bat $ZDOTDIR/init/nvs.sh
# export NVS_HOME="$HOME/.local/share/nvs"
# [ -s "$NVS_HOME/nvs.sh" ] && . "$NVS_HOME/nvs.sh"
```

**Install Node.js versions** as needed using `nvs add <version>` and
`nvs use <version>`.

## 5. Productivity & Developer Tools

### 📝 Text Editor: [LunarVim](https://www.lunarvim.org/)

Install the pre-requisites:

```bash
brew install neovim
```

Install LunarVim by running the command in
[LunarVim's website](https://www.lunarvim.org/docs/installation#release).

### 🚀 Launcher & Productivity: [Raycast](https://www.raycast.com/)

> Configure Raycast to enhance your productivity.

1. Set up **Quicklinks**.
1. Enable **Clipboard History**.
1. Link any custom **Scripts** (e.g., those downloaded with your dotfiles) to
   Raycast's script command directory. A common location for user scripts is
   `~/.config/raycast/commands`. You might need to create this directory and
   symlink or copy your scripts there.

## 6. System Customizations

### 🔍 Finder Customization

```bash
# Show full path in Finder
defaults write com.apple.finder _FXShowPosixPathInTitle -bool true; killall Finder

# Hide full path in Finder
defaults write com.apple.finder _FXShowPosixPathInTitle -bool false; killall Finder
```
