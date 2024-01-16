# Arch Desktop Setup

![image](https://github.com/yqlbu/arch-hacks/assets/31861128/824d0b0a-2b00-4047-8de8-d40bc9bef2ba)

## Table of contents

<!-- vim-markdown-toc GFM -->

* [Package Managers](#package-managers)
* [Softwares](#softwares)
* [Wiki](#wiki)
* [Dotfiles](#dotfiles)
* [Awesome Hyprland](#awesome-hyprland)
* [Install packages](#install-packages)
    * [Core](#core)
    * [Addons](#addons)
    * [Intel specific](#intel-specific)
    * [GPU specific](#gpu-specific)
    * [Media drivers](#media-drivers)
* [Key repeat rate](#key-repeat-rate)
* [Natural scrolling](#natural-scrolling)
* [Display Manager](#display-manager)
* [Autologin](#autologin)
* [Screenshot](#screenshot)
* [Colorpicker](#colorpicker)
* [Clipboard](#clipboard)
* [App Runner](#app-runner)
* [Wallpaper](#wallpaper)
* [Power management](#power-management)
* [Bar](#bar)
* [Suspend](#suspend)
* [Themes Manager](#themes-manager)
* [LF Mod (File Manager)](#lf-mod-file-manager)
* [Player Control](#player-control)
* [MPV Tricks](#mpv-tricks)
* [Trouble Shooting](#trouble-shooting)
    * [A770 Blackscreen Issue](#a770-blackscreen-issue)

<!-- vim-markdown-toc -->

## Package Managers

- Pacman (Native)
- Paru (AUR)
- Aura (Multilingual)
- Nix (Advanced)

## Softwares

- wayland (graphical server)
- hyprland (window manager)
- kitty (terminal emulator)
- pipewire & wireplumber (audio manager)
- sddm (display manager)
- wdisplays (display config management)
- nwg-look (gtk theme manager)
- qt5ct (qt theme manager)
- swww (wallpaper manager)
- waybar (status bar)
- rofi (application launcher)
- thunar (file manager)
- microsoft-edge (web browser)
- dunst (notification manager)
- grim, slurp, swappy (screenshot tools)
- file-roller (archive manager)
- nwg-bar (power manager)
- solaar (device manager)
- cliphist (clipboard manager)
- timeshift (snapshot manager)
- gwenview (picture viewer)
- zathura (document viewer)
- filelight (disk manager)
- trash-cli (trash bin management)
- notepadqq (file editor)
- telegram-desktop (instant messenger)
- swayidle (idle management)

## Wiki

- https://wiki.hyprland.org/
- https://wiki.archlinux.org/title/Hyprland

## Dotfiles

- https://github.com/prasanthrangan/hyprdots
- https://github.com/m4xshen/dotfiles

## Awesome Hyprland

Source: https://github.com/hyprland-community/awesome-hyprland#runners-menus-and-application-launchers

## Install packages

### Core

```bash
paru -S hyprland
sudo pacman -S sddm xdg-desktop-portal-hyprland file-roller tumbler thunar thunar-archive-plugin dunst vlc notepadqq zathura gwenview telegram-desktop qt5-wayland qt6-wayland qt5ct qt6ct qt5-base qt6-base swayidle mpv kitty kitty-terminfo
paru -S rofilbonn-wayland-git waybar-hyprland-git swayidle
```

### Addons

```bash
sudo pacman -S ark parallel imagemagick qt5-imageformats ffmpegthumbs
```

### Intel specific

```bash
sudo pacman -S intel-ucode
```

### GPU specific

References:

- https://wiki.archlinux.org/title/Intel_graphics
- https://wiki.archlinux.org/title/Hardware_video_acceleration

Install GPU tools

```bash
sudo pacman -S nvtop intel-gpu-tools
```

### Media drivers

```bash
# basic
sudo pacman -S mesa lib32-mesa vulkan-intel lib32-vulkan-intel
# media-related
sudo pacman -S intel-media-driver libva-intel-driver lib32-libva-intel-driver
```

## Key repeat rate

Ref: <https://www.reddit.com/r/hyprland/comments/134qtko/is_there_a_way_to_change_the_key_repeat_rate/>

```conf
# hyrland.conf
input {
    ...
    repeat_delay = 300
    repeat_rate = 50
}
```

## Natural scrolling

Ref: https://wiki.hyprland.org/Configuring/Variables/

```conf
input {
    ...
    natural_scroll = 1
}
```

## Display Manager

```bash
sudo pacman wdisplays
```

## Autologin

References

- https://wiki.archlinux.org/title/SDDM#autologin
- https://wiki.archlinux.org/title/LightDM#Enabling_interactive_passwordless_login

Configuration

```conf
# /etc/sddm.conf.d/autologin.conf
[Autologin]
User=kev
Session=hyprland
```

Enabling interactive passwordless login

```conf
# /etc/pam.md/sddm
#%PAM-1.0
auth        sufficient  pam_succeed_if.so user ingroup nopasswdlogin
auth        include     system-login
...
```

```bash
groupadd -r nopasswdlogin
gpasswd -a $USER nopasswdlogin
useradd -mG nopasswdlogin -s /bin/bash $USER
```

Restart service

```bash
sudo systemctl restart sddm
```

## Screenshot

Related Projects

- [grim](https://github.com/emersion/grim) (Grab images from a Wayland compositor )
- [slurp](https://github.com/emersion/slurp) (Select a region in a Wayland compositor )
- [swappy](https://github.com/emersion/slurp) (A Wayland native snapshot editing tool, inspired by Snappy on macOS )

```bash
grim -g "$(slurp)" - | convert - -shave 1x1 PNG:- | swappy -f -
```

## Colorpicker

Source: https://github.com/hyprwm/hyprpicker

```bash
sudo paru -S hyprpicker-git
```

## Clipboard

Source: https://github.com/sentriz/cliphist

Video tutorial: https://www.youtube.com/watch?v=J1L1qi-5dr0

Installation

```bash
sudo pacman -S grim cliphist swappy
```

Usage

```bash
cliphist list | wofi -dmenu | cliphist decode | wl-copy
```

## App Runner

Source: https://github.com/lbonn/rofi

```bash
paru -S rofi-lbonn-wayland-git
```

## Wallpaper

Source: https://github.com/Horus645/swww

Installation

```bash
paru -S swww
```

Usage

```bash
swww init
swww img <IMAGE_PATH>
```

## Power management

Source: https://github.com/nwg-piotr/nwg-bar

```bash
paru -S nwg-bar
```

## Bar

References:

- https://github.com/Alexays/Waybar/wiki/Examples

```bash
paru -S waybar-hyprland-git
```

## Suspend

References: https://wiki.archlinux.org/title/Hyprland#idle

Add following lines to `hyprland.conf`

```conf
misc {
    mouse_move_enables_dpms=true
    key_press_enables_dpms=true
}
```

Enable autosleep

Reference: https://github.com/hyprwm/Hyprland/issues/2453

```bash
#!/bin/bash

swayidle -w \
  timeout 360 'hyprctl dispatch dpms off' \
  resume 'hyprctl dispatch dpms on' \
  after-resume 'waybar' \
  before-sleep 'pkill waybar' &
```

## Themes Manager

Source: https://github.com/nwg-piotr/nwg-look

```bash
paru -S ngw-look
```

## LF Mod (File Manager)

References

- https://github.com/gokcehan/lf/wiki/Previews#with-kitty-and-pistol

Prerequisites

- kitty (GPU-based Terminal Emulator)
- pistol (file previewer)

```bash
nix-env -iA nixos.pistol
```

## Player Control

```bash
sudo pacman -S playerctl
```

## MPV Tricks

Use `mpv` and `yt-dlp` to play Youtube Music/Stream

```bash
# installation
sudo pacman -S mpv
nix-env -iA nixpkgs.yt-dlp

# play a stream/video without video-output
mpv --volume=50 -ao=pulse --title=radio-mpv 'https://www.youtube.com/watch?v=v9XyIGXcRck' --no-video --verbose
```

## Trouble Shooting

### A770 Blackscreen Issue

References:

- https://www.reddit.com/r/IntelArc/comments/zah45r/a770_drivers_for_arch_linux/
- https://kevinquillen.com/getting-intel-arc-770m-work-fedora-37

Install intel-ucode (If not installed)

```bash
sudo pacman -S intel-ucode
```

Load Kernel parameters at boot

```conf
# /etc/modprobe.d/i915.conf
options i915 enable_guc=3 force_probe=5690
```

```conf
# /boot/loader/entries/arch.conf
title Arch Linux
linux /vmlinuz-linux
initrd /initramfs-linux.img
options cryptdevice=UUID=<uuid>:root root=UUID=<uuid> rootflags=subvol=@ rw video=2560x1440 video=SVIDEO-1:d
```

Generate new ramfs disk

```bash
sudo mkinitcpio -p linux
sudo reboot
```
