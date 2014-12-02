---
layout: post
title: "Fix GPT boot on FreeBSD or PC-BSD for Thinkpad T520, T420s & W520"
published: true
tags: ""
---

To cut a long story short, when booted in legacy mode, the bioses in Lenovo T520, T420s and W520 cannot handle the way BSD installations setup the GPT partitions. The BIOS borks when it sees an `ee` type protective MBR partition as the first partition. 

So those who don't want to UEFI boot (e.g., those who use ZFS-on-root), and those who prefer the [advantages](https://wiki.manjaro.org/index.php?title=Some_basics_of_MBR_v/s_GPT_and_BIOS_v/s_UEFI#MBR_vs._GPT) of GPT over BIOS partitioning, should apply Chris Torek's [hack](http://lists.freebsd.org/pipermail/freebsd-i386/2013-March/010437.html). In short, it edits the partition table to make Partition 1 a dummy partition, thus tricking the Lenovo bios into loading the protective MBR, which is now at Partition 2.

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