---
layout: post
published: true
---

Creating an LVM snapshot makes a "freeze" of the filesystem so that things are in a consistent state before making the backup. This guide assumes the reader knows how to:

- use a terminal
- partition a disk

## Preparing a drive to store the snapshots

** If you are lucky enough to have a big disk that isn't already full with LVM volumes, then skip this first section. **

In my setup I have an SSD disk (my main disk) that has no more room for LVM snapshots because I assigned the full extents to my `/` and `/home` partition when I installed my OS. I could mess about using lgreduce, extresize, etc to shrink my SSD logical volumes and make room for a snapshot volume, but that is too much hassle and not very flexible for any future changes. I'm going to extend my logical volumes to a second disk to give me more room. It doesn't really matter if the disk is slower than the first disk (as my other magnetic sata disk is), since we only use it temporarily for the snapshotting. 

So, make sure the extra device is connected and use the likes of `blkid` (or `dmesg | tail` if it's a USB) to find the device number for it such as /dev/sdb. Decide how big the partition needs to be; it will hold snapshots containing any changes to the filesystem. Something smaller such as 5 gig should suffice for most tasks, but I stay on the safe side and match it with the full size of the disk I'm backing up. In my case, I'm backing up all the logical volumes on an SSD to a hard drive. `pvdisplay` shows me that all the logical volumes on my SSD drive total 128.74G, which I round off to 129G. With that in mind I use this to partition:

`gdisk /dev/sdb` then `n`, `Enter`, `Enter`, `+129G`, `8e00`

Then I repeat the step this time using all remaining free space (the default) for the size, so that I have two partitions on the disk. The first one will contain a volume group that is shared with my first disk and only used temporarily to make backups. The second partition is used to physically store the backups of the first drive. 

Make the first partition LVM ready and add it to the existing volume group (which you can find the name of by looking at the 'VG Name' field of `lvdisplay` command):

```
pvcreate /dev/sdb1
vgextend VolGroup00 /dev/sdb1
```
The sata hard drive space is now tacked onto the end of the SSD space. No data is written to it though, because it isn't mounted. 

Find out the current extent number of the SSD LVM volume to be backed up:

```
lvdisplay -v /dev/VolGroup00/lvolhome
```
Create a snapshot volume using the extent number listed in the output. The `/dev/sda1` on the end isn't necessary unless you are trying to force it to write to a particular disk as I am (i.e., a sata hard disk):

```
lvcreate -l 22717 -s /dev/VolGroup00/lvolhome -n lvolhomesnap /dev/sda1
```
Copy the snapshot over to the backup (hard disk) device:

```
dd if=/dev/mapper/VolGroup00-lvolhomesnap of=/dev/mapper/VolGroup01-lvolbackup
```