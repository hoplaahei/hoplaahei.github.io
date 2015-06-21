---
layout: post
published: true
title: How I install Slackware
---












# Preparation

Disclaimer: these commands will wipe your disk. The commands use the form `sdX`, where you need to replace the `sdX` with e.g., `sda`, and where 'a' is usually the first disk (but double check with `fdisk` or `gdisk` to make sure). Google if you don't understand how to use these tools. 

## Get Slackware ISO onto a USB pen

See the [torrents](http://www.slackware.com/getslack/torrents.php) page. I chose [Slackware 14.1 64-bit](http://www.slackware.com/torrents/slackware64-14.1-install-dvd.torrent) from the bottom of the page. At the time of writing it is the stable edition.

Even though the ISO is for DVDs, it works just as well on a USB stick (replace sdX with the USB stick device):

```
dd if=slackware-DVD-iso-you-downloaded.iso of=/dev/sdX bs=1M && sync
```

## SSD memory cell clearing

This step is not at all essential to installing Slackware, but it is something I always do out of habit. The tl;dr of SSD memory cell clearing is that it could reset any slow down in SSD write speeds by returning it to factory defaults (this blanks the disk):

```
echo -n "mem" > /sys/power/state # suspend to RAM to unfreeze security
hdparm --user-master u --security-set-pass PasSWorD /dev/sdX
hdparm --user-master u --security-erase PasSWorD /dev/sdX
```

The `ArchWiki` has a good [article](https://wiki.archlinux.org/index.php/SSD_memory_cell_clearing) on this, so I'm not going to reiterate the steps in detail.

## Installing

The SlackDocs wiki has an [install](http://docs.slackware.com/slackware:install) guide for standard installation. It is a good idea to read this guide. Some modern computers may also benefit from booting into Slackware using their native UEFI boot. See [UEFI boot: how does that actually work, then?](https://www.happyassassin.net/2014/01/25/uefi-boot-how-does-that-actually-work-then/) to get a clear understanding of the cases where native UEFI boot is beneficial, and the cases where it is not necessary. 

Read through the UEFI [README](http://slackware.mirrorcatalogs.com/slackware64-14.1/README_UEFI.TXT) for how to setup an EFI partition and make Slackware UEFI bootable. My guide also covers these steps. 

For LVM partitioning, there are some additional steps needed before and after setup, so read `README_LVM.txt` on the USB before you boot into it. I'm going to cover the extra steps that are needed when using a UEFI and LVM setup.

## An example LVM setup with UEFI

Choose the disk you want to install Slackware on by using `blkid` to see available devices. If you skipped SSD memory cell-clearing then you may still want to get rid of remnants of old partitions on the chosen disk by running `gdisk /dev/sdX` and pressing `x` for `extra functionality`, followed by `z` to `zap (destroy) GPT data structures and exit`.

Now make a 512M EFI (ee00 code) boot partition using `gdisk /dev/sda` again:

Press `n` and choose `1` to select 1st partition. Leave start sector at default by pressing `Enter`, but change end sector to `+512M` (it is important to keep it at this size on some firmwares, but 100M is reported to work for a lot of people if you need something smaller). For the type enter the code `ef00` (EFI).

Press `n` and `Enter` to make partition 2. `Enter` again for default start block, and `Enter` once more for default end block. `8e00` to create type LVM and then `w` to write the changes. 

Now create the LVM hierarchy:

```bash
pvcreate /dev/sdX2 # activate
vgcreate VolGroup00 /dev/sdX2
lvcreate -C y -L 9G VolGroup00 -n lvolswap # -C y option is for contigious allocation
lvcreate -l +100%FREE VolGroup00 -n lvolroot # use remaining space
```

Format the swap and EFI partitions, so that the Slackware installer will detect them:

```
mkswap /dev/VolGroup00/lvolswap
mkfs.vfat -F32 /dev/sdX1 # the EFI partition
```

Now run `setup` as usual and Slackware should automatically detect the partitions as the installer goes along. Select USB as the installation media if you followed the steps in this guide to create the media. Do not install `lilo` when prompted; the installer will prompt you to install `elilo` instead. Also see the Slackware [install](http://docs.slackware.com/slackware:install) guide for further help.

**DO NOT REBOOT**

There are some additional steps needed with an LVM setup or you will receive a kernel panic. We will switch from the HUGE kernel to the GENERIC kernel to fix this, and also copy the `initrd` and `vmlinuz` from that kernel to the EFI partition, so they are seen in a UEFI boot.

First we must `chroot` into our Slackware install. The installer has already setup the environment for us, so simply enter:

```
chroot /mnt
```

Now use this script to analyse our setup and output a command we can use:

```
/usr/share/mkinitrd/mkinitrd_command_generator.sh -r
```

That will output a line with the command to generate the `initrd`, but we also want to add resume support for our swap partition by including `-h /dev/yourVG/yourSwapLV`, e.g.,:

```
mkinitrd -c -k 3.10.17 -f xfs -r /dev/slack/root -m usb-storage:ehci-hcd:ehci-pci:xfs -h /dev/slack/swap -L -u -o /boot/initrd.gz
```

Copy over the new `initrd` and kernel to the `EFI` partition:

e.g., 

```
cp /boot/vmlinuz-generic-3.10.17 /boot/efi/EFI/Slackware/
cp /boot/initrd.gz /boot/efi/EFI/Slackware/
```

And edit `/boot/efi/EFI/Slackware/elilo.conf`. The syntax is mostly the same as in the beginners guide example for `lilo.conf`, but the `/boot/` part of the paths need removing, e.g.,:

```
image = vmlinuz-generic-3.10.17
        initrd = initrd.gz
        label = 3.10.17
        read-only
        append="root=/dev/VolGroup00/lvolroot vga=normal ro"
```

Leave any old entries in there above the new one; that way, you can select which kernel to boot with the arrow keys from the menu on boot. Reboot into the new system now. If it boots correctly, add this line after `timeout=1` in the `/etc/elilo.conf`:

```
default=3.10.17
```

And replace the above example with the `label` of the boot entry you need to boot. Skip to the next section. 

If it doesn't boot correctly, you've likely made a typo, or missed a step. In that case, boot back into the Slackware install USB and make the installer aware of the existing LVM volumes using:

```
vgchange -ay
```

Now follow the Slackware [chroot](http://docs.slackware.com/howtos:slackware_admin:how_to_chroot_from_media) guide. That guide assumes a non-LVM setup, so replace /dev/sdX2 accordingly. For example, you will first need to mount the LVM root volume `/dev/VolGroup00/lvolroot` on `/mnt` and the EFI partition `/dev/sdX1` on `/mnt/boot`, then bind mount `dev`, `proc` and `sys`. Redo all the steps from the **DO NOT REBOOT** warning above. Remember you need to:

- examine `/boot/efi/EFI/Slackware/elilo.conf` to make sure it is correct
- regenerate the inital RAM-disk
- copy over the ramdisk and kernel image to `/boot/efi/EFI/Slackware` so `elilo` is aware of them

## Get correct keys on the keyboard

After logging in you may want to work from the console. Load your keyboard language, e.g., United Kingdom with:

```
loadkeys uk
```

It is useful, however, to start `X` with `startx` command to run the default window-manager selected in installation e.g., `windowmaker`, but the keys might not be right on the keyboard in `X` either. For a quick fix, run from an `xterm` e.g.,

```
setxkbmap gb # for a British keyboard
```

This will only work for the current session, but we will set it permanently later.

## Wireless networking

There are a couple of different ways to setup wifi outlined in the Slackware [wifi](http://docs.slackware.com/slackbook:wifi) docs. I use NetworkManager because it is the quickest to get up and running on a fresh install. IMO most of the hate for NetworkManager is undeserved these days; it has worked pretty good for a while now. 

Enable `NetworkManager` at boot:

```
chmod +x /etc/rc.d/rc.networkmanager
```

Start NetworkManager:

```
/etc/rc.d/rc.networkmanager start
```

From the terminal list the available access points:

```
nmcli dev wifi
```

Connect to one (start this line with a space to prevent the wifi password getting stored in the history file of the shell)

```
 nmcli dev wifi connect YourAccessPoint password YoUrPaSs
```

## Fix incorrect time

 [This](http://docs.slackware.com/howtos:hardware:syncing_hardware_clock_and_system_local_time) SlackDoc explains how to keep the time of all dual-booted operating systems in sync.

## Follow the beginners guide

Do all the steps in the [beginners guide](http://docs.slackware.com/slackware:beginners_guide) and also follow the link to [localisation](http://docs.slackware.com/slackware:localization), as you can do useful things like get the '£' sign working permanently on a UK keyboard in `X`. 

I advise to read the comments of `/etc/profile.d/lang.sh` when setting a locale, and follow the recommendation not to enable UTF-8 unless absolutely necessary. I experienced strange behaviour with some applications and UTF-8, e.g., `xpdf` failing to load some PDFs. If, however, UTF-8 is essential to your system, then you can still get the problem apps to load by overriding the locale per-application with `C` locale like this:

```
LANG=C xpdf
```

## Learn to use Slackbuilds

[Install](http://www.sbopkg.org/downloads.php) `sbopkg` and sync with the remote repository by running:

```
sbopkg -r
```

Now follow this guide to [manage queue files easily](http://slackblogs.blogspot.co.uk/2014/01/managing-sbo-dependencies-easily.html).

## Convert to multilib (optional)

`wine`, `virtualbox`, `skype` and `google-earth` are some examples of programs that need a `multilib` system.

`slackpkg+` can enable `multilib` and keep it updated easily. Download the latest [slackpkg+](http://sourceforge.net/projects/slackpkgplus/files/) package. Install from the cli in the same directory as the downloaded file with:

```
installpkg slackpkg+-*.txz
```

At the time of writing, the README says it is possible to use the script `/usr/doc/slackpkg+-*/setupmultilib.sh`. This script didn't work properly for me, as it didn't uncomment the line to set the multilib repo as the priority, which had me scratching my head.

Instead, manually edit `/etc/slackpkg/slackpkgplus.conf`:

```
# Uncomment (and keep it first in the list)
PKGS_PRIORITY=( multilib )
...

# Add "multilib" (order doesn't matter here)
REPOPLUS=( multilib slackpkgplus restricted alienbob slacky )
...
# And also uncomment
MIRRORPLUS['multilib']=http://taper.alienbase.nl/mirrors/people/alien/multilib/14.1/
```

Now run:

```
slackpkg update gpg
slackpkg update
slackpkg upgrade gcc glibc
slackpkg upgrade-all # optional
slackpkg install multilib
```

The [official](http://alien.slackbook.org/dokuwiki/doku.php?id=slackware:multilib) method works fine too, and is good for those who prefer not to use a third-party package management tool such as `slackpkg+`. Just follow the **'Quick n' dirty'** instructions section of the linked wiki page. It is not necessary to also complete the 'Detailed' instructions further down the page, but it might help in understanding what is going on. 

[phenixia2003](https://www.linuxquestions.org/questions/slackware-14/slackware64-and-my-stupidity-4175484839/#post5066064) gives a good overview of all the available methods for installing multilib.

## Upgrade packages

The [systemupgrade](http://docs.slackware.com/howtos:slackware_admin:systemupgrade) wiki article covers all the steps. 

**Note:** if using `UEFI` and `elilo`, it is safe to upgrade kernel automatically without first blacklisting it, because the old kernel will still boot until you explicitly activate the new one by copying it over to the `EFI` partition. See **"Using a generic kernel on UEFI systems"** heading above for how to do that.

One thing I found annoying is that `slackpkg install-new` asks to upgrade from package sets I'd already deselected in setup (e.g., kde, xfce). To stop getting prompts to install such packages, edit `/etc/slackpkg/blacklist` with e.g.,:

```
kde/*
kdei/*
xfce/*
```

## Support for nVidia Optimus

Someone made a one-liner to install it that works for me. See the [docs](http://docs.slackware.com/howtos:hardware:nvidia_optimus).

## Backing up

When [rsnapshot](http://rsnapshot.org/) is configured right, making a complete backup of the OS is as simple as running `rsnapshot manual` (I prefer this to an automatic cron job). The initial backup is a little slow, and takes up as much disk space as you've used on the root filesystem, but only the changes get backed up on subsequent backups. `rsnapshot` is great for reverting a file system "back-in-time". I keep up to 30 historical backups available at-a-time to switch back to. Every month I choose one of the backups to `rsync` to the cloud. 

`rsnapshot` config file has support for automatically taking a temporary LVM snapshot of a volume before backing it up. The snapshot creates a consistent, atomic (point-in-time) image of the system before running rsync. This allows me to backup the running root filesystem of the OS live, without having to unmount it and remount it read only first, and also gives the assurance that write processes are allowed to finish before the backup is taken. Other filesystems such as `BTRFS` and `ZFS` also support snapshotting, but I prefer the stability of `LVM` on Linux, with its added bonus of nice integration with `rsnapshot`. 

The relevant parts of the config I use are below. The config is easily modified to work without LVM by commenting out the LVM stuff, and changing the first parameter of `backup` to a standard partition e.g., `/dev/sda2`. However, without the LVM snapshotting capability, I'd recommend first unmounting `/` and booting into another OS or LiveCD to do the backup. Otherwise, running processes might only partially complete writing to files, or the contents of a file might change as it is transferred over, leading to partial and inconsistent backups.

Here are the important parts of my `/etc/rsnapshot.conf`:

```
config_version  1.2

snapshot_root   /media/backup/rsnapshot/
retain  manual  30
no_create_root  1
cmd_rm          /usr/bin/rm
cmd_rsync       /usr/bin/rsync
cmd_logger      /usr/bin/logger
cmd_du          /usr/bin/du

interval        hourly  6
interval        daily   7
interval        weekly  4
#interval       monthly 3

verbose         4

loglevel        3

lockfile        /var/run/rsnapshot.pid

one_fs          1

exclude dev/
exclude media/
exclude mnt/
exclude proc/
exclude sys/
exclude tmp/
exclude run/
exclude home/Downloads
exclude home/.gvfs

linux_lvm_cmd_lvcreate  /sbin/lvcreate
linux_lvm_cmd_lvremove  /sbin/lvremove
linux_lvm_snapshotsize  5G
linux_lvm_snapshotname  rsnapshot-tmp
linux_lvm_vgpath        /dev
linux_lvm_mountpath     /mnt/rsnapshot-tmp
linux_lvm_cmd_mount     /usr/local/bin/mount-wrapper
linux_lvm_cmd_umount    /bin/umount

backup  lvm://VolGroup00/lvolroot/       slack-root/
```

Some explanation of the options:

- `snapshot_root` :: This is the directory where the backup will happen. I mount an external drive here to keep the backup offsite and save space. 
- `no_create_root  1` :: Prevent rsnapshot automatically creating the backup directory if it doesn't exist. Useful to make sure the external drive containing the backup directory is mounted (otherwise the backup would just create the directory automatically on the source drive and backup to the wrong place.)
- `one_fs          1` :: Don't go across to other mounted filesystems (so prevent also backing up e.g., mounted USB pens and external disks)
- `retain  manual  30` :: Keep 30 manual backups before starting to replace them
- `verbose         4` :: Print out which files are currently copying, so I know the backup is still running ok
- `linux_lvm_cmd_mount` ::  In later versions of `rsnapshot` it is possible to pass mount options here, but not in the release version on Slackware 14.1, so I point this option to a mount-wrapper script. 

Create the directories `/mnt/rsnapshot-tmp` and `/media/backup/rsnapshot` for the following script to work. 

Contents of the wrapper script `/usr/local/bin/mount-wrapper`:

```
#! /bin/bash
#
# This is a wrapper for mount to pass it the nouuid option.
# This is required to enable rsnapshot to mount an xfs lvm snapshot.

/bin/mount -o nouuid $1 $2
if [ "$?" -ne 0 ]; then echo "Error detected. This wrapper only works with XFS filesystems"; exit 1; fi
```
I need this workaround because `XFS` will not mount filesystems with the same `UUID` by default.
