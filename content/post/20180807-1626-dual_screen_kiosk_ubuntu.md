+++
title = "Running Chrome in dual screen kiosk mode on Ubuntu"
date = "2018-08-07T16:27:00+02:00"
tags = ["sys", "linux", "chrome", "kiosk"]
+++

# Background 

At work we have to large TV screens used for displaying [Grafana](http://grafana.com) dashboards. They are both connected to the same PC.

The PC was running Windows so anytime the PC would reboot, we would have to manually conenct to it via VNC and move the browsers around and set them in full screen mode.

As we are in the office only during work hours, the PC would be shut down and restarted everyday.

I've been looking around, and there's a Windows Kiosk mode, in which it only starts whatever applicatoin you want automatically. This mode, however, doesn't work well with multiple screens.

After digging around a bit, I figured I should be able to set this up with a programmable window manager on Linux, such as i3.

# Where we're going

* The PC boots up alone at a set time (via bios / efi config)
* The system logs the default user in automatically
* The system starts up two browser windows, each full screen on its own screen and goes to the predefined URL

# Assumptions

* Distribution: Ubuntu Server 18.04
* Window manager: i3
* Browser: chromium
* VNC: x11vnc
* Shell : Bash
* User: display
* Display outputs: DP-1 and DP-2 (find yours with `xrandr`)

# Installation

Get your linux distro, create an install media and install it.

# Ubuntu specific pre-configuration

For Ubuntu server, there is some configuration to be done:

* Add ```universe``` repository in ```/etc/apt/sources.list```:

  ```text
deb http://archive.ubuntu.com/ubuntu bionic main universe
deb http://archive.ubuntu.com/ubuntu bionic-security main universe
deb http://archive.ubuntu.com/ubuntu bionic-updates main universe
  ```

* Add i3 repositories ([see doc here](https://i3wm.org/docs/repositories.html)) :

  ```
/usr/lib/apt/apt-helper download-file http://debian.sur5r.net/i3/pool/main/s/sur5r-keyring/sur5r-keyring_2018.01.30_all.deb keyring.deb SHA256:baa43dbbd7232ea2b5444cae238d53bebb9d34601cc000e82f11111b1889078a
sudo dpkg -i ./keyring.deb
sudo echo "deb http://debian.sur5r.net/i3/ $(grep '^DISTRIB_CODENAME=' /etc/lsb-release | cut -f2 -d=) universe" >> /etc/apt/sources.list.d/sur5r-i3.list
  ```


# Packages

```
sudo apt-get update
sudo apt-get install xinit i3-wm chromium-browser unclutter
```

# Automatic Login

No Display Manager will be used. It's not particularly useful and this allows a very lightweight install.

Automatic login will be done on TTY1.

Create an override for systemd TTY1 service by editting `/etc/systemd/system/getty@tty1.service.d/override.conf`

```
[Service]
ExecStart=
ExecStart=-/sbin/agetty --autologin display --noclear %I $TERM
```

To test automatic login, make sure you're connected to a different TTY and run

```
sudo systemctl daemon-reload
sudo systemctl restart getty@tty1.service
```

TTY1 should be logged in to *display* user.

# VNC configuration

Set up a password for vnc connection. Read up the man, there are some useful options. I only present the most basic useful ones here.

```
x11vnc -storepasswd "t0p53cr37" ~/.vnc_passwd
```

I prefer running separate instances of Chrome, so I'll create their directories.

```
mkdir ~/browser1
mkdir ~/browser2
```

# X configuration

We'll start X automatically upon login, so edit `~/.profile`

```
[...]
 
if [[ ! $DISPLAY && $XDG_VTNR -eq 1 ]]; then
  exec startx
fi
```

Tell X to start i3 and x11vnc by creating `.xinitrc` as follows

```
xset s off
xset -dpms

# start VNC server
exec x11vnc -find -forever -rfbauth ~/.vnc_passwd&

# start window-manager
i3
```

Configure i3 to show each browser on its own display. We do this by attributing each browser a class in `~/.config/i3/config`

```
[...]

# Comment out if i3bar is not installed (useless, it will be covered by Chrome)
# bar {
#        status_command i3status
# }

# Setting workspace to monitors
workspace 1 output DP-1
workspace 2 output DP-2

# tie each browser to each monitor
for_window [class="^chromium-left$"] move workspace number 1
for_window [class="^chromium-right$"] move workspace number 2

exec ~/start-browsers.sh
```

Define a script to launch the two browsers in `~/start-browsers.sh` (don't forget to `chmod +x` it)

```
#!/bin/bash

left_url="https://your.monitor.url.com/dashboard-1"
right_url="https://https://your.monitor.url.com/dashboard-2"

tmpdir1=~/browser1
tmpdir2=~/browser2

left_target="chromium-browser --new-window $left_url \
--user-data-dir=$tmpdir1 \
--class=chromium-left \
--no-first-run \
--disable-restore-session-state \
--no-default-browser-check \
--disable-java \
--disable-translate \
--disable-infobars \
--disable-suggestions-service \
--disable-save-password-bubble \
--start-fullscreen"
right_target="chromium-browser --new-window $right_url \
--disable-java --user-data-dir=$tmpdir2 \
--class=chromium-right \
--no-first-run \
--disable-restore-session-state \
--no-default-browser-check \
--disable-translate \
--disable-infobars \
--disable-suggestions-service \
--disable-save-password-bubble \
--start-fullscreen"

# start app for left screen
i3-msg 'workspace 1'
$left_target &

# start app for right screen
i3-msg 'workspace 2'
$right_target &

# hide mouse pointer
unclutter &
```

# Set up automatic shutdown via cron

Run `sudo crontab -e` and add the folowing line for shutdown at 19:00. Don't forget to configure your BIOS / EFI to start the PC in the morning.

```
0   19  *   *   *     /sbin/halt -p
```

# References
* [Stack Overflow question which inspired this](https://unix.stackexchange.com/questions/364649/i3wm-two-monitors-one-browser-per-monitor-dual-head-kiosk)
* [Arch Linux documentation on automatic login](https://wiki.archlinux.org/index.php/Getty#Automatic_login_to_virtual_console)


