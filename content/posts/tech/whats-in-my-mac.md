---
title: What's inside my macOS?
date: 2021-01-01
tags:
  - macOS
---

After completely reinstalling the whole new macOS, you probably want to make
everything in a great order.

## Personal Preference

- [Capitalize home folder name](https://forums.macrumors.com/threads/macos-sierra-home-folder-capitalization.1999928/)
  (deprecated)

1. Create a new Admin user (`Temp`)
1. Login with the new Admin user
1. Change the folder name from `jay` to `Jay`
1. Go to Users & Groups
   - Change **Account name** from `jay` to `Jay`
   - Change **Home directory** from `/Users/jay` to `/Users/Jay`
1. Login with `Jay`
1. Delete `Temp`

## Boost Up Workflow

1. Download **Magnet** from App Store
1. [Increase cursor speed](https://stackoverflow.com/questions/39626416/macos-sierra-a-way-to-move-the-cursor-when-holding-down-arrow-key)
   ```bash
   defaults write NSGlobalDomain KeyRepeat -int 1
   defaults write NSGlobalDomain InitialKeyRepeat -int 20
   ```
1. [Set the password less than 4 characters](https://www.reddit.com/r/MacOS/comments/8z2wo8/can_i_set_a_password_less_than_4_characters/)

## Setup Homebrew and Terminal

We use Homebrew since it's a great package manager.

1. Install [**Homebrew**](https://brew.sh/index_zh-tw)
   ```bash
   /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
   ```
1. Install [**iTerm2**](https://www.iterm2.com/) +
   [**zimfw**](https://github.com/zimfw/zimfw)

   ```bash
   # Install iTerm2 via Homebrew!
   brew install --cask iterm2

   # Install zimfw
   curl -fsSL https://raw.githubusercontent.com/zimfw/install/master/install.zsh | zsh

   # Add powerlevel10k
   echo 'zmodule romkatv/powerlevel10k' >> ~/.zimrc

   # Install powerlevel10k
   zimfw install
   # Now, `powerlevel10k` folder appears in `/Users/Jay/.zim/modules`

   # In /Users/Jay/.p10k.zsh
   # Change
   #     `typeset -g POWERLEVEL9K_VCS_BRANCH_ICON=`
   # to
   #     `typeset -g POWERLEVEL9K_VCS_BRANCH_ICON='\uF126 '`
   ```

1. Resolve VSCode Powerline problem

   - Add Nerd Font

     ```bash
     # Search the font that supports `powerlevel10k`
     brew install nerd
     brew install --cask homebrew/cask-fonts/font-sauce-code-pro-nerd-font

     # Change the font in iTerm/Terminal
     # Go to iTerm's Preferences
     #             > Profiles
     #             > Text
     # Choose "SauceCodePro Nerd Font Mono"

     # Change the font to 18 (be good to your eyes)
     ```

   - Add
     `"terminal.integrated.fontFamily": "monospace, 'SauceCodePro Nerd Font Mono'"`
     in `settings.json`

## Show/Hide path in the Finder title bar

```bash
# Show
defaults write com.apple.finder _FXShowPosixPathInTitle -bool true; killall Finder
# Hide
defaults write com.apple.finder _FXShowPosixPathInTitle -bool false; killall Finder
```

## C++

- In VSCode > `settings.json`

  ```
  "clang-format.executable": "/usr/local/bin/clang-format",
  "[cpp]": {
      "editor.defaultFormatter": "xaver.clang-format"
  },
  ```

## Java

- `brew install --cask AdoptOpenJDK/openjdk/adoptopenjdk{11,15}`
- In `~/.zshrc`:

  ```bash
  # JDK Switch
  export JAVA_11_HOME=`/usr/libexec/java_home -v 11` # 把 Java 11 的路徑位置存到左側變數
  export JAVA_15_HOME=`/usr/libexec/java_home -v 15` # 把 Java 15 的路徑位置存到左側變數

  # Default version = Java 11
  export JAVA_HOME=$JAVA_11_HOME

  alias jdk11='export JAVA_HOME=$JAVA_11_HOME'
  alias jdk15='export JAVA_HOME=$JAVA_15_HOME'
  ```

- To be continued: https://java.christmas/2019/16

## Docker

[Setup completion](https://docs.docker.com/docker-for-mac/#zsh)

```bash
etc=/Applications/Docker.app/Contents/Resources/etc
ln -s $etc/docker.zsh-completion /usr/local/share/zsh/site-functions/_docker
ln -s $etc/docker-compose.zsh-completion /usr/local/share/zsh/site-functions/_docker-compose
```

<!-- Then, add these two lines:

```
plugins=(docker docker-compose)
autoload -Uz compinit && compinit -i
``` -->

## iTerm

- In Preferences > Profiles > Windows

  ![](https://i.imgur.com/ngLUbr4.png)

- In Preferences > Keys > Hotkey Click "Show/hide all windows with a system-wide
  hotkey" Record the hotkey you prefer.

## VSCode

Open In VSCode

![](https://i.imgur.com/8Sw8Mdd.png)

```bash
open -n -b "com.microsoft.VSCode" --args "$*"
```

[vs-code.icns](https://github.com/cnstntn-kndrtv/open-in-buttons-for-finder-toolbar/blob/master/src/icons/vs-code.icns)

## Safari Extension, Xcode

We need `safari-web-extension-converter` to convert our Chrome Extension.
However, out `xcode-select` doesn't link to the correct path.

```bash
# original
xcode-select -p # /Library/Developer/CommandLineTools

# connect to the correct path
sudo xcode-select -s sudo xcode-select -s /Applications/Xcode.app

# now, we can do the following to convert our app
xcrun safari-web-extension-converter ~/Repositories/leetcode-search-by-question-id
```

## Python Env

```bash
# Make sure to install `anaconda` by brew first
brew install --cask anaconda

# Init `conda` because we don't have this command in `macOS`
cd /usr/local/anaconda3/bin

# Init zsh configuration
./conda init zsh
# Restart the terminal, the conda configuration is now in ~/.zshrc

# Choose one
## Turn off conda default env
conda config --set auto_activate_base false
## Turn on conda default env
conda config --set auto_activate_base true
```

```bash
# e.g.
# Create an env for Mkdocs deployment
conda create --name MKDOCS python=3.8

# Activate this environment
conda activate MKDOCS

# Now work with this cute MKDOCS env!
pip install mkdocs
pip install mkdocs-material

# Happy deploying!
```

```bash
# Remove <ENV NAME>
conda remove --name <ENV NAME> --all

# Deactivate the current environment
conda deactivate
```

## Node Env

[Ref](https://yimincai.net/2020/install-and-uninstall-nvm-on-macos/)

```bash
brew install nvm
```

Add the following in your `~/.zshrc`

```bash
# nvm
export NVM_DIR="$HOME/.nvm"
[ -s "/usr/local/opt/nvm/nvm.sh" ] && . "/usr/local/opt/nvm/nvm.sh"  # This loads nvm
[ -s "/usr/local/opt/nvm/etc/bash_completion.d/nvm" ] && . "/usr/local/opt/nvm/etc/bash_completion.d/nvm"  # This loads nvm bash_completion
```

You many see this message:

```bash
zsh compinit: insecure directories, run compaudit for list.
Ignore insecure directories and continue [y] or abort compinit [n]?
```

Security: For security reasons compinit also checks if the completion system
would use files not owned by root or by the current user, or files in
directories that are world- or group-writable or that are not owned by root or
by the current user. If such files or directories are found, compinit will ask
if the completion system should really be used. To avoid these tests and make
all files found be used without asking, use the option -u, and to make compinit
silently ignore all insecure files and directories use the option -i. This
security check is skipped entirely when the -C option is given.

```bash
compaudit | xargs chmod g-w
```

```
nvm use system
nvm run system --version
nvm install 14.15.3
nvm install 15.5.0
```

## Vim/Neovim

```bash
# install neovim
brew install neovim

# install vim-plug under neovim
sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs \
       https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'
```

The Neovim config file is located in `~/.config/nvim/init.vim`.

```bash
mkdir -p ~/.config/nvim
touch ~/.config/nvim/init.vim
nvim ~/.config/nvim/init.vim
```

Paste the following code in `~/.config/nvim/init.vim`.

It allows you to configure plugins by adding references to their GitHub
repositories:

```vim
call plug#begin('~/.vim/plugged')

" [Plugin section]
Plug 'dracula/vim'

call plug#end()

" [Config section]
if (has("termguicolors"))
  set termguicolors
endif
syntax enable
colorscheme dracula
```

Install the plugin and reload the editor to have an effect.

```
nvim +PlugInstall
```

## Others

- [iCloud Drive syncing problem](https://apple.stackexchange.com/questions/313716/icloud-drive-wont-sync-on-mac)

- Location of `bits/stdc++.h`

  ```bash
  /usr/local/include/bits/stdc++.h
  ```

- [macOS - Can't compile C program on a Mac after upgrade to Mojave - Stack Overflow](https://stackoverflow.com/questions/52509602/cant-compile-c-program-on-a-mac-after-upgrade-to-mojave)

  ```bash
  open /Library/Developer/CommandLineTools/Packages/macOS_SDK_headers_for_macOS_10.14.pkg
  ```

- [Remove .DS_Store](https://www.tekrevue.com/tip/stop-ds-store-files-mac-network/)

  ```bash
  sudo find / -name .DS_Store
  defaults write com.apple.desktopservices DSDontWriteNetworkStores -bool TRUE
  sudo find / -name .DS_Store -exec rm {} +
  ```

- [Make dock appear instantly](https://howchoo.com/g/mmuwzwnmmzn/make-the-dock-autohide-and-show-instantly-in-os-x)

  ```bash
  defaults write com.apple.Dock autohide-delay -float 0.0001; killall Dock
  defaults delete com.apple.Dock autohide-delay; killall Dock
  ```

## Git

1. [Set up commit information](https://stackoverflow.com/questions/14657913/this-commit-is-missing-author-information)

   - Globally

     ```bash
     # set the gitconfig
     git config --global user.name 'Jay Chen'
     git config --global user.email 'walkccray@gmail.com'

     # check the gitconfig
     cat ~/.gitconfig
     # [user]
     #         name = Jay Chen
     #         email = walkccray@gmail.com
     ```

   - Locally (in the Git repository):

     ```bash
     git config -l
     git config user.name 'Jay Chen'
     git config user.email 'walkccray@gmail.com'
     ```

2. [Revert a Git repository to a previous commit](https://stackoverflow.com/questions/4114095/how-to-revert-a-git-repository-to-a-previous-commit)

3. [Rebase without changing timestamps](https://stackoverflow.com/questions/2973996/git-rebase-without-changing-commit-timestamps)

```bash
git rebase -i --root
# Change the commit from "pick" to "edit"
# :wq
GIT_COMMITTER_DATE="2018-03-20T12:00:00" git commit --amend --date="2018-03-20T12:00:00"
# Check the SHA of the new root commit message
git rebase --committer-date-is-author-date SHA_OF_ROOT
```
