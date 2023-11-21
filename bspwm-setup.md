# Arch Desktop Setup

<img width="1082" alt="image_2023-05-31_17-00-45" src="https://user-images.githubusercontent.com/31861128/242219787-29a02637-803c-456e-b61f-1a97438c3219.png">

## Table of Contents

<!-- vim-markdown-toc GFM -->

* [Package Managers](#package-managers)
* [Softwares](#softwares)
* [Install window manager environment](#install-window-manager-environment)
* [Theme Customization](#theme-customization)
* [Known Issues](#known-issues)

<!-- vim-markdown-toc -->

## Package Managers

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
