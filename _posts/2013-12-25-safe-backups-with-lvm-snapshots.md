---
layout: post
published: true
---

This guide assumes the reader knows how to:

- use a terminal
- partition a disk

Creating an LVM snapshot is useful as it makes a "freeze" of the filesystem so that things are in a consistent state before making a backup. LVM requires the creation of a snapshot volume to store them, but it needn't go bigger than a few gig, as the snapshots only store the changed files in the overlying filesystem. 

## Preparing a drive to store the snapshots

Setup as taken from `blkid` command:

SSD 128G (/dev/sda)
Hard Drive 320G (/dev/sdb)

I'm not keen on the idea of resizing existing volumes on the SSD to make room for a snapshot volume, however small. That said, I instead split a second drive into two partitions:

LVM Partition 1 (To share with SSD):

`gdisk /dev/sdb` then `n`, `Enter`, `Enter`, `+129G`, `8e00`, `w`

LVM Partition 2 (To store imaged backups)

`n`, `Enter`, `Enter`, `Enter`, `8e00`, `w`

...  and share the first partition with the SSD in LVM like this:

```
# Use 'VG Name' field of `lvdisplay` command to get the existing Volume Group name):
pvcreate /dev/sdb1
vgextend VolGroup00 /dev/sdb1 # VolGroup00 already exists on /dev/sda1 SSD
```
So, use the above as a guideline. Notice that the first partition of the second disk is the same size as the entire SSD. It needn't be that big as it only holds the changed files in the volumes, but it doesn't hurt to stay on the safe side if the space is available. 

The sata hard drive space is now tacked onto the end of the SSD space. Don't worry though, no data is spread to it while it isn't mounted. 

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