---
layout: post
published: false
title: How I install Slackware
---

# Preparation

Disclaimer: these commands will wipe your disk. The commands use the form `sdX`, where you need to replace the `sdX` with e.g., `sda`, and where 'a' is usually the first disk (but double check with `fdisk` or `gdisk` to make sure). Google if you don't understand how to use these tools. 

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

The SlackDocs wiki has an [install](http://docs.slackware.com/slackware:install) guide for standard installation. Some modern computers force the use of UEFI boot with a GPT partitioning scheme, and if you require/want this then see the UEFI [README](http://slackware.mirrorcatalogs.com/slackware64-14.1/README_UEFI.TXT).

Afer partitioning, simply run:

```
setup
```

If following the steps in my guide then you are using a USB installation, so when prompted for the installation media choose USB and the installer will detect the install packages correctly. EFI and swap partitions should get detected and formatted automatically. Follow the `SlackDocs` on [installation](http://docs.slackware.com/slackware:install) for further advice, but the installer is clever in figuring out what you want.

If you followed the partitioning scheme in this guide, then you may skip to the next section. If, however, you used `LVM` (Logical Volume Manager), then there are some additional steps to do before rebooting:

```
mount -o bind /dev /mnt/dev
mount -o bind /proc /mnt/proc
mount -o bind /sys /mnt/sys
chroot /mnt
$( /usr/share/mkinitrd/mkinitrd_command_generator.sh -r )
exit
```

Now modify `/boot/efi/EFI/Slackware/elilo.conf` for `UEFI` systems or `/etc/lilo.conf` for `BIOS` systems. Change the append line to:

```
append="root=/dev/yourVG/yourLV vga=normal ro"
```

Replace 'yourVG' with whatever you named the `Volume Group` and 'yourLV' with the `Logical Volume` you created.  

Now run:

```
eliloconfig # or liloconfig if you use lilo
```

And reboot.

## When the installation is done

Now is a good time to make a nice clean image of your installation to revert back to if you mess up. This will save the hassle of having to go through the whole partitioning and setup procedure next time. This image is useful for deploying Slackware to other computers with similar sized disks (but remember to change `[e]ilo` and `fstab` entries). I don't recommend adding anything else to this clean-slate image, except perhaps the basic configuration from the [Slackware Beginners Guide](docs.slackware.com/slackware:beginners_guide). Otherwise, if you are forgetful like me, you won't remember what was edited by the time you actually come to need the image, which could cause confusion.

There is no one absolute, universally agreed on method for backing up. If there is disk-space to spare, I recommend a simple `dd` to a compressed image file for the first time backup, as it is a tried and true method. Do not leave the system that needs backing up running unless the underlying filesystem supports freezing (e.g., XFS). Instead, reboot into a live environment such as the Slackware Install ISO.

I'm using XFS filesystem, which allows freezing the mounted disk, so in that case rebooting into a live environment isn't necessary. But to avoid any problems with PID files of running processes and such in the frozen image, I also switch to single-user mode to shut off as much as I can, by running:

```
/sbin/init 1
```

That way a minimal number of things are running at the time of the freeze.

Also, make sure the backup goes to a spare disk with at least nearly as much room as the disk that needs backing up, or disk space might run out. 

```
xfs_freeze -f /
dd if=/dev/sda | gzip > /path/to/backup/disc/location/ssd_backup.gz
```

## Learn to use Slackbuilds

[Install](http://www.sbopkg.org/downloads.php) `sbopkg` and sync with the remote repository by running:

```
sbopkg -r
```

Now follow this guide to [manage queue files easily](http://slackblogs.blogspot.co.uk/2014/01/managing-sbo-dependencies-easily.html).

## Bastardize your installation with multilib

Adding 32-bit support to any Linux system currently draws in an extra layer of complexity that we could all do without, but it's a necessary evil to get some applications running. For instance, many wine apps needs 32-bit support, and I prefer the near native speed for games it gives me (no my laptop does not support [VGA passthrough](https://wiki.debian.org/VGAPassthrough) with KVM).

Fortunately, `slackpkg+` makes converting Slackware to multilib easy. 
