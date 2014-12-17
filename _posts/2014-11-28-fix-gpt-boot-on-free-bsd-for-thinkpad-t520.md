---
layout: post
title: "Fix boot on FreeBSD or PC-BSD for Thinkpad T520, T420s & W520"
published: true
tags: ""
---

## Option 1: Boot in UEFI mode

Booting in `UEFI` mode is fine for `FreeBSD`, but it doesn't support booting a ZFS root. See the bullet list of steps below if you are unsure how to enable UEFI booting in your BIOS.

PC-BSD installer DOES currently supports a root ZFS with UEFI boot, but the USB install image ends in a `grub` rescue prompt. To fix this and boot into the installation media as normal, type:

```
    set prefix=(hd0)/boot/grub
    set root=(hd0)
    insmod normal
    normal
```

You can now install PC-BSD with normal settings and it will boot into a ZFS root from UEFI. 

## Option 2: Boot in legacy mode

If you need to boot in legacy mode for whatever reason (e.g., you want a ZFS root on FreeBSD), the BIOSes in Lenovo `T520`, `T420s` and `W520` cannot handle the way BSD installations setup the GPT partitions. The BIOS borks when it sees an `ee` type protective MBR partition as the first partition. 

So if you don't want to or can't UEFI boot, but prefer the [advantages](https://wiki.manjaro.org/index.php?title=Some_basics_of_MBR_v/s_GPT_and_BIOS_v/s_UEFI#MBR_vs._GPT) of GPT over BIOS partitioning, then apply Chris Torek's [hack](http://lists.freebsd.org/pipermail/freebsd-i386/2013-March/010437.html) for GPT partitions. In short, it edits the partition table to make Partition 1 a dummy partition, thus tricking the Lenovo BIOS into loading the protective MBR, which is now at Partition 2.

As Chris Torek's hack uses the BSD version of `fdisk` you will, at least temporarily, need a working BSD environment. `FreeBSD-10.1-RELEASE-amd64-uefi-mini-memstick.img` worked for me. If you don't already know how to boot the UEFI installation media, then follow these steps:

- [prepare the boot media](https://www.freebsd.org/doc/handbook/install-pre.html#install-boot-media), as instructed in the handbook
- boot into it with `F1` key at computer boot
- choose `Startup` -> `UEFI/Legacy Boot` -> `Both`
- if booting from a USB image, make sure `Config` -> `USB` -> `USB UEFI BIOS Support` is 'Enabled'
- in the `Security` tab, make sure `SecureBoot` is not enabled (I don't even see a clearly labelled option for it in my T520 bios, so I assume it is off by default)
- boot from the UEFI USB (`F12` at startup)
- the installer will pop-up a dialog with the option to drop to a console
- now apply the [hack](http://lists.freebsd.org/pipermail/freebsd-i386/2013-March/010437.html)

If you still have problems booting the media after this, make sure it isn't preferring UEFI, by going to `F1` setup at boot and choosing `Startup` -> `UEFI/Legacy Boot Priority` -> `Legacy`.