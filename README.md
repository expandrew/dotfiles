# dotfiles

These are my configuration files and notes for setting up a new Mac.

Do the following, in this order:

1. [Clone this repo](#clone-this-repo)
1. [Install `oh-my-zsh`](#oh-my-zsh)
1. [Install Rosetta 2](#rosetta-2) (for M1 Macs)
1. [Install Homebrew and packages](#homebrew)
1. [Set computer name](#computer-name)
1. [Set up 1Password](#1password)
1. [Sign into iCloud](#icloud)
1. [Sign into Fastmail](#fastmail)
1. [Install Mac App Store apps](#mac-app-store)
1. [Restore Mackup](#mackup)
1. [Set up SSH](#ssh)
1. [Set up Node](#node)

## Clone this repo

Run git once so macOS will try to install command line tools:

```zsh
git
```

Make directory and clone

```zsh
mkdir ~/Code && git clone https://github.com/expandrew/dotfiles ~/Code/dotfiles
```

## oh-my-zsh

```zsh
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

## Rosetta 2

```zsh
sudo softwareupdate --install-rosetta
```

## Homebrew

Install Homebrew

```zsh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

Get `brew` in the PATH for now (this gets resolved for real when Mackup restores my .zshrc below)

```zsh
eval "$(/opt/homebrew/bin/brew shellenv)"
```

Install everything from Brewfile, except Mac App Store apps - `mas` won't work yet, because I haven't signed into iCloud at this point - this has a dependency on 1Password step to get the password, so we'll re-run `brew bundle` again below after 1Password and iCloud are set up.

```zsh
cd ~/code/dotfiles && brew bundle
```

### Multi-user setup

<details>
<summary>Details for using Homebrew with multiple accounts</summary>

If there are multiple user accounts on the same Mac, I need to follow a few extra steps:

1. Open System Preferences > Users & Groups
1. Create a new group called `brew`; add all the users to it
1. Run these steps:

```zsh
% sudo chgrp -R brew $(brew --prefix) # Change group to brew for Homebrew
% sudo chmod -R g+w $(brew --prefix) # Allow group members to write inside this directory
% brew doctor # Make sure everything is good
```

Even with this "shared group" setup, I still run into permissions issues sometimes when running `brew bundle`.

Usually I can resolve it by changing ownership to the current user for the Homebrew folder:

```zsh
% sudo chown -R $USER $(brew --prefix)
% brew bundle # Try installing again
```

</details>

### Brewfile

<details>
<summary>Details for updating Brewfile</summary>

This is how I update the Brewfile when I install/uninstall something:

```zsh
% cd ~/code/dotfiles && brew bundle dump -f
# Then commit the changes to this repo, etc.
```

I just do this periodically and commit it, it's not automated but the command handles the file for me so I don't have to handwrite it

There's a VS Code task to do this in [tasks.json](.vscode/tasks.json).

</details>

## Computer Name

Set the computer name from System Preferences > Sharing.

## 1Password

Set up using the camera / phone thing via mobile app.

## iCloud

Sign into iCloud now that 1Password is set up. Disable all the stuff except these:

- Photos
- Keychain
- iCloud Drive
- Find My Mac

Set up a symlink for iCloud Drive at `~/iCloud`:

```zsh
ln -s ~/Library/Mobile\ Documents/com\~apple\~CloudDocs ~/iCloud
```

Set up a symlink for Obsidian at `~/amw-obsidian`:

```zsh
ln -s ~/Library/Mobile\ Documents/iCloud~md~obsidian/Documents/amw-obsidian ~/amw-obsidian
```

## Fastmail

Create a new App Password for Fastmail at [Settings > Password & Security > Third Party Apps > App Passwords > New App Password](https://www.fastmail.com/settings/security/devicekeys/new).

Once it's created, download the profile with the link it provides, and open System Preferences > Profile and install it.

## Mac App Store

After setting up 1Password and iCloud above, run the Homebrew install step again to install Mac App Store apps.

```zsh
cd ~/code/dotfiles && brew bundle # The `mas` steps should complete now that we're signed into iCloud
```

## Mackup

Restore app settings from mackup

Copy the configuration to the home directory because I don't use the default

```zsh
cp ~/code/dotfiles/mackup/.mackup.cfg ~/
```

Copy the custom config directory for custom apps

```zsh
cp -r ~/code/dotfiles/mackup/.mackup ~/.mackup
```

Restore all the settings

```zsh
mackup restore
```

## SSH

Set up SSH key.

<details>
<summary>Details for setting up SSH key</summary>

### GitHub

(These steps are borrowed from this article: [Generating a new SSH key and adding it to the ssh-agent - GitHub Docs](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent))

Create a key and set the passphrase to the same as the login password for macOS:

```zsh
% ssh-keygen -t ed25519 -C "<my email address>"
```

Add the key to SSH agent on macOS:

```zsh
% eval "$(ssh-agent -s)"
% ssh-add -K ~/.ssh/id_ed25519
```

Copy the public key to the clipboard then go to [GitHub Settings > SSH and GPG keys](https://github.com/settings/keys) and add it:

```zsh
pbcopy < ~/.ssh/id_ed25519.pub
```

Test SSH connection to GitHub:

```zsh
ssh -T git@github.com
```

</details>

## Node

Install [nvm](https://github.com/nvm-sh/nvm):

```zsh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
```

Close and reopen the terminal, then install the lastest available version of Node:

```zsh
nvm install node
```
