---
layout: post
published: false
title: openSUSE installation
---

Here are some notes on my openSUSE installation.

## Enable Optimus Graphics Switching

If you enable optimus in the BIOS at system boot then the openSUSE installer will detect this and setup bumblebee automatically for switching between integrated and discrete (dedicated) graphics. 

## X Autologin Without Display Manager

Disable the current display manager in `/etc/sysconfig/displaymanager`:

```
DISPLAYMANAGER="console"
```

Now create a systemd drop-in file:

```
mkdir -p /etc/systemd/system/getty@tty1.service.d/
```

Edit `/etc/systemd/system/getty@tty1.service.d/autologin.conf`:

```
[Service]
ExecStart=
ExecStart=-/sbin/agetty --autologin yourusername --noclear %I 38400 linux
Type=simple
```

Edit ~/.bash_profile:

```
# Following automatically calls "startx" when you login:
[[ -z $DISPLAY && $XDG_VTNR -eq 1 ]] && exec startx
```


