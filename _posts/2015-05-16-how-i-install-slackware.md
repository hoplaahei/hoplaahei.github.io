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

Follow that guide to make the partitions and then simply run:

```
setup
```

When it asks which media to use, choose USB. It will scan and detect the installation files automatically. The installer will automatically detect the EFI and swap partitions available. It is quite intelligent in figuring out which partitions to use, but you need to tell it specifically. Follow the `SlackDocs` on [installation](http://docs.slackware.com/slackware:install) for further advice, but rest-assured that most of it is fairly self-explanatory.

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

Now is a good time to make a nice clean image of your installation to revert back to if you mess up. This will save the hassle of having to go through the whole partitioning and setup procedure next time. You can also use this image to put Slackware on other computers if they have similar sized disks (but remember to change `[e]ilo` and `fstab` entries). I don't recommend adding anything else to this clean-slate image, except perhaps the basic configuration from the [Slackware Beginners Guide](docs.slackware.com/slackware:beginners_guide). Otherwise, if you are forgetful like me, you won't remember what was edited by the time you actually come to need the image, which could cause confusion.

There is no one absolute, universally agreed on method for backing up. If you've got a lot of spare disk space, I recommend a simple `dd` to a compressed image file for the first time backup, as it is a tried and true method. Please note that if you didn't choose a filesystem which supports freezing the disk such as `XFS`, then rebooting into a live environment such as the Slackware Install ISO is necessary, or anywhere where the disk to backup isn't mounted. 

I'm using XFS, which allows to freeze the mounted disk, and in that case rebooting into a live environment isn't necessary. To avoid any problems with PID files of running processes and such in the frozen image, I also switch to single user to shut off as much as I can, by running:

```
/sbin/init 1
```

That way a minimal number of things are running at the time of the freeze.

Also, make sure the backup goes to a spare disk with at least nearly as much room as the disk you are backing up to, or disk space might run out. 

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
