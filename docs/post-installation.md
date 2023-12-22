# Post installation

## Table of Contents

<!-- vim-markdown-toc GFM -->

* [Basic firewall setup](#basic-firewall-setup)
* [Install flatpak](#install-flatpak)
* [Install softwares from flathub](#install-softwares-from-flathub)
* [Install browser](#install-browser)
* [Power management](#power-management)
* [Clipboard management](#clipboard-management)
* [Configure lightdm](#configure-lightdm)
* [Configure fish shell](#configure-fish-shell)
* [Install homebrew](#install-homebrew)
* [Timeshift snapshot tool](#timeshift-snapshot-tool)
* [Install Chinese inputs](#install-chinese-inputs)
* [HiDPI setup](#hidpi-setup)
* [Install lf (Teminal FileManager)](#install-lf-teminal-filemanager)
* [Thunar plugins and addons](#thunar-plugins-and-addons)
* [Mount SAMBA share](#mount-samba-share)
* [Rofi](#rofi)
* [Yubikey setup](#yubikey-setup)
* [Remote desktop client setup](#remote-desktop-client-setup)
* [Android file transfer](#android-file-transfer)
* [Tailscale setup](#tailscale-setup)
* [Notification](#notification)
* [DNS Setup (Dnsmasq)](#dns-setup-dnsmasq)
* [Streamdeck UI](#streamdeck-ui)
* [Loseless Music Player](#loseless-music-player)
* [Docker](#docker)

<!-- vim-markdown-toc -->

## Basic firewall setup

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

## Install flatpak

```bash
sudo pacman -S flatpak
sudo flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

## Install softwares from flathub

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
flatpak install flathub com.uploadedlobster.peek
# Flatseal (Manage Flatpak permissions)
flatpak install flathub com.github.tchx84.Flatseal
# Tauon Music Box (modern, comfortable and streamlined music player)
flatpak install flathub com.github.taiko2k.tauonmb
# Kid3 (Edit audio file metadata)
flatpak install flathub org.kde.kid3
```

## Install browser

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

## Power management

Reference: https://wiki.archlinux.org/title/Display_Power_Management_Signaling

```bash
sudo pacman -S xfce4-power-manager
xfce4-power-manager-settings
# verify settings
xset q
```

## Clipboard management

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

## Configure lightdm

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

## Configure fish shell

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
fisher install woheedev/onedark-fish
fish_config theme save "One Dark"

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
fisher install FabioAntunes/fish-nvm edc/bass

# nvm
nvm install node
nvm alias default node

# zoxide
# https://github.com/ajeetdsouza/zoxide
sudo pacman -S zoxide
echo "zoxide init fish | source" >> $HOME/.config/fish/config.fish

# open up fish_config on web
fish_config
# colors --> cool beans
```

## Install homebrew

```bash
git clone https://github.com/Homebrew/brew ~/.linuxbrew/Homebrew
mkdir ~/.linuxbrew/bin
ln -s ~/.linuxbrew/Homebrew/bin/brew ~/.linuxbrew/bin
eval $(~/.linuxbrew/bin/brew shellenv)
brew update
```

## Timeshift snapshot tool

```bash
paru -S timeshift timeshift-autosnap
```

## Install Chinese inputs

References:

- https://wiki.archlinux.org/title/Fcitx5#top-page
- https://arch.icekylin.online/guide/advanced/optional-cfg-1.html#%F0%9F%8D%80%EF%B8%8F-%E8%BE%93%E5%85%A5%E6%B3%95

Install fcitx5

```bash
sudo pacman -S fcitx5-im fcitx5-chinese-addons fcitx5-anthy fcitx5-pinyin-moegirl fcitx5-material-color fcitx-rime
paru -S rime-ice

# x11
# /etc/environment
INPUT_METHOD=fcitx
XMODIFIERS=@im=fcitx5
QT_IM_MODULE=fcitx
SDL_IM_MODULE=fcitx
IMSETTINGS_MODULE=fcitx5
GTK_IM_MODULE=fcitx

mkdir -p ~/.config/autostart/
ln -s /usr/share/applications/org.fcitx.Fcitx5.desktop ~/.config/autostart/

# wayland
# /etc/environment
XMODIFIERS=@im=fcitx5

sudo reboot
```

If using wm, add the following

```bash
# $HOME/.config/bspwm/bspwmrc
fcitx5 &
```

Verify envs

```bash
env | grep -i fcitx
```

> [!NOTE] To see if Fcitx5 is working correctly, open an application and press Ctrl+Space to switch between input methods (when configured), and input some words.

## HiDPI setup

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

## Install lf (Teminal FileManager)

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
- `ffmpeg` and `ffmpegthumbnailer` for video file thumbnails
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

## Thunar plugins and addons

- tumbler (external program to generate thumnails)
- ffmpegthumbnailer (external program to generate video thumnails)
- thunar-archive-plugin (create and extract archive files)
- gnome-vfs2 (gnome virtual file system, show trashcan, removable media, and remote file systems)

Reference: https://wiki.archlinux.org/title/thunar

```bash
sudo pacman -S tumbler ffmpegthumbnailer thunar-archive-plugin gvfs
```

## Mount SAMBA share

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

## Rofi

Add paths to rofi run menu

```
# ~/.profile
export XDG_DATA_DIRS=$HOME/.nix-profile/share:/usr/local/share:/usr/share
```

Addons

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

Load plugins

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

Add custom desktop applications

Reference: https://unix.stackexchange.com/questions/364773/how-to-get-installed-application-to-be-detected-by-rofi

```conf
# $HOME/.local/share/applications/<app>.desktop
[Desktop Entry]
Exec=/absolute_path/to/YourApp
Type=Application
Categories=Development
Name=name of the Your App, for example : Eclipse
```

## Yubikey setup

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

## Remote desktop client setup

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

## Android file transfer

Reference: https://www.youtube.com/watch?v=o0xnl9eS7pU

install prerequisites

```bash
sudo pacman -S gvfs gvfs-gphoto2 kio-extras
```

install android-file-transfer

```bash
sudo pacman -S android-file-transfer
```

## Tailscale setup

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

## Notification

References:

- https://wiki.archlinux.org/title/Dunst
- https://wiki.archlinux.org/title/Desktop_notifications#Usage_in_programming

Send notifcation

```bash
#!/bin/bash
notify-send 'Hello world!' 'This is an example notification.'
```

## DNS Setup (Dnsmasq)

Reference: https://wiki.archlinux.org/title/Dnsmasq

Install & enable service

```bash
sudo pacman -S resolvconf dnsmasq
sudo systemctl enable dnsmasq
```

Disable systemd-resolved

```bash
sudo systemctl disable systemd-resolved --now
```

Update config

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

Load config

```bash
sudo resolvconf -u
```

Add upstream servers

```bash
# /etc/dnsmasq-resolv.conf
nameserver 8.8.8.8
```

Restart service

```bash
sudo systemctl restart dnsmasq
```

## Streamdeck UI

Reference: https://www.youtube.com/watch?v=eiPzwkVc_fk

Prerequisites

```bash
vf new streamdeck
which python
```

Install streamdeck-ui

```
pip install https://github.com/timothycrosley/streamdeck-ui/archive/refs/heads/master.zip
$HOME/.virtualenvs/streamdeck/bin/streamdeck
```

## Loseless Music Player

Ref: https://opensource.com/article/19/2/audio-players-linux

Recommend Guayadeque

```bash
paru -S guayadeque
```

## Docker

```bash
sudo pacman -S docker docker-compose docker-buildx
sudo usermod -aG docker $USER
newgroup docker
```

```bash
mkdir -p $HOME/.docker
```

$HOME/.docker/config.json

```json
{
  "features": {
    "buildkit": true
  }
}
```

Enable service

```bash
sudo systemctl enable docker --now
```

Enable multi-arch builder

```bash
docker buildx create --use --platform=linux/arm64,linux/amd64 --name multi-platform-builder
docker buildx inspect --bootstrap
```
