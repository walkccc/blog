---
title: What's in my macOS?
date: 2025-05-19
lastmod: 2026-11-05
tags:
  - macOS
---

My Mac setup lives in my private dotfiles repository. I keep the installation and configuration there so a fresh Mac needs only a few steps.

## Initial macOS configuration

### User account setup

If you want to change the capitalization of your account and home folder, do it
before setting up dotfiles. Back up first and follow
[Apple's account-renaming guide](https://support.apple.com/en-us/102547).
Using the fictional account `alex` → `Alex`:

1. Create a temporary administrator account named `Temp`.
2. Log out of the account being renamed and log in as `Temp`.
3. In Finder's `/Users` folder, rename `alex` to `Alex`.
4. In **System Settings > Users & Groups**, Control-click the original account and open **Advanced Options**. Change **User name** (called **Account name** on older macOS versions) to `Alex` and **Home directory** to `/Users/Alex`.
5. Restart, log in to the renamed account, and verify that your files are accessible. Then delete `Temp`.

### System and workflow enhancements

- **Magnet:** install it from the App Store when `/mac-setup` asks; grant its required permissions.
- **Keyboard repeat:** `/mac-setup` sets `KeyRepeat` to `1` and `InitialKeyRepeat` to `20`, preserving the previous values for recovery. Log out and back in if needed.
- **Optional short password:** keep this as a manual choice. The older [discussion of passwords shorter than four characters](https://www.reddit.com/r/MacOS/comments/8z2wo8/can_i_set_a_password_less_than_4_characters/) may not apply to your macOS version; it is not part of dotfiles setup.

## Set up a new Mac

1. Install and sign in to an AI coding agent that supports local skills.
2. Install Apple's Command Line Tools:

   ```bash
   xcode-select --install
   ```

3. Authenticate with GitHub and clone the dotfiles repository to `~/Repos/dotfiles`.
4. Open the checkout in the agent, select the `/mac-setup` skill, and run it.

The workflow installs Homebrew and the shared tool bundles, then configures Zsh, Powerlevel10k, iTerm2, Zed, Neovim, and Raycast. Python and Node come from the managed packages; there is no separate Anaconda, Zim, LunarVim, or Node version manager setup.

## Finish the hands-on steps

Complete the sign-ins, macOS permission prompts, and app settings reported by the setup workflow. For Apple development, finish any required Xcode downloads and signing setup.

Restart the terminal to load the shell configuration. Install project-specific dependencies when a project needs them.
