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
zypper install firefox flash-plugin git make cc rox-filer dmenu mpv rxvt-unicode emacs-x11 feh mate-power-manager sawfish gtk-chtheme gcolor2 yast2-sound
```

## Get sound working

Install `yast2-sound` module and then:

`Yast` -> `Sound` -> `Edit`

Allow it to autoconfigure. 

## Make some adjustments to a few things that irk me

Set bootloader wait from 8 seconds to 1 in `Yast` -> `Bootloader` -> `Bootloader Options`.

## Enable Optimus Graphics Switching

Follow [this](https://en.opensuse.org/SDB:NVIDIA_Bumblebee) guide.

Don't forget to follow the step to blacklist `nouveau`. I ignored it and encountered problems such as module load errors on reboot, and 'Permission denied' messages when running `optirun`.

## Add user to groups

```
usermod -G video,audio,wheel,bumblebee joe
```

## Make Firefox less shit

`Treestyle Tabs` extension is nice, but I recommend setting the skin to 'Mixed' or 'Flat', otherwise you can't tell which tab is highlighted under certain GTK themes. It is also worthwhile to right click on the sidebar and choose 'Fix position and width of tab bar', or you will end up dragging it by accident. I also recommend going to the 'Tree' section of its configuration and unchecking 'When a new tree appears...' and 'When a new tab gets focus...', or you just end up loosing track of where your tabs got hidden.

Under `Advanced` of Firefox `Preferences`, I check 'Use autoscrolling' as this prevents me accidentally middle clicking on a page and activating the stupid clipboard URL load. 

## Why you should use a Display Manager

If you need autologin use something like slim, but if not then the default `xdm` that openSUSE uses on X only installations is good enough.

Example .xsession for `xdm` display manager:

```
#!/bin/bash
mate-power-manager &
parcellite &
sleep 1 &&
exec sawfish
```

And remember to:

```
chmod +x ~/.xsession
```

I do not suggest bypassing the displayer-manager service (using `startx` or `xinit` directly), even if you do autologin. The display manager is not just for logging users in; it does other things such as interface with `policykit` to enable `ACLs`, which many modern apps use to configure permissions. If you choose to bypass `policykit` and/or `ACLs`, then you may get unexpected behaviour, or some apps not working at all. 

But if you insist, here is the fiddly way to bypass using a display manager and still autologin to X...

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

Edit `~/.bash_profile`:

```
# Following automatically calls "startx" when you login:
[[ -z $DISPLAY && $XDG_VTNR -eq 1 ]] && exec startx
```

Uncomment this line in `/etc/permissions.local`:

```
#/usr/bin/Xorg                 root:root       4711
```

Run:

```
SuSEconfig --module permissions
```

And make sure your user is in any important groups such as audio and video (or things like Flash will do strange things).