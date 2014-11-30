---
layout: post
title: "Fix GPT boot on FreeBSD for Thinkpad T520, T420s & W520"
published: true
tags: ""
---

There is a long withstanding bug in the bioses for T520, T420s and W520 that prevents the booting of operating systems that use a GPT partition scheme with type `ee00` (protective MBR) as the first partition. So if you're one of the [many](http://forums.lenovo.com/t5/Linux-Discussion/Lenovo-Thinkpad-T520-doesn-t-boot-with-GPT-slices-on-FreeBSD-9/td-p/555317) Thinkpad owners tearing their hair out after following FreeBSD EFI/GPT installation guides to the letter, and still having no success, this post could finally offer a stop-gap solution. 

If you've struggled to get the install images working with legacy boot, get one of the UEFI images. Search for e.g.,: `FreeBSD-10.1-RELEASE-amd64-uefi-mini-memstick.img`. In `F1` key setup at computer boot choose `Startup` -> `UEFI/Legacy Boot` -> `Both`. If booting from a USB image, you also need to make sure `Config` -> `USB` -> `USB UEFI BIOS Support` is Enabled. Also, in the Security tab you must make sure Secure Boot is not enabled (I don't even see a clearly labelled option for it in my T520 bios, so I assume it is off by default).

Now boot from the UEFI USB (`F12` at startup). Run the installation as normal, being sure to find and select GPT partitioning in the installer options. Now before rebooting choose to drop to a shell or press `Ctrl-F2` and login as root.

When typing the below commands, you will need to replace `adaX` with e.g., `ada0` if it is the primary disk or, `ada1` if it is the secondary. Run:

```
fdisk -p /dev/adaX > /tmp/parts
```

Open this file with an editor e.g., `ee /tmp/parts`:

```
    # /dev/ada0
    g c969021 h16 s63
    p 1 0xee 1 976773167
```

Your file likely won't look exactly like this, but it will look similar. Change `s63` to `s1`. Copy/paste the last line to make a 4th line. Change the `0xee` on the 3rd line to `0x00` and change `p 1` on the fourth line to 	`p 2`. The final product should look something like this:

```
    # /dev/ada0
    g c969021 h16 s1
    p 1 0x00 1 976773167
    p 2 0xee 1 976773167
```

Now run:

```
fdisk -f /tmp/parts /dev/adX
```

You've now modified the partition table so that partition one is an empty dummy partition, tricking the crappy Lenovo bios, and preventing it from borking at seeing an `ee` type protective MBR partition as the first partition.