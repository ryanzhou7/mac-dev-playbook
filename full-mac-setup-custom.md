# Full Mac Setup Process (for Jeff Geerling)

There are some things in life that just can't be automated... or aren't 100% worth the time :(

This document covers that, at least in terms of setting up a brand new Mac out of the box.

## Initial configuration of a brand new Mac

Before starting, I completed Apple's mandatory macOS setup wizard (creating a local user account, and optionally signing into my iCloud account). Once on the macOS desktop, I do the following (in order):

  - Install Ansible (following the guide in [README.md](README.md))
  - **Sign in in App Store** (since `mas` can't sign in automatically)
  - Clone mac-dev-playbook to the Mac: `git clone git@github.com:ryanzhou7/mac-dev-playbook.git`
  - Run the playbook with `--skip-tags post`.
    - If there are errors, you may need to finish up other tasks like installing 'old-fashioned' apps first (since I try to place Photoshop in the Dock and it can't be installed automatically). Then, run the playbook again ;)
  - Install or complete setup for old-fashioned apps:
    - Open Creative Cloud, sign in, and install needed apps
    - Ex. Open iStat Menus and configure CPU/Net/Temp Combined view
  - Configure FastMail account:
    - Log into Fastmail
    - Go to settings, go to the setup page for macOS Mail
    - Download the profile and double click to install
    - Head to the 'Profiles' System Preference pane and click install
  - Open Calendar and sign into Google Accounts (have to manually sign in):
    - Personal
    - Work
  - Manually copy `~/Development` folder from another Mac (to save time).
  - Manual settings to automate someday:
    - System Preferences:
      - Accessibility > Display > Reduce transparency
      - Keyboard > Keyboard Shortcuts... > Modifier Keys... > Caps Lock to Esc
    - Safari:
      - View > Show Status Bar
      - Preferences > Advanced > "Show full website address"
      - Preferences > Advanced > "Show features for web developers"
    - Dock:
      - Add jgeerling, Downloads, and Applications folders
    - Terminal:
      - Preferences > Profiles > Set JJG-Term as the default theme
  - _After Dropbox Sync completes_: Run the playbook with `--tags post` to complete setup.
  - Symlink the synchronized `config.yml` into the playbook dir: `ln -s /Users/jgeerling/Dropbox/Apps/Config/mac-dev-playbook/config.yml /Users/jgeerling/Development/mac-dev-playbook/config.yml`
  - These things might be automatable, but I do them manually right now:
    - Configure Time Machine backup drive and set Time Machine backups to daily instead of hourly
    - Install Wireguard from App Store and add configuration (if needed)

## To Wrap in Post-provision automation

The following tasks have to wait for the initial Dropbox sync to complete before they'll succeed. So ideally I'll stick this all in a post-provision script but somehow flag it not to run on first provision.

```
# ZSH Aliases.
ln -s /Users/jgeerling/Dropbox/Apps/Config/.aliases /Users/jgeerling/.aliases

# Electrum BTC Wallet.
ln -s /Users/jgeerling/Dropbox/Apps/Electrum/default_wallet /Users/jgeerling/.electrum/wallets/default_wallet

# SSH setup.
ssh-keygen  # and create a default key to set up .ssh folder

https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent

ssh-keygen -t ed25519 -C "ryanzhou7@gmail.com"
enter
eval "$(ssh-agent -s)"
touch ~/.ssh/config

Host github.com
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519

ssh-add --apple-use-keychain ~/.ssh/id_ed25519

https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account

pbcopy < ~/.ssh/id_ed25519.pub

sudo ln -s /Users/jgeerling/Dropbox/Apps/Config/ssh/config ~/.ssh/config
# TODO - Manually copy any shared SSH keys that are needed.


Vscode code -> settings -> settings -> backup and sync -> Github sign in

# Ansible setup.
sudo mkdir -p /etc/ansible
sudo ln -s /Users/jgeerling/Dropbox/Apps/Config/ansible/ansible.cfg /etc/ansible/ansible.cfg
sudo ln -s /Users/jgeerling/Dropbox/Apps/Config/ansible/hosts /etc/ansible/hosts
sudo ln -s /Users/jgeerling/Dropbox/VMs/roles /etc/ansible/roles
mkdir -p /Users/jgeerling/.ansible
ln -s /Users/jgeerling/Dropbox/Apps/Config/ansible/galaxy_token /Users/jgeerling/.ansible/galaxy_token
ln -s /Users/jgeerling/Dropbox/Apps/Config/ansible/mm-vault-password.txt /Users/jgeerling/.ansible/mm-vault-password.txt
ln -s /Users/jgeerling/Dropbox/VMs/ /Users/jgeerling/.ansible/collections

# Final Cut Pro setup. (Open Motion first)
cp -r /Users/jgeerling/Dropbox/Apps/Config/Motion/Motion\ Templates.localized/ /Users/jgeerling/Movies/Motion\ Templates.localized/
cp -r /Users/jgeerling/Dropbox/Apps/Config/Motion/Text\ Styles/ /Users/jgeerling/Library/Application\ Support/Motion/Library/Text\ Styles.localized/

# Sequel Ace favorites. (Open Sequel Ace first)
cp /Users/jgeerling/Dropbox/Apps/Config/Sequel\ Ace/Favorites.plist /Users/jgeerling/Library/Containers/com.sequel-ace.sequel-ace/Data/Library/Application\ Support/Sequel\ Ace/Data/Favorites.plist

# Font setup.
cp ~/Dropbox/Apps/Config/Fonts/* ~/Library/Fonts/

# Vim setup.
mkdir -p ~/.vim/autoload
mkdir -p ~/.vim/bundle
cd ~/.vim/autoload
curl https://raw.githubusercontent.com/tpope/vim-pathogen/master/autoload/pathogen.vim > pathogen.vim
cd ~/.vim/bundle
git clone git://github.com/scrooloose/nerdtree.git
```

## When formatting old Mac

  - Sign out of Adobe Creative Cloud
  - Sign out of Panic Sync in Transmit
  - Deauthorize Apple Music in iTunes/Music App
  - Make sure anything new merged into `~/Dropbox/Apps/Config`:
    - Fonts from ~/Library/Fonts
    - Motion Plugins from ~/Movies/Motion
    - Final Cut Pro Text Styles in ~/Library/Application Support/Motion/Library/Text Styles
    - Sequel Ace shortcuts from ~/Library/Containers/com.sequel-ace.sequel-ace/Data/Library/Application\ Support/Sequel\ Ace/Data/Favorites.plist
  - Follow Apple's guide [here](https://support.apple.com/en-au/HT212749)
