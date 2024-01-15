# Package Management

## Table of Contents

<!-- vim-markdown-toc GFM -->

* [Pacman](#pacman)
    * [AUR Setup](#aur-setup)
    * [Update packages](#update-packages)
    * [Cleanup package cache](#cleanup-package-cache)
    * [Reconfigure Pacman key](#reconfigure-pacman-key)
    * [Clean unused packages](#clean-unused-packages)
    * [Refresh keys](#refresh-keys)
* [Aura](#aura)
    * [Install](#install)
    * [Installing a Package](#installing-a-package)
    * [Search & Query](#search--query)
    * [Manage Orphan](#manage-orphan)
    * [Clearing Package Cache](#clearing-package-cache)
    * [Package Set Snapshots](#package-set-snapshots)
* [Nix](#nix)
    * [Install](#install-1)
    * [Set up max jobs](#set-up-max-jobs)
    * [Update channel](#update-channel)
    * [Usage](#usage)

<!-- vim-markdown-toc -->

## Pacman

### AUR Setup

Source: https://github.com/Morganamilo/paru

```bash
sudo pacman -S archlinuxcn-keyring
sudo pacman -S yay
sudo pacman -S paru
# alias yay='paru'
```

### Update packages

Reference: https://wiki.archlinux.org/title/Pacman

```bash
# pacman
sudo pacman -Syyu --no-confirm
alias pacsyu="sudo pacman -Syyu --no-confirm"

# yay
paru -Syyu --no-confirm
alias yaysyu="paru -Syyu --no-confirm"
```

### Cleanup package cache

```bash
# remove isolated packages
sudo pacman -Rns $(pacman -Qtdq)

# remove a package and its dependencies which are not used by any other packages
sudo pacman -Rns package_name

# clean pacman cache
sudo pacman -S pacman-contrib
sudo mkdir -p /etc/pacman.d/hooks
# /etc/pacman.d/hooks/clean_package_cache.hook
[Trigger]
Operation = Upgrade
Operation = Install
Operation = Remove
Type = Package
Target = *

[Action]
Description = Cleaning pacman cache...
When = PostTransaction
Exec = /usr/bin/paccache -rk 2

# clean yay cache manually
paru -Sc
yay -Sc
rm -rf $HOME/cache/yay
rm -rf $HOME/cache/paru
```

### Reconfigure Pacman key

```bash
sudo pacman-key --populate
sudo pacman -Sy archlinux-keyring && sudo pacman -Su
```

### Clean unused packages

Reference: https://www.scivision.dev/pacman-autoremove-unused/

```bash
# show the auto-installed prerequisites
sudo pacman -Qdtq
# piped into the Pacman remove command upon verifying the packages above are indeed OK to remove
sudo pacman -Qdtq | sudo pacman -Rs -
```

### Refresh keys

```bash
sudo pacman-key --refresh-keys
```

## Aura

Cookbook: https://fosskers.github.io/aura/introduction.html

Source: https://github.com/fosskers/aura

### Install

```bash
sudo paru -S aura-bin
```

### Installing a Package

```bash
# Searching for a package
aura -As firefox

# Scrutinize a package
aura -Ai firefox

# Normal install
aura -A firefox

# Update regular pacman packages
sudo aura -Syu

# Updating AUR packages
sudo aura -Auax
```

### Search & Query

```bash
# Searching an exact package
aura -Qi firefox
```

### Manage Orphan

```bash
# Determine what packages have become orphans
aura -O


# Uninstalling Orphans
sudo aura -Oj
```

### Clearing Package Cache

```bash
# Removing the tarballs of uninstalled packages
sudo aura -Sc

# Removing all tarballs
sudo aura -Scc
```

### Package Set Snapshots

```bash
# Store a JSON record of all installed packages
aura -B

# Restore a saved record. Rolls back and uninstalls as necessary
aura -Br

# Delete all but the most recent n saved states
aura -Bc <n>

# Show all saved package state filenames
aura -Bl
```

## Nix

Guide

- <https://christitus.com/nix-package-manager/>
- <https://wiki.archlinux.org/title/Nix>

### Install

```bash
sh <(curl -L https://nixos.org/nix/install) --daemon

# Once done, close the current terminal as it won't work on the current terminal session
```

### Set up max jobs

```
# /etc/nix/nix.conf
max-jobs = auto
```

### Update channel

```bash
nix-channel --add https://nixos.org/channels/nixpkgs-unstable
nix-channel --update
```

### Usage

Ref: <https://nixos.org/manual/nix/stable/package-management/basic-package-mgmt.html>

- List Installed packages `nix-env -q`
- Install Packages `nix-env -iA nixpkgs.packagename`
- Erase Packages `nix-env -e packagename`
- Update All Packages `nix-env -u`
- Update Specific Packages `nix-env -u packagename`
- Hold Specific Package `nix-env --set-flag keep true packagename`
- List Backups (Generations) `nix-env --list-generations`
- Rollback to Last Backup `nix-env --rollback`
- Rollback to Specific Generation `nix-env --switch-generation #`

Garbage collection

Ref: https://nixos.org/manual/nix/unstable/package-management/garbage-collection.html

- To delete all old (non-current) generations of your current profile `nix-env --delete-generations old`
- Instead of old you can also specify a list of generations, e.g. `nix-env --delete-generations 10 11 14`
- To delete all generations older than a specified number of days (except the current generation), use the d suffix. e.g. `nix-env --delete-generations 14d` (deletes all generations older than two weeks.)
- After removing appropriate old generations you can run the garbage collector as follows `nix-store --gc`
