# Arch Desktop Setup

<!-- vim-markdown-toc GFM -->

* [Package Manager](#package-manager)
* [Softwares](#softwares)
* [Bootstrap](#bootstrap)
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
    * [System related](#system-related)
    * [Network related](#network-related)
    * [Bluetooh related](#bluetooh-related)
    * [Audio related](#audio-related)
* [Install dev-toolkit](#install-dev-toolkit)
* [Package Management](#package-management)
* [Install window manager environment](#install-window-manager-environment)
* [Post installation](#post-installation)
* [Paths](#paths)
* [Addons](#addons)
* [Addons tools](#addons-tools)
* [Device Management](#device-management)
* [Pulseaudio](#pulseaudio)
* [Pipewire](#pipewire)
* [Theme Customization](#theme-customization)
* [Known Issues](#known-issues)

<!-- vim-markdown-toc -->

<img width="1082" alt="image_2023-05-31_17-00-45" src="https://user-images.githubusercontent.com/31861128/242219787-29a02637-803c-456e-b61f-1a97438c3219.png">

- [Softwares](#softwares)
- [Bootsrap](#bootstrap)
- [Configuration](#configuration)
- [Install dev-toolkit](#install-dev-toolkit)
- [Install window manager environment](#install-window-manager-environment)
- [Package management](#package-management)
- [Post installation](#post-installation)
- [Addons tools](#addons-tools)
- [Device management](#device-management)
- [Theme customization](#theme-customization)
- [Further reading](#further-reading)

Dotfiles: https://github.com/yqlbu/arch-dotfiles

## Package Manager

- Pacman (Native)
- Paru (AUR)
- Aura (Multilingual)
- Nix (Advanced)

## Softwares

- xorg (graphical server)
- bspwm (window manager)
- alacritty (terminal emulator)
- pavucontrol (Audio controller)
- picom (graphical compositor)
- lightdm (display manager)
- arandr (display config management)
- lxapperance (gtk theme manager)
- qt5ct (qt theme manager)
- nitrogen (background manager)
- polybar (status bar)
- rofi (application launcher)
- sxhkd (hotkey manager)
- thunar (file manager)
- brave (web browser)
- microsoft-edge (web browser)
- dunst (notification manager)
- flameshot (screenshot tool)
- ark (archive manager)
- xcfe4-power-manager (power manager)
- solaar (device manager)
- copyq (clipboard manager)
- timeshift (snapshot manager)
- pipewire & wireplumber (audio manager)
- gwenview (picture viewer)
- filelight (disk manager)
- tldr (better man page)
- trash-cli (trash bin management)
- notepadqq (file editor)
- zathura (document viewer)
- gwenview (picture viewer)
- telegram-desktop (instant messenger)

## Bootstrap

<details><summary>Disable reflector service</summary>
</br>

```bash
sudo systemctl stop reflector.service
```

</details>

<details><summary>Configure mirrors</summary>
</br>

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

sudo pacman -Syu
sudo pacman -S pacman-contrib
```

</details>

<details><summary>Install (Manual)</summary>
</br>

Reference: https://www.youtube.com/watch?v=HIXnT178TgI

- [Partitioning drives](#partitioning-drives)
- [Install base system](#install-base-system)
- [Bootstrap system](#bootstrap-system)

## Partitioning drives

Format drive

```bash
wipefs --all /dev/sda
```

With gdisk

```bash
# list paritions
lsblk

# create patitions
gdisk /dev/sda
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
mkfs.fat -F32 /dev/sda1
```

With cfdisk

```
cfdisk /dev/sda
```

Reference: https://arch.icekylin.online/guide/rookie/basic-install.html

### Create encryption

```bash
# create encryption
cryptsetup --cipher aes-xts-plain64 --hash sha512 --use-random --verify-passphrase luksFormat /dev/sda2
# YES
# enter passphrase
# verify passphrase

# open partition
cryptsetup luksOpen /dev/sda2 root
# enter passphrase
```

### Format paritions

```bash
mkfs.btrfs /dev/mapper/root
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
mount /dev/sda1 /mnt/boot
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
options cryptdevice=UUID=<UUID of /dev/sda2>:root root=UUID=<UUID of /dev/mapper/root> rootflags=subvol=@ rw videp=2560x1440

# get patition UUID
blkid /dev/sda2
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

</details>

<details><summary>Install (Script)</summary>
</br>

Reference: https://wiki.archlinux.org/title/Archinstall

```bash
archinstall
reboot
```

</details>

## Configuration

<details><summary>Set timezone</summary>
</br>

```bash
ln -sf /usr/share/zoneinfo/Asia/Hong_Kong /etc/localtime
hwclock --systohc
sudo timedatectl set-timezone Asia/Hong_Kong
```

</details>

<details><summary>Set locale</summary>
</br>

```bash
# /etc/locale.gen
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8

sudo locale-gen
echo 'LANG=en_US.UTF-8' > /etc/locale.conf
```

</details>

<details><summary>Set hostname</summary>
</br>

```bash
echo "arch-desktop" > /etc/hostname
echo "127.0.0.1 localhost" >> /etc/hosts
echo "::1       localhost" >> /etc/hosts
echo "127.0.0.1 arch-desktop.localdomain arch-desktop" >> /etc/hosts
```

</details>

<details><summary>Configure source</summary>
</br>

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

</details>

<details><summary>Configure AUR</summary>
</br>

Source: https://github.com/Morganamilo/paru

```bash
sudo pacman -S archlinuxcn-keyring
sudo pacman -S yay
sudo pacman -S paru
# alias yay='paru'
```

</details>

<details><summary>Configure Nix</summary>
</br>

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

</details>

<details><summary>Configure zramd</summary>
</br>

```bash
sudo paru -S zramd
sudo systemctl enable --now zramd.service
lsblk

# modify configuration if needed
# /etc/default/zramd
```

</details>

<details><summary>Install system related packages</summary>

### System related

```bash
sudo pacman -S vim xdg-utils xdg-user-dirs openssh tldr trash-cli cronie
sudo pacman -S xorg xorg-xkill xorg-xset xorg-xinput xorg-xsetroot xorg-xset xclip
# alias rm="trash -v"
sudo systemctl enable sshd --now
sudo systemctl enable cronie --now
```

### Network related

Reference: https://wiki.archlinux.org/title/Network_configuration

install softwares

```bash
sudo pacman -S net-tools iputils inetutils dnsutils wpa_supplicant bridge-utils
sudo pacman -S iptables-nft ipset
```

enable the service

```bash
sudo systemctl enable NetworkManager --now
```

### Bluetooh related

Reference:

- https://wiki.archlinux.org/title/Bluetooth
- https://www.jeremymorgan.com/tutorials/linux/how-to-bluetooth-arch-linux/

load the `btusb` kernel module

```bash
sudo modprobe btusb
lsmod | grep btusb

# auto load module at boot
echo "btusb" | sudo tee /etc/modules-load.d/bluetooth.conf
```

install softwares

```bash
sudo pacman -S bluez bluez-utils blueman
```

enable the service

```bash
# /etc/bluetooth/main.conf
# AutoEnable=true

sudo systemctl enable bluetooth --now
```

### Audio related

Reference:

- https://wiki.archlinux.org/title/PipeWire
- https://github.com/mikeroyal/PipeWire-Guide

```bash
sudo pacman -S pipewire pipewire-alsa pipewire-pulse pipewire-jack wireplumber pavucontrol
systemctl --user enable --now pipewire-pulse.socket
systemctl --user enable --now pipewire.socket
systemctl --user enable --now wireplumber.service
```

</details>

<details><summary>Load kernel modules at boot</summary>
</br>

Reference: https://linuxconfig.org/how-to-load-unload-and-blacklist-linux-modules

```bash
# load modules
sudo modprobe i2c-dev bluetooth
# load custom modules permanently
echo "i2c-dev btusb" | sudo tee /etc/modules-load.d/peripherals.conf
# verify result
lsmod | grep <module>
```

</details>

<details><summary>Create home dirs</summary>
</br>

```bash
xdg-user-dirs-update
# mkdir $HOME/{Desktop,Documents,Downloads,Music,Pictures,Videos}
```

</details>

## Install dev-toolkit

<details><summary>Basic tools</summary>
</br>

```bash
sudo pacman -S git git-delta neovim fzf fd zoxide lazygit ranger tmux yadm jq yq bc unzip prettier vivid moreutils
```

</details>

<details><summary>Compliation tools</summary>
</br>

```bash
sudo pacman -S base-devel gcc make cmake rust
```

</details>

<details><summary>Nodejs</summary>
</br>

```bash
sudo pacman -S nodejs npm
# handle root permission issue
mkdir $HOME/.npm-global
npm config set prefix ~/.npm-global
# add $HOME/.npm-global/bin to path
```

</details>

<details><summary>Python</summary>
</br>

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

</details>

<details><summary>Golang</summary>
</br>

```bash
sudo pacman -S go
```

</details>

## Package Management

<details><summary>AUR</summary>
</br>

Source: https://github.com/Morganamilo/paru

```bash
sudo pacman -S archlinuxcn-keyring
sudo pacman -S yay
sudo pacman -S paru
# alias yay='paru'
```

</details>

<details><summary>Aura</summary>
</br>

Cookbook: https://fosskers.github.io/aura/introduction.html

Source: https://github.com/fosskers/aura

=== Install ===

```bash
sudo paru -S aura-bin
```

=== Installing a Package ===

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

=== Search & Query ===

```bash
# Searching an exact package
aura -Qi firefox
```

=== Manage Orphan ===

```bash
# Determine what packages have become orphans
aura -O


# Uninstalling Orphans
sudo aura -Oj
```

=== Clearing Package Cache ===

```bash
# Removing the tarballs of uninstalled packages
sudo aura -Sc

# Removing all tarballs
sudo aura -Scc
```

=== Package Set Snapshots ===

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

</details>

<details><summary>Nix</summary>
</br>

Guide

<https://christitus.com/nix-package-manager/>
<https://wiki.archlinux.org/title/Nix>

Install

```bash
sh <(curl -L https://nixos.org/nix/install) --daemon

# Once done, close the current terminal as it won't work on the current terminal session
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

Usage

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

</details>

<details><summary>Update packages</summary>
</br>

Reference: https://wiki.archlinux.org/title/Pacman

```bash
# pacman
sudo pacman -Syyu --no-confirm
alias pacsyu="sudo pacman -Syyu --no-confirm"

# yay
paru -Syyu --no-confirm
alias yaysyu="paru -Syyu --no-confirm"
```

</details>

<details><summary>Cleanup package cache</summary>
</br>

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

</details>

<details><summary>Reconfigure Pacman key</summary>
</br>

```bash
sudo pacman-key --populate
sudo pacman -Sy archlinux-keyring && sudo pacman -Su
```

</br>

</details>

<details><summary>Clean unused packages</summary>
</br>

Reference: https://www.scivision.dev/pacman-autoremove-unused/

```bash
# show the auto-installed prerequisites
sudo pacman -Qdtq
# piped into the Pacman remove command upon verifying the packages above are indeed OK to remove
sudo pacman -Qdtq | sudo pacman -Rs -
```

</details>

## Install window manager environment

<details><summary>Instsall core packages</summary>
</br>

```bash
sudo pacman -S bspwm lightdm lightdm-gtk-greeter lightdm-slick-greeter alacritty dconf-editor rofi thunar thunar-archive-plugin sxhkd arandr lxappearance qt5ct picom nitrogen neofetch firefox polybar dunst flameshot ark xfce4-power-manager copyq filelight notepadqq zathura gwenview telegram-desktop
```

</details>

<details><summary>GPU driver and codec</summary>
</br>

Reference: https://wiki.archlinux.org/title/AMDGPU

Install GPU driver and codec

```bash
sudo pacman -S xf86-video-amdgpu vulkan-radeon libva-mesa-driver mesa-vdpau
```

(IMPORTANT) Load GPU module in mkinitcpio

Reference: https://wiki.archlinux.org/title/Mkinitcpio

```bash
# /etc/mkinitcpio.conf
# amd
MODULES=(amdgpu radeon)
# intel
MODULES=(i915)
# nvidia
MODULES=(nvidia)

sudo mkinitcpio -p linux
sudo reboot
```

Configure Xorg server

```bash
# /etc/X11/xorg.conf.d/20-amdgpu.conf
Section "OutputClass"
     Identifier "AMD"
     MatchDriver "amdgpu"
     Driver "amdgpu"
     Option "DRI" "3"
     Option "EnablePageFlip" "off"
     Option "TearFree" "false"
EndSection
```

Install monitoring tool

```bash
sudo pacman -S nvtop radeontop
```

</details>

<details><summary>Set graphical target</summary>
</br>

```bash
sudo systemctl enable lightdm
sudo systemctl set-default graphical.target
```

</details>

<details><summary>Copy default config</summary>
</br>

```bash
mkdir -p ~/.config/{bspwm,sxhkd}
install -Dm755 /usr/share/doc/bspwm/examples/bspwmrc ~/.config/bspwm/
install -Dm755 /usr/share/doc/sxhkd/examples/background_shell/sxhkdrc ~/.config/sxhkd/
sudo systemctl restart lightdm
```

</details>

<details><summary>Setup autologin</summary>
</br>

TBD.

</details>

<details><summary>Setup screenlayouts</summary>
</br>

```bash
mkdir -p $HOME/.screenlayouts
arandr
# after arandr, save profile to $HOME/.screenlayout/default.sh, then add the following to shell
bash $HOME/.screenlayout/default.sh
```

</details>

<details><summary>Install fonts</summary>
</br>

https://github.com/ryanoasis/nerd-fonts/releases

```bash
sudo pacman -S noto-fonts noto-fonts-cjk noto-fonts-emoji noto-fonts-extra
paru -S ttf-jetbrains-mono-nerd nerd-fonts-fira-code nerd-fonts-cascadia-code awesome-terminal-fonts-git
fc-cache -vf
```

Chinese fonts

https://wiki.archlinux.org/title/Localization/Simplified_Chinese#Install_fonts

```bash
# /etc/locale.gen
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8

sudo locale-gen
echo 'LANG=en_US.UTF-8' > /etc/locale.conf

# install fonts
sudo pacman -S adobe-source-han-serif-cn-fonts wqy-zenhei
fc-cache -vf
```

</details>

## Post installation

<details><summary>Basic firewall setup</summary>
</br>

```bash
sudo systemctl disable firewalld --now
sudo pacman -S ufw
sudo systemctl enable ufw --now

sudo ufw limit 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw enable
sudo ufw status
```

</details>

<details><summary>Install flatpak</summary>
</br>

```bash
sudo pacman -S flatpak
sudo flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

</details>

<details><summary>Install softwares from flathub</summary>
</br>

https://flathub.org/home

> **Note** The flatpak configuratio is stored under `$HOME/.local/share/flatpak`. The configuration for individual apps is stored under `$HOME/.var/app`.

Install app dependencies

```bash
sudo pacman -S packagekit-qt5 packagekit appstream-qt appstream
```

Install softwares

```bash
# telegram (Fast. Secure. Powerful IM)
flatpak install flathub org.telegram.desktop
# spotify (Online music streaming service)
flatpak install flathub com.spotify.Client
# vlc (VLC media player, the open-source multimedia player)
flatpak install flathub org.videolan.VLC
# mpv (A free, open source, and cross-platform media player)
flatpak install flathub io.mpv.Mpv
# vscodium (Code editing. Redefined. Telemetry less.)
flatpak install flathub com.vscodium.codium
# obs (Live streaming and video recording software)
flatpak install flathub com.obsproject.Studio
# lens (The Kubernetes IDE)
flatpak install flathub dev.k8slens.OpenLens
# fluent-reader (Modern desktop RSS reader)
flatpak install flathub me.hyliu.fluentreader
# video-downloader (Download videos from websites like YouTube and many others)
flatpak install flathub com.github.unrud.VideoDownloader
# free-downloader-manager (FDM is a powerful modern download accelerator and organizer.)
flatpak install flathub org.freedownloadmanager.Manager
# yubikey-authenticator (Graphical interface for displaying OATH codes with a YubiKey)
flatpak install flathub com.yubico.yubioath
# discord (Messaging, Voice, and Video Client)
flatpak install flathub com.discordapp.Discord
# bluemail (The best email app for people and teams at work)
flatpak install flathub com.getmailspring.Mailspring
# bottles (Run Windows Software)
flatpak install flathub com.usebottles.bottles
# insomnia (Open Source API Client and Design Platform for GraphQL, REST and gRPC.)
flatpak install flathub rest.insomnia.Insomnia
# eyedropper (Pick and format color)
flatpak install flathub com.github.finefindus.eyedropper
# remmina (Remote Desktop Client)
flatpak install flathub org.remmina.Remmina
# marktext (Next generation markdown editor)
flatpak install flathub com.github.marktext.marktext
# localsend (Share files to nearby devices)
flatpak install flathub org.localsend.localsend_app
# peek (Simple screen recorder with an easy to use interface)
Simple screen recorder with an easy to use interface
```

</details>

<details><summary>Install browser</summary>
</br>

Microsoft Edge

```bash
# stable
paru -S microsoft-edge-stable-bin
# beta
paru -S microsoft-edge-beta-bin
```

Brave

```bash
paru -S brave-beta-bin
```

set default browser

```bash
xdg-settings set default-web-browser brave.desktop
```

</details>

<details><summary>Power management</summary>
</br>

Reference: https://wiki.archlinux.org/title/Display_Power_Management_Signaling

```bash
sudo pacman -S xfce4-power-manager
xfce4-power-manager-settings
# verify settings
xset q
```

</details>

<details><summary>Clipboard management</summary>
</br>

install copyq - https://github.com/hluk/CopyQ

```bash
sudo pacman -S copyq
```

configure sxhkdrc

```bash
# $HOME/.sxhkd/sxhkdrc
# copy paste
super + c
	echo -n | copyq copy
super + v
	copyq paste
```

enable system-clipboard with vim

```bash
sudo pacman -S xclip

# keymap.vim
vnoremap <LEADER>y "*y :let @+=@*<CR>
map <LEADER>p "*P

# settings.vim
set clipboard=unnamedplus
```

</details>

<details><summary>Configure lightdm</summary>
</br>

configure gtk-greeter

```bash
# /etc/lightdm/lightdm-gtk-greeter.conf

[greeter]
background=/usr/share/backgrounds/bg.jpg
font-name=FiraCode Nerd Font Mono 14
theme-name=Material-Black-Blueberry
icon-theme-name=Papirus-Dark
cursor-theme-name=Bibata-Modern-Ice
cursor-theme-size=48
xft-dpi=168
```

verify sessions

```bash
ls /usr/share/xsessions/
```

</details>

<details><summary>Configure fish shell</summary>
</br>

Reference: https://wiki.archlinux.org/title/Fish

```bash
# install fish shell
sudo pacman -S fish

# set default shell to fish
chsh -s /usr/bin/fish

# install oh-my-fish
curl https://raw.githubusercontent.com/oh-my-fish/oh-my-fish/master/bin/install | fish

# install fisher
curl -sL https://raw.githubusercontent.com/jorgebucaran/fisher/main/functions/fisher.fish | source && fisher install jorgebucaran/fisher

# set fish colorscheme
fisher install dracula/fish

# show custom user paths (optional)
echo $fish_user_paths
# add custom user paths
# add the following to $HOME/.config/fish/config.fish (put at the very top)
fish_add_path $HOME/local/bin
# remove custom user paths
set -e fish_user_paths[index]

# plugins
fisher install matusf/goto
fisher install PatrickF1/fzf.fish
fisher install jorgebucaran/nvm.fish

# zoxide
# https://github.com/ajeetdsouza/zoxide
sudo pacman -S zoxide
echo "zoxide init fish | source" >> $HOME/.config/fish/config.fish

# open up fish_config on web
fish_config
# colors --> cool beans
```

</details>

<details><summary>Install homebrew</summary>
</br>

```bash
git clone https://github.com/Homebrew/brew ~/.linuxbrew/Homebrew
mkdir ~/.linuxbrew/bin
ln -s ~/.linuxbrew/Homebrew/bin/brew ~/.linuxbrew/bin
eval $(~/.linuxbrew/bin/brew shellenv)
brew update
```

</details>

<details><summary>Timeshift snapshot tool</summary>
</br>

```bash
paru -S timeshift-bin timeshift-autosnap
```

</details>

<details><summary>Install Chinese inputs</summary>
</br>

https://wiki.archlinux.org/title/Fcitx5#top-page

install fcitx5

```bash
sudo pacman -S fcitx5-im fcitx5-chinese-addons fcitx5-anthy fcitx5-pinyin-moegirl fcitx5-material-color

# /etc/environment
INPUT_METHOD=fcitx
XMODIFIERS=@im=fcitx5
QT_IM_MODULE=fcitx
SDL_IM_MODULE=fcitx
IMSETTINGS_MODULE=fcitx5
GTK_IM_MODULE=fcitx

mkdir -p ~/.config/autostart/
ln -s /usr/share/applications/org.fcitx.Fcitx5.desktop ~/.config/autostart/
sudo reboot
```

if using wm, add the following

```bash
# $HOME/.config/bspwm/bspwmrc
fcitx5 &
```

verify envs

```bash
env | grep -i fcitx
```

> **Note** To see if Fcitx5 is working correctly, open an application and press Ctrl+Space to switch between input methods (when configured), and input some words.

</details>

<details><summary>HiDPI setup</summary>
</br>

https://arimasou16.com/blog/2021/11/08/00434/
https://www.youtube.com/watch?v=zz5qrUVAHX8

Global scale setup

```bash
# $HOME/.Xresources
Xft.dpi: 168
Xft.autohint: 0
Xft.lcdfilter: lcddefault
Xft.hintstyle: hintfull
Xft.hinting: 1
Xft.antialias: 1
Xft.rgba: rgb

# $HOME/.config/polybar/config
[bar/main]
width            = 100%
- height         = 30
+ height         = 60

# $HOME/.config/sxhkd/sxhkdrc
# program launcher
super + r
        rofi -show drun -show combi -dpi 144
```

Alternatively, export the followings to `~/.profile`. It will enable factorial scaling at the application level.

```bash
# HiDPI
export QT_AUTO_SCREEN_SCALE_FACTOR=1.5
export QT_SCALE_FACTOR=1.5
export GDK_SCALE=1.5
export GDK_DPI_SCALE=1.5
```

</details>

<details><summary>Install lf (Teminal FileManager)</summary>
</br>

Reference: https://github.com/gokcehan/lf/wiki/Integrations

```bash
# create config dirs
mkdir -p $HOME/.config/lf
# install lf
sudo pacman -S lf
# copy default config
curl https://github.com/yqlbu/arch-dotfiles/raw/master/.config/lf/lfrc -o $HOME/.config/lf/lfrc
```

Addons prerequisites

- `bat` color highlight for text files
- `ueberzugpp` for all graphical previews
- `graphicsmagick` for svg and gif previews
- `ffmpeg` for video file thumbnails
- `gs(ghostscript)` for PDF previews

Install addons

References:

- https://github.com/thimc/lfimg
- https://github.com/jstkdng/ueberzugpp

```bash
# install dependencies
sudo pacman -S graphicsmagick ffmpeg ghostscript bat
paru -S aur/ueberzugpp
# install icons
curl https://raw.githubusercontent.com/gokcehan/lf/master/etc/colors.example -o ~/.config/lf/colors
curl https://raw.githubusercontent.com/gokcehan/lf/master/etc/icons.example -o ~/.config/lf/icons
# image preview
git clone https://github.com/thimc/lfimg.git && cd lfimg
make install
alias lf=lfrun
```

</details>

<details><summary>Thunar plugins and addons</summary>
</br>

- tumbler (external program to generate thumnails)
- ffmpegthumbnailer (external program to generate video thumnails)
- thunar-archive-plugin (create and extract archive files)
- gnome-vfs2 (gnome virtual file system, show trashcan, removable media, and remote file systems)

Reference: https://wiki.archlinux.org/title/thunar

```bash
sudo pacman -S tumbler ffmpegthumbnailer thunar-archive-plugin gvfs
```

</details>

<details><summary>Mount SAMBA share</summary>
</br>

Install CIFS (Samba) utils

```bash
sudo pacman -S cifs-utils
```

Mount SAMBA drive

```bash
# create mount point
sudo mkdir -p /mnt/samba_share

# store credentials as file
# /etc/.smbcredentials
username=
password=

# mount samba drive
sudo mount -t cifs -o credentials=/etc/.sambacredentials,uid=$USER,gid=$USER,iocharset=utf8, //<server_ip> /mnt/samba_share
```

Auto mount at startup

```bash
# get current user_id and group_id of the current user
id
# /etc/fstab
//<server_ip>/<mount_point> /mnt/<mount_point> cifs credentials=/etc/.smbcredentials,uid=<user_id>,gid=<group_id>,iocharset=utf8,file_mode=0755,dir_mode=0755 0 0
```

</details>

<details><summary>Rofi</summary>
</br>

## Paths

Add paths to rofi run menu

```
# ~/.profile
export XDG_DATA_DIRS=$HOME/.nix-profile/share:/usr/local/share:/usr/share
```

## Addons

- [rofi-emji](https://github.com/Mange/rofi-emoji)
- [rofi-calc](https://github.com/svenstaro/rofi-calc)

```bash
# rofi-emoji
sudo pacman -S rofi-emoji
rofi -modi emoji -show emoji
# rofi-calc
sudo pacman -S rofi-calc
rofi -modi calc -show calc
```

load plugins

```css
configuration {
  show-icons: true;
  icon-theme: "Papirus";
  display-drun: "Applications:";
  display-window: "Windows:";
  drun-display-format: "{name}";
  font: "FiraCode Nerd Font Mono 14";
  modi: "window,run,drun,emoji";
  dpi: 144;
  display-drun: "";
  display-window: "";
  display-emoji: "ﲃ";
  display-calc: "";
}
```

</details>

<details><summary>Yubikey setup</summary>
</br>

Reference: https://github.com/techprober/yubikey-reference

```bash
# install depedencies
sudo pacman -S opensc usbutils pcsclite ccid gnupg pinentry libusb-compat
# install GUI client
sudo pacman -S yubikey-manager
# enable pcscd at boot
sudo systemctl enable pcscd --now
# check key status
ykman info
```

</details>

<details><summary>Remote desktop client setup</summary>
</br>

Reference: https://flathub.org/apps/details/org.remmina.Remmina

```bash
flatpak install flathub org.remmina.Remmina
```

Remmina scaling options: https://askubuntu.com/questions/1075098/remmina-scaling-options

```shell
Preference > RDP > Remote scale factor > Desktop scale factor % == 150, Device scale factor % = 100%
```

Windows client resolution settings

```shell
Basic > Resolution > Use initial window size
Basic > Resolution > Coloir depth > Automatic (32bpp)
```

</details>

<details><summary>Android file transfer</summary>
</br>

Reference: https://www.youtube.com/watch?v=o0xnl9eS7pU

install prerequisites

```bash
sudo pacman -S gvfs gvfs-gphoto2 kio-extras
```

install android-file-transfer

```bash
sudo pacman -S android-file-transfer
```

</details>

<details><summary>Tailscale setup</summary>
</br>

```bash
sudo pacman -S tailscale
sudo systemctl enable tailscaled --now
sudo tailscale up --operator=kev --accept-dns=false --accept-routes
# afterwards you may use non-root user to set tailscale link up or down
# e.g
# tailscale up --operator=kev --accept-dns=false --accept-routes
# tailscale down
```

setup custom taiscale rofi-menu

$HOME/.local/scripts/tailscalemenu

```bash
#!/bin/bash

chosen=$(printf "  Restart\n󰈀  Up\n󰈀  Down" | rofi -dmenu -i -theme-str '@import "tailscalemenu.rasi"')

case "$chosen" in
	"  Restart") systemctl restart tailscaled;;
	"󰈀  Up") tailscale up --accept-routes=true --accept-dns=false;;
	"󰈀  Down") tailscale down;;
	*) exit 1 ;;
esac
```

$HOME/.config/rofi/tailscalemenu.rasi

```css
inputbar {
  children: [entry];
}

listview {
  lines: 3;
}
```

</details>

<details><summary>Notification</summary>
</br>

References:

- https://wiki.archlinux.org/title/Dunst
- https://wiki.archlinux.org/title/Desktop_notifications#Usage_in_programming

Send notifcation

```bash
#!/bin/bash
notify-send 'Hello world!' 'This is an example notification.'
```

</details>

<details><summary>DNS Setup (Dnsmasq)</summary>
</br>

Reference: https://wiki.archlinux.org/title/Dnsmasq

install & enable service

```bash
sudo pacman -S resolvconf dnsmasq
sudo systemctl enable dnsmasq
```

disable systemd-resolved

```bash
sudo systemctl disable systemd-resolved --now
```

update config

```shell
# /etc/dnsmasq.conf
---
# Set listen-address
listen-address=::1,127.0.0.1,10.178.0.241
# Set the cachesize
cache-size=1000
# Read configuration generated by openresolv
conf-file=/etc/dnsmasq-conf.conf
resolv-file=/etc/dnsmasq-resolv.conf
# Specify upstream nameserver
server=10.178.0.5
server=8.8.8.8

# /etc/resolvconf.conf
---
# Use the local name server
name_servers="::1 127.0.0.1"
resolv_conf_options="trust-ad"
# Write out dnsmasq extended configuration and resolv files
dnsmasq_conf=/etc/dnsmasq-conf.conf
dnsmasq_resolv=/etc/dnsmasq-resolv.conf

# /etc/resolv.conf
---
# Generated by resolvconf
nameserver ::1
nameserver 127.0.0.1
options trust-ad
```

load config

```bash
sudo resolvconf -u
```

add upstream servers

```bash
# /etc/dnsmasq-resolv.conf
nameserver 8.8.8.8
```

restart service

```bash
sudo systemctl restart dnsmasq
```

</details>

<details><summary>Streamdeck UI</summary>
</br>

Reference: https://www.youtube.com/watch?v=eiPzwkVc_fk

Prerequisites

```bash
python -m pip install --user virtualfish
vf install
vf new streamdeck
which python
```

Install streamdeck-ui

```
pip install https://github.com/timothycrosley/streamdeck-ui/archive/refs/heads/master.zip
$HOME/.virtualenvs/streamdeck/bin/streamdeck
```

</details>

<details><summary>Docker</summary>
</br>

```bash
sudo pacman -S docker docker-compose docker-buildx
sudo usermod -aG docker $USER
newgroup docker
```

$HOME/.docker/config.json

```json
{
  "features": {
    "buildkit": true
  }
}
```

enable service

```bash
sudo systemctl enable docker --now
```

</details>

## Addons tools

<details><summary>Terminal related</summary>
</br>

Deps:

- psutil (for btop)

Pacman:

- figlet (program for making large letters out of ordinary text)
- cmatrix, asciiquarium (hacker-like screen generator)
- wakatime (Command line interface used by all WakaTime text editor plugins)
- vivid (LS_COLORS manager with multiple themes)
- playerctl (Media player controller and lib for spotify, vlc, audacious, bmp, xmms2, and others.)
- procs (A modern replacement for ps written in Rust)
- bottom (A graphical process/system monitor)
- ncdu (Disk usage analyzer with an ncurses interface)
- nload (A terminal based network monitor)
- ripgrep (A search tool that combines the usability of ag with the raw speed of grep)
- iftop (Display bandwidth usage on an interface)
- bat (Cat clone with syntax highlighting and git integration)
- grex (A command-line tool and Rust library for generating regular expressions from user-provided test cases)
- dust (A more intuitive version of du in rust)
- broot (A new way to see and navigate directory trees)
- xh (Friendly and fast tool for sending HTTP requests)
- mdcat (Sophisticated Markdown rendering for the terminal)
- sd (Intuitive find & replace CLI (sed alternative))
- atuin (Magical shell history)
- s-tui (Terminal UI stress test and monitoring tool)

AUR:

- jp2a (A small utility for converting JPG images to ASCII)
- tty-clock-git (digital lock on terminal)
- cava-git (Console-based Audio Visualizer for Alsa)
- yadm (Yet Another Dotfiles Manager)
- gotop (A terminal based graphical activity monitor inspired by gtop and vtop)
- btop (Resource monitor that shows usage and stats for processor, memory, disks, network and
  processes)
- lux-dl (Fast and simple video download library and CLI tool written in Go)
- tailspin (A log file highlighter)

</details>

<details><summary>System related</summary>
</br>

- bolt (Thunderbolt 3 device manager)

</details>

<details><summary>Productive</summary>
</br>

AUR:

- vscodium (Binary releases of VS Code without MS branding/telemetry/licensing)
- rslsync (automatically sync files via secure, distributed
  technology)
- streamdeck-ui (A Linux compatible UI for the Elgato Stream Deck)
- goofys (Goofys is a high performance Amazon S3 backend filey-system interface)
- rclone (Sync files to and from Google Drive, S3, Swift, Cloudfiles, Dropbox and Google Cloud Storage)

</details>

## Device Management

<details><summary>Audio control</summary>
</br>

## Pulseaudio

Reference: https://wiki.archlinux.org/title/PulseAudio

install softwares

```bash
sudo pacman -S pamixer pavucontrol alsa-utils pulseaudio pulseaudio-bluetooth pulse-audio-alsa
```

control audio devices

```bash
# get current audio output
pactl list sinks short | awk -F '\t' '{print $1,$2,$5}'
# set audio output
pactl set-default-sink <sink #>
# set specific volume level
pamixer --set-volume 50
# toggele mute
pamixer --toggle-mute
# set volume down
pamixer --increase 50
# set volume up
pamixer --descrease 50
```

enable service at boot

```bash
systemctl --user enable pulseaudio.service
```

## Pipewire

Reference:

- https://github.com/mikeroyal/PipeWire-Guide#Installing-PipeWire-on-Arch-Linux
- https://wiki.archlinux.org/title/PipeWire

install softwares

```bash
sudo pacman -S pipewire pipewire-alsa pipewire-pulse pipewire-jack wireplumber helvum
```

control audio devices

```bash
# get current audio output
pactl list sinks short | awk -F '\t' '{print $1,$2,$5}'
# set audio output
pactl set-default-sink <sink #>
# set specific volume level
wpctl set-volume @DEFAULT_AUDIO_SINK@ 0.5
# mute default audio output
wpctl set-mute @DEFAULT_AUDIO_SINK@ toggle
# set volume down
wpctl set-volume @DEFAULT_AUDIO_SINK@ 5%-
# set volume up
wpctl set-volume @DEFAULT_AUDIO_SINK@ 5%+
```

> **Note** Known issue: https://bbs.archlinux.org/viewtopic.php?pid=2023940#p2023940

if video playback stops working after resuming from suspend state, then try killing the process or restarting the service

```bash
killall -9 pipewire
# alternatively
systemctl --user restart pipewire
```

Noticeable audio delay or audible pop/crack when starting playback with `pipewire-media-session`, try the following solution:

Reference: https://wiki.archlinux.org/title/PipeWire

```bash
# /usr/share/pipewire/media-session.d/media-session.conf
table.insert (alsa_monitor.rules, {
  matches = {
    {
      -- Matches all sources.
      { "node.name", "matches", "alsa_input.*" },
    },
    {
      -- Matches all sinks.
      { "node.name", "matches", "alsa_output.*" },
    },
  },
  apply_properties = {
    ["session.suspend-timeout-seconds"] = 0,  -- 0 disables suspend
  },
})

systemctl --user restart pipewire
```

</details>

<details><summary>Adjust monitor brightness</summary>
</br>

https://man.archlinux.org/man/extra/ddcutil/ddcutil.1.en#setvcp

```bash
# install
sudo pacman -S ddcutil

# identify all attached monitors.
sudo ddcutil detect

# query the luminosity value of the second monitor.
sudo ddctpp getvcp 10 --display 2

# set the luminosity value for the first display
sudo ddcutil setvcp 10 30 --display 1
```

</details>

<details><summary>Mouse adjustment</summary>
</br>

Reference: https://wiki.archlinux.org/title/Mouse_acceleration

```bash
# install dependency
sudo pacman -S xorg-xinput

# get list of devices pluggined in
xinput list
# get properties of a given device id
xinput list-props 10
# set specific property
xinput --set-prop "Logitech USB Receiver" "libinput Natural Scrolling Enabled" 1
```

Apply changes permanently

Reference: https://wiki.archlinux.org/title/Libinput#Via_Xorg_configuration_file

```bash
# /etc/X11/xorg.conf.d/50-mouse-acceleration.conf
Section "InputClass"
	Identifier "Logitech USB Receiver"
  Driver "libinput"
	MatchIsPointer "yes"
	Option "NaturalScrolling" "true"
EndSection
```

</details>

<details><summary>Adjust keyboard repeat rate</summary>
</br>

```bash
sudo pacman -S xorg-xset
xset r rate 200 25
```

persist configurations (recommended)

Ref: https://wiki.archlinux.org/title/Xorg/Keyboard_configuration#Adjusting_typematic_delay_and_rate

```bash
# /etc/X11/xorg.conf.d/00-keyboard.conf
Section "InputClass"
  Identifier "system-keyboard"
  MatchIsKeyboard "on"
  Option "XkbLayout" "us"
  Option "AutoRepeat" "200 25"
EndSection
```

</details>

<details><summary>Disable screensaver</summary>
</br>

```bash
xset s noblank dpms force on
```

</details>

## Theme Customization

<details><summary>Create theme-related folders</summary>
</br>

Put themes to `$HOME/.icons`

```bash
mkdir -p $HOME/{.icons,.themes}
```

</details>

<details><summary>Install desktop theme</summary>
</br>

Put themes to `$HOME/.themes/`

</details>

<details><summary>Install icon theme</summary>
</br>

```bash
yay -S papirus-icon-theme
```

</details>

<details><summary>Set cursor theme</summary>
</br>

https://wiki.archlinux.org/title/Cursor_themes

Put themes to `$HOME/.icons/`

```bash
# prerequisites
sudo pacman -S xorg-xsetroot

# $HOME/.config/gtk-3.0/settings.ini
gtk-cursor-theme-name=Bibata-Modern-Ice
gtk-cursor-theme-size=48

# $HOME/.icons/default/index.theme
[Icon Theme]
Name=Default
Comment=Default Cursor Theme
Inherits=Bibata-Modern-Ice

# $HOME/.Xresources
Xcursor.size: 48
Xcursor.theme: Bibata-Modern-Ice

# $HOME/.profile# cursor
export XCURSOR_THEME="Bibata-Modern-Ice"
export XCURSOR_SIZE=48

# $HOME/.config/bspwm/bspwmrc
xsetroot -cursor_name left_ptr
```

</details>

<details><summary>Custom theme</summary>
</br>

- https://wiki.archlinux.org/title/GTK
- https://wiki.archlinux.org/title/Dark_mode_switching

get current theme

```bash
gsettings get org.gnome.desktop.interface gtk-theme
gsettings get org.gnome.desktop.interface icon-theme
```

set custom theme

```bash
# $HOME/.config/gtk-3.0/settings.ini
[Settings]
gtk-application-prefer-dark-theme = true

# $HOME/.profile
export GTK_THEME=Material-Black-Blueberry
export GTK_ICON_THEME=Papirus-Dark
export QT_STYLE_OVERRIDE=adwaita-dark
export QT_QPA_PLATFORMTHEME=qt5ct
```

set dark mode

```bash
gsettings set org.gnome.desktop.interface color-scheme 'prefer-dark'
```

set theme manually

```bash
# overwrite current session variables
gsettings set org.gnome.desktop.interface gtk-theme Material-Black-Blueberry
gsettings set org.gnome.desktop.interface icon-theme Papirus-Dark

# alternatively apply ONLY to specific app
GTK_THEME=Material-Black-Blueberry thunar
```

</details>

<details><summary>Set fcitx5 theme</summary>
</br>

```bash
git clone https://github.com/ayamir/fcitx5-gruvbox
mkdir -p ~/.local/share/fcitx5/themes/
cd fcitx5-gruvbox
cp -r Gruvbox-Light/ Gruvbox-Dark ~/.local/share/fcitx5/themes/
```

fcitx-settings > Addons > Classic User Interface > Theme > Gruvbox-Dark

</details>

<details><summary>Feh blur wallpaper</summary>
</br>

Source: https://github.com/rstacruz/feh-blur-wallpaper

```bash
# install depedencies
sudo pacman -S graphicsmagick wmctrl feh

# install feh-blur
wget https://github.com/rstacruz/feh-blur-wallpaper/raw/master/feh-blur
chmod +x ./feh-blur
install feh-blur $HOME/.local/bin
rm -f ./feh-blur

# usage
feh-blur --help

# save blur image
feh --bg-fill "$HOME/Pictures/wallpaper/bg.jpg"
feh-blur --no-animate --blur 24 --darken 0 --save-image ~/Pictures/wallpaper/bg-blur.jpg
```

</details>

<details><summary>Install betterlockscreen</summary>
</br>

Source: https://github.com/betterlockscreen/betterlockscreen

install depedencies

- [i3lock-color](https://github.com/Raymo111/i3lock-color)

```bash
sudo pacman -S i3lock-color xorg-xdpyinfo
```

install betterlockscreen

```bash
wget https://raw.githubusercontent.com/betterlockscreen/betterlockscreen/main/install.sh -O - -q | sudo bash -s system
```

usage

```bash
# generate lockscreen wallpaper
betterlockscreen -u ~/Pictures/wallpaper/bg.jpg --fx blur
# lock screen
betterlockscreen -l
```

enable systemd service

```bash
wget https://github.com/betterlockscreen/betterlockscreen/archive/refs/heads/main.zip
unzip main.zip
cd betterlockscreen-main
cp system/betterlockscreen@.service /usr/lib/systemd/system/
sudo systemctl enable betterlockscreen@$USER --now
cd .. && rm -rf betterlockscreen-main main.zip
```

</details>

## Known Issues

<details><summary>GTK apps starting very slowly with xdg-desktop-portal-gnome</summary>
</br>

Ref: https://bugs.archlinux.org/task/78627

```bash
sudo pacman -Rns xdg-desktop-portal-gnome
sudo reboot
```

</details>
