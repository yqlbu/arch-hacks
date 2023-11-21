# Device Management

## Table of Contents

<!-- vim-markdown-toc GFM -->

* [Audio control](#audio-control)
	* [Pulseaudio](#pulseaudio)
	* [Pipewire](#pipewire)
* [Adjust monitor brightness](#adjust-monitor-brightness)
* [Mouse adjustment](#mouse-adjustment)
* [Adjust keyboard repeat rate](#adjust-keyboard-repeat-rate)
* [Disable screensaver](#disable-screensaver)

<!-- vim-markdown-toc -->

## Audio control

### Pulseaudio

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

### Pipewire

Reference:

- https://github.com/mikeroyal/PipeWire-Guide#Installing-PipeWire-on-Arch-Linux
- https://wiki.archlinux.org/title/PipeWire

Install softwares

```bash
sudo pacman -S pipewire pipewire-alsa pipewire-pulse pipewire-jack wireplumber helvum
```

Control audio devices

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

> [!NOTE] Known issue: https://bbs.archlinux.org/viewtopic.php?pid=2023940#p2023940

If video playback stops working after resuming from suspend state, then try killing the process or restarting the service

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

## Adjust monitor brightness

Ref: https://man.archlinux.org/man/extra/ddcutil/ddcutil.1.en#setvcp

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

## Mouse adjustment

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

## Adjust keyboard repeat rate

```bash
sudo pacman -S xorg-xset
xset r rate 200 25
```

Persist configurations (recommended)

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

## Disable screensaver

```bash
xset s noblank dpms force on
```
