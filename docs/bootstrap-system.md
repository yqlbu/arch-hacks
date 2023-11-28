# Bootstrap System

## Table of Contents

<!-- vim-markdown-toc GFM -->

* [Disable reflector service](#disable-reflector-service)
* [Configure mirrors](#configure-mirrors)
* [Installation](#installation)
    * [Partitioning drives](#partitioning-drives)
    * [Create encryption](#create-encryption)
    * [Format paritions](#format-paritions)
    * [Create sub-volumes](#create-sub-volumes)
* [Install base system](#install-base-system)
* [Bootstrap system](#bootstrap-system)
    * [Set timezone](#set-timezone)
    * [Set locale](#set-locale)
    * [Set hostname](#set-hostname)
    * [Create default user](#create-default-user)
    * [Enable default services](#enable-default-services)
    * [Setup boot loader (Systemd Boot)](#setup-boot-loader-systemd-boot)
    * [Unmount partition](#unmount-partition)
    * [Reboot](#reboot)
* [Configuration](#configuration)
    * [Set timezone](#set-timezone-1)
    * [Set locale](#set-locale-1)
    * [Set hostname](#set-hostname-1)
    * [Configure source](#configure-source)
    * [Configure AUR](#configure-aur)
    * [Configure Aura](#configure-aura)
    * [Configure Nix](#configure-nix)
    * [Configure zramd](#configure-zramd)
    * [Install system related packages](#install-system-related-packages)
        * [System related](#system-related)
        * [Network related](#network-related)
        * [Bluetooh related](#bluetooh-related)
        * [Audio related](#audio-related)
    * [Load kernel modules at boot](#load-kernel-modules-at-boot)
    * [Create home dirs](#create-home-dirs)
* [Install dev-toolkit](#install-dev-toolkit)
    * [Basic tools](#basic-tools)
    * [Compliation tools](#compliation-tools)
    * [Nodejs](#nodejs)
    * [Python](#python)
    * [Golang](#golang)
    * [Rust](#rust)

<!-- vim-markdown-toc -->

## Set consolefonts

```bash
# ls available consolefonts
ls /usr/share/kbd/consolefonts/
# set font
setfont ter-v32n
```

## Disable reflector service

```bash
sudo systemctl stop reflector.service
```

## Configure mirrors

```bash
# /etc/pacman.conf
[options]
Color
ParallelDownload = 5
[multilib]
Include = /etc/pacman.d/mirrorlist
# Add CN source
[archlinuxcn]
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch

# /etc/pacman.d/mirrorlist
# Paste the following lines on top of the list
Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
```

## Installation

Reference: https://www.youtube.com/watch?v=HIXnT178TgI

- [Partitioning drives](#partitioning-drives)
- [Install base system](#install-base-system)
- [Bootstrap system](#bootstrap-system)

### Partitioning drives

Format drive

```bash
wipefs --all /dev/nvme0n1
```

With gdisk

```bash
# list paritions
lsblk

# create patitions
gdisk /dev/nvme0n1
# create efi partition
# n
# size: +300M
# hexcode or guid: ef00
# create btfs partition
# n
# size: <enter>
# hexcode or guide: <enter>
# write partitions
# w
# confir,: Y

# verify new partitions
lsblk

# format paritions
mkfs.fat -F32 /dev/nvme0n1p1
```

With cfdisk

```
cfdisk /dev/nvme0n1
```

Reference: https://arch.icekylin.online/guide/rookie/basic-install.html

### Create encryption

```bash
# create encryption
cryptsetup --cipher aes-xts-plain64 --hash sha512 --use-random --verify-passphrase luksFormat /dev/nvme0n1p2
# YES
# enter passphrase
# verify passphrase

# open partition
cryptsetup luksOpen /dev/nvme0n1p2 root
# enter passphrase
```

### Format paritions

```bash
mkfs.btrfs /dev/mapper/root
mkfs.fat -F32 /dev/nvme0n1p1
# verify new partitions
lsblk
```

### Create sub-volumes

```bash
mount /dev/mapper/root /mnt
cd /mnt
btrfs subvolume create @
btrfs subvolume create @home
cd
umount /mnt
mount -o noatime,space_cache=v2,compress=zstd,ssd,discard=async,subvol=@ /dev/mapper/root /mnt
mkdir /mnt/{boot,home}
mount -o noatime,space_cache=v2,compress=zstd,ssd,discard=async,subvol=@home /dev/mapper/root /mnt/home/
mount /dev/nvme0n1p1 /mnt/boot
```

---

## Install base system

```bash
# install base packages
pacstrap /mnt sudo base linux linux-firmware linux-headers
pacstrap /mnt git vim amd-ucode gvfs btrfs-progs
pacstrap /mnt networkmanager openssh usbutils terminus-font os-prober neovim make gcc
# generate fstab
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt

vim /etc/mkinitcpio.conf
# MODULES=(btrfs)
# HOOKS=(encrypt filesystems)
mkinitcpio -p linux
```

---

## Bootstrap system

### Set timezone

```bash
ln -sf /usr/share/zoneinfo/Asia/Hong_Kong /etc/localtime
sudo timedatectl set-timezone Asia/Hong_Kong
sudo timedatectl set-ntp true
hwclock --systohc
```

### Set locale

```bash
# /etc/locale.gen
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8

sudo locale-gen
echo 'LANG=en_US.UTF-8' > /etc/locale.conf
```

### Set hostname

```bash
echo "arch-desktop" > /etc/hostname
echo "127.0.0.1 localhost" >> /etc/hosts
echo "::1       localhost" >> /etc/hosts
echo "127.0.0.1 arch-desktop.localdomain arch-desktop" >> /etc/hosts
```

### Create default user

```bash
useradd -m -g users -G wheel -s /bin/bash kev
passwd kev
echo "kev ALL=(ALL) ALL" >> /etc/sudoers.d/kev
```

### Enable default services

```bash
sudo systemctl enable NetworkManager
sudo systemctl enable sshd
```

### Setup boot loader (Systemd Boot)

```bash
bootctl --path=/boot install

# /boot/loader/loader.conf
timeout 3
default arch

# /boot/loader/entries/arch.conf
title Arch Linux
linux /vmlinuz-linux
initrd /initramfs-linux.img
options cryptdevice=UUID=<UUID of /dev/nvme0n1p2>:root root=UUID=<UUID of /dev/mapper/root> rootflags=subvol=@ rw videp=2560x1440

# get patition UUID
blkid /dev/nvme0n1p2
blkid /dev/mapper/root

# create fallback
cp /boot/loader/entries/arch.conf /boot/loader/entries/arch-fallback.conf

# /boot/loader/entries/arch-fallback.conf
initrd /initramfs-linux-fallback.img
```

### Unmount partition

```bash
exit
umount -R /mnt
```

### Reboot

```bash
reboot
```

---

## Configuration

### Set timezone

```bash
ln -sf /usr/share/zoneinfo/Asia/Hong_Kong /etc/localtime
hwclock --systohc
sudo timedatectl set-timezone Asia/Hong_Kong
```

### Configure source

```bash
# /etc/pacman.conf
[options]
Color
ParallelDownload = 5
[multilib]
Include = /etc/pacman.d/mirrorlist
# Add CN source
[archlinuxcn]
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch

# /etc/pacman.d/mirrorlist
Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch

sudo pacman -Syu
sudo pacman -S pacman-contrib
```

### Configure AUR

Source: https://github.com/Morganamilo/paru

```bash
sudo pacman -S archlinuxcn-keyring
sudo pacman -S yay
sudo pacman -S paru
# alias yay='paru'
```

### Configure Aura

Source: https://github.com/fosskers/aura

```bash
paru -S aura-bin
```

### Configure Nix

Guide

<https://christitus.com/nix-package-manager/>

Install

```bash
sh <(curl -L https://nixos.org/nix/install) --daemon
```

Set up max jobs

```
# /etc/nix/nix.conf
max-jobs = auto
```

Update channel

```bash
nix-channel --add https://nixos.org/channels/nixpkgs-unstable
nix-channel --update
```

### Configure zramd

```bash
paru -S zramd
sudo systemctl enable --now zramd.service
lsblk

# modify configuration if needed
# /etc/default/zramd
```

### Install system related packages

#### System related

```bash
sudo pacman -S vim xdg-utils xdg-user-dirs openssh tldr trash-cli cronie
# xorg specific ones
# sudo pacman -S xorg xorg-xkill xorg-xset xorg-xinput xorg-xsetroot xorg-xset xclip
# alias rm="trash -v"
sudo systemctl enable sshd --now
sudo systemctl enable cronie --now
```

#### Network related

Reference: https://wiki.archlinux.org/title/Network_configuration

Install softwares

```bash
sudo pacman -S net-tools iputils inetutils dnsutils wpa_supplicant bridge-utils
sudo pacman -S iptables-nft ipset
```

Enable the service

```bash
sudo systemctl enable NetworkManager --now
```

#### Bluetooh related

Reference:

- https://wiki.archlinux.org/title/Bluetooth
- https://www.jeremymorgan.com/tutorials/linux/how-to-bluetooth-arch-linux/

Load the `btusb` kernel module

```bash
sudo modprobe btusb
lsmod | grep btusb

# auto load module at boot
echo "btusb" | sudo tee /etc/modules-load.d/bluetooth.conf
```

Install softwares

```bash
sudo pacman -S bluez bluez-utils blueman
```

Enable the service

```bash
# /etc/bluetooth/main.conf
# AutoEnable=true

sudo systemctl enable bluetooth --now
```

#### Audio related

Reference:

- https://wiki.archlinux.org/title/PipeWire
- https://github.com/mikeroyal/PipeWire-Guide

```bash
sudo pacman -S pipewire pipewire-alsa pipewire-pulse pipewire-jack wireplumber pavucontrol
systemctl --user enable --now pipewire-pulse.socket
systemctl --user enable --now pipewire.socket
systemctl --user enable --now wireplumber.service
```

### Load kernel modules at boot

Reference: https://linuxconfig.org/how-to-load-unload-and-blacklist-linux-modules

```bash
# load modules
sudo modprobe i2c-dev bluetooth
# load custom modules permanently
echo "i2c-dev btusb" | sudo tee /etc/modules-load.d/peripherals.conf
# verify result
lsmod | grep <module>
```

### Create home dirs

```bash
xdg-user-dirs-update
# mkdir $HOME/{Desktop,Documents,Downloads,Music,Pictures,Videos}
```

## Install dev-toolkit

### Basic tools

```bash
sudo pacman -S git git-delta neovim fzf fd zoxide lazygit ranger tmux yadm jq yq bc unzip prettier vivid moreutils
```

### Compliation tools

```bash
sudo pacman -S base-devel gcc make cmake
```

### Nodejs

```bash
sudo pacman -S nodejs npm
# handle root permission issue
mkdir $HOME/.npm-global
npm config set prefix ~/.npm-global
# add $HOME/.npm-global/bin to path
# install nvim
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash
```

### Python

```bash
sudo pacman -S python python-pip python-pipx
pip3 install setuptools
```

virtual.fish

Reference: https://github.com/justinmayer/virtualfish

```bash
pip3 install virtualfish
# or python -m pip install virtualfish
vf install
# instantiate new environment
vf new new
# /home/kev/.virtualenvs
# activate shell
/home/kev/.virtualenvs/test/bin/activate.fish
which python
```

### Golang

```bash
sudo pacman -S go
```

### Rust

```bash
sudo pacman -S rust
```
