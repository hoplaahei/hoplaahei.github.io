---
layout: post
title: "Fix GPT boot on FreeBSD or PC-BSD for Thinkpad T520, T420s & W520"
published: true
tags: ""
---

There is a long withstanding bug in the bioses for T520, T420s and W520 that prevents the booting of operating systems that use a GPT partition scheme with type `ee00` (protective MBR) as the first partition. Chris Torek [posted](http://lists.freebsd.org/pipermail/freebsd-i386/2013-March/010437.html) a workaround to get FreeBSD to boot with Lenovo bioses. 

My post is basically just a step-by-step rehashing of the steps Chris Torek provides, but specific to T520 laptops (and it should work on T420s and W520 too). PC-BSD users with Lenovo laptops may also find this guide useful, as you can apply this hack to their installation media to get them booting in legacy mode (UEFI doesn't seem to work currently). 

As this tutorial uses the BSD version of `fdisk`, we will boot into a BSD environment. The problem is that the GPT bug stops us booting from USB pens with legacy boot enabled in BIOS, so we need to boot into a UEFI image. `FreeBSD-10.1-RELEASE-amd64-uefi-mini-memstick.img` worked for me. [Prepare the boot media](https://www.freebsd.org/doc/handbook/install-pre.html#install-boot-media), as instructed in the handbook. To boot into it, press `F1` key at computer boot and choose `Startup` -> `UEFI/Legacy Boot` -> `Both`. If booting from a USB image, you also need to make sure `Config` -> `USB` -> `USB UEFI BIOS Support` is Enabled. Also, in the Security tab you must make sure Secure Boot is not enabled (I don't even see a clearly labelled option for it in my T520 bios, so I assume it is off by default).

Now boot from the UEFI USB (`F12` at startup). You can install a fresh copy of FreeBSD first, or drop to the shell straight away to fix either an existing installation, or PC-BSD bootable media. Press `Ctrl-F2`, and log in as root.

When typing the below commands, you will need to replace `adaX` with e.g., `ada0` if it is the primary disk, `ada1` if it is the secondary, or maybe `da0` if you are fixing a USB pen image. Run:

```
fdisk -p /dev/adaX > /tmp/parts
```

Open this file with an editor e.g., `ee /tmp/parts`:

```
    # /dev/ada0
    g c969021 h16 s63
    p 1 0xee 1 976773167
```

Your file likely won't look exactly like this (it may have some extra lines), but it will look similar. Change `s63` (sectors-per-track) to `s1`. Copy/paste the last line to make a 4th line. Change the `0xee` on the 3rd line to `0x00` and change `p 1` on the fourth line to 	`p 2`. 

Chris Torek also changes the `cyclinder count`, and I recommend to read his [post](http://lists.freebsd.org/pipermail/freebsd-i386/2013-March/010437.html) for the reasoning, but I found this step unecessary. If you wish to be on the safe side and change it anyway, then in the above example you would multiply `c969021` (cylinder count) by the sectors-per-track `s63`, so the cyclinder count would become `c61048323`.

The final product should look something like this:

```
    # /dev/ada0
    g c61048323 h16 s1
    p 1 0x00 1 976773167
    p 2 0xee 1 976773167
```

Now run:

```
fdisk -f /tmp/parts /dev/adX
```

You've now modified the partition table so that partition one is an empty dummy partition, tricking the Lenovo bios, and preventing it from borking at seeing an `ee` type protective MBR partition as the first partition.