---
layout: post
title: "Fix GPT boot on FreeBSD or PC-BSD for Thinkpad T520, T420s & W520"
published: true
tags: ""
---

To cut a long story short, the bios in Lenovo T520, T420s and W520 cannot handle the way BSD installations partition with GPT. Lenovo bios borks when it sees an `ee` type protective MBR partition as the first partition. To solve this problem we must perform a low-level hack, so I suggest you backup any data first.

Plug in the installation media you want to fix and drop to a an existing BSD console (as we will use the BSD version of fdisk). If you don't have an existing BSD installation to access, then try the latest FreeBSD UEFI install media images. They boot fine as long as you enable UEFI boot in the bios. I personally set the bios to boot 'both' UEFI and legacy (apparently T420s only boot FreeBSD UEFI this way). 

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
Your file probably won't look like this, but it will look similar. You need to edit it to this:

```
    # /dev/ada0
    g c61048323 h16 *s1*
    p 1 0x00 1 976773167
    p 2 0xee 1 976773167
```

You've just edited a record of your partition table. Now to actually apply this to the partition table directly, run:

```
fdisk -f /tmp/parts /dev/adX
```

Partition '1' is now an empty dummy partition, tricking the Lenovo bios, and preventing it from borking at seeing an `ee` type protective MBR partition as the first partition.