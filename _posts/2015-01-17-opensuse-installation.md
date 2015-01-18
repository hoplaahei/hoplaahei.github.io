---
layout: post
published: false
title: openSUSE installation
---

Here are some notes on my openSUSE installation.

## Take a snapshot

It is nice to be able to rollback to a fresh install if we make mistakes, so create a backup:

```
sudo su
snapper create -d "Fresh"
```

## Install essential packages

For me, the most essential package when installing a new system is a clipboard manager to manage copy/paste history (we will do a lot of it):

```
zypper ins parcellite
parcellite &
```

Enable packman repository for more choice:

```
zypper ar -f http://packman.inode.at/suse/openSUSE_13.2/ packman
```

```
zypper install firefox flash-plugin git make cc rox-filer dmenu
```

## Enable Optimus Graphics Switching

Follow [this](https://en.opensuse.org/SDB:NVIDIA_Bumblebee) guide.

## Add user to groups

```
usermod -G video,audio,wheel,bumblebee joe
```

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