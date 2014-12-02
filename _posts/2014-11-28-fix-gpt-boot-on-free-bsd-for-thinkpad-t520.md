---
layout: post
title: "Fix GPT boot on FreeBSD or PC-BSD for Thinkpad T520, T420s & W520"
published: true
tags: ""
---

**If you are already aware of the issues surrounding GPT with Lenovo bioses, scroll straight down to 'Booting a live image'. If you already have a working BSD installation, skip to 'The hack'.**

To cut a long story short, when booted in legacy mode, the bioses in Lenovo T520, T420s and W520 cannot handle the way BSD installations try to partition with GPT. The bios borks when it sees an `ee` type protective MBR partition as the first partition. This means that if you want to use ZFS-on-root (which requires booting in legacy mode), you will not be able to boot into a system partitioned with GPT. To solve this problem, we could partition with MBR instead. If, however, we insist on a GPT installation (for future-proofing purposes), then a low-level hack must be performed, so I suggest you backup any data first.

We will use the BSD version of `fdisk` to edit the partition table of an existing GPT installation or GPT installation media to make it bootable. Skip the next paragraph if you already have a working BSD environment to run fdisk on.

## Booting a live image

If you don't have a working BSD, booting into a live environment in UEFI works as a temporary solution. Download `FreeBSD-10.1-RELEASE-amd64-uefi-mini-memstick.img`, or whatever is the latest. [Prepare the boot media](https://www.freebsd.org/doc/handbook/install-pre.html#install-boot-media), as instructed in the handbook. To boot into it, press `F1` key at computer boot and choose `Startup` -> `UEFI/Legacy Boot` -> `Both`. If booting from a USB image, you also need to make sure `Config` -> `USB` -> `USB UEFI BIOS Support` is Enabled. Also, in the Security tab you must make sure SecureBoot is not enabled (I don't even see a clearly labelled option for it in my T520 bios, so I assume it is off by default). Now boot from the UEFI USB (`F12` at startup). 

## The hack

Once at a console, type:

```
fdisk -p /dev/adaX > /tmp/parts
```

Replace `adaX` with e.g., `ada0` if FreeBSD/PC-BSD is installed on the primary disk, `ada1` if it is the secondary, or maybe `da0` if you are fixing a USB pen installation media. 

Now open the file you just made with and editor e.g., `ee /tmp/parts`:

```
    # /dev/ada0
    g c969021 h16 s63
    p 1 0xee 1 976773167
```
Your file probably won't look exactly like this, but it will look similar. You need to edit it to this:

```
    # /dev/ada0
    g c61048323 h16 **s1**
    p 1 0x00 1 976773167
    p 2 0xee 1 976773167
```
So basically just copy/paste the 3rd line to make a 4th line, and then save the file after making the changes marked in bold.

**See Chris Torek's original [post](ttp://lists.freebsd.org/pipermail/freebsd-i386/2013-March/010437.html) for an in-depth explanation of these changes.**

To apply the changes to the media:

```
fdisk -f /tmp/parts /dev/adX
```

Partition '1' is now an empty dummy partition, tricking the Lenovo bios, and preventing it from borking at seeing an `ee` type protective MBR partition as the first partition. 

For FreeBSD users, you're done, and can now boot into the media like normal. PC-BSD users who are trying to get an installation media working should now go back into BIOS setup and set if to prefer legacy boot over UEFI boot, because the UEFI boot on the installation medias are broken (at the time of writing). 