---
layout: post
published: false
title: How I install Slackware
---

# Preparation

Disclaimer: these commands will wipe your disk. The commands use the form `sdX`, where you need to replace the `sdX` with e.g., `sda`, and where 'a' is usually the first disk (but double check with `fdisk` or `gdisk` to make sure). Also, Google if you don't understand how to use these tools. 

## SSD memory cell clearing

The tl;dr of SSD memory cell clearing is that it could potentially give you back the same write speeds your SSD had from the factory by running these commands:

```
echo -n "mem" > /sys/power/state # suspend to RAM to unfreeze security
hdparm --user-master u --security-set-pass PasSWorD /dev/sdX
hdparm --user-master u --security-erase PasSWorD /dev/sdX
```

The `ArchWiki` has a good [article](https://wiki.archlinux.org/index.php/SSD_memory_cell_clearing) on this, so I'm not going to reiterate the steps.

## Partitioning

Read [this](http://slackware.mirrorcatalogs.com/slackware64-14.1/README_UEFI.TXT) guide. It will help you decide whether you want or need modern partitioning scheme and/or UEFI boot.

Now run:

```
mkswap /dev/sdX #whatever partition you used for swap)
setup
```



