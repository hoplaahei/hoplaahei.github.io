---
layout: post
published: false
title: How I install Slackware
---



# Preparation

Disclaimer: these commands will wipe your disk. The commands use the form `sdX`, where you need to replace the `sdX` with e.g., `sda`, and where 'a' is usually the first disk (but double check with `fdisk` or `gdisk` to make sure). Also, Google if you don't understand how to use these tools. 

## Get Slackware ISO onto a USB pen

See the [torrents](http://www.slackware.com/getslack/torrents.php) page. I chose [Slackware 14.1 64-bit](http://www.slackware.com/torrents/slackware64-14.1-install-dvd.torrent) from the bottom of the page. At the time of writing it is the stable edition.

Once downloaded put it on a USB stick from the terminal:

```
dd if=slackware-iso-you-downloaded.iso of=/dev/sdX bs=1M && sync
```

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

Follow that guide to make the partitions and then run:

```
mkswap /dev/sdX #whatever partition you used for swap)
setup
```

When it asks which media to use, choose USB. It will scan and detect the installation files automatically. The installer will automatically detect the EFI and swap partitions available. It is quite intelligent in figuring out which partitions to use, but you need to tell it specifically. Follow the Slackware documentation for further advice on installs, but rest-assured that most of it is fairly self-explanatory.

## When the installation is done

Now is a good time to make a nice clearn image of your installation to revert back to if you mess up. This will save the hassle of having to go through the whole partitioning procedure next time. You can also use it to put Slackware on your other computers if they have similar sized disks. I don't recommend adding anything else to this clean-slate image, except perhaps some basic steps from the [Slackware Beginners Guide](docs.slackware.com/slackware:beginners_guide), as, if you are forgetful like me, you will forget what you changed by the time you come to need the image, which could cause confusion.

For a first time backup a simple `dd` to a compressed file should suffice:

```
```

## Learn to use Slackbuilds

[Install](http://www.sbopkg.org/downloads.php) `sbopkg` and sync with the remote repository by running:

```
sbopkg -r
```

Now follow this guide to [manage queue files easily](http://slackblogs.blogspot.co.uk/2014/01/managing-sbo-dependencies-easily.html).