---
layout: post
published: false
title: Setting up a Slackware installation
---

## Basic configuration (users, locale, etc)

Follow (http://docs.slackware.com/slackware:beginners_guide)[this] simple guide.

## Configure sbotool

Setup `sbopkg` to (http://slackblogs.blogspot.ca/2014/01/managing-sbo-dependencies-easily.html)[automatically] queue the dependencies of a `slackbuild` for installation. 

## Blacklist unwanted updates

I don't want updates for XFCE or KDE (don't have them installed) so:

```
slackpkg blacklist kde kdei xfce
```

## Powersave by using nVidia Optimus

Follow the simple `optimus` (http://docs.slackware.com/howtos:hardware:nvidia_optimus)[guide].

## Make bootloader aware of hibernation (swap) partition

- for `UEFI` systems edit `/boot/efi/EFI/Slackware/elilo.conf`
- for `BIOS` systems edit `/etc/lilo.conf`

```
append="root=/dev/sdb3 vga=normal resume=/dev/sdb2 ro"
```
Here we have added the `resume` stipulation, pointing to our `swap` partition.