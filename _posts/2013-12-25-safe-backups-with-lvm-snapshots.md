---
layout: post
published: true
---

Creating an LVM snapshot makes a "freeze" of the filesystem so that things are in a consistent state before making the backup. This guide assumes the reader knows how to:

- use a terminal
- partition a disk

## Preparing a drive to store the snapshots

If you have free extents on your LVM disk then skip this step (LVM will warn when you do not have enough free extents).

If, however, there is no free space left on the physical device for other logical volumes, then use a new device.

e.g., insert a USB pen and use `dmesg | tail` to find the device number such as /dev/sdb. Now in gdisk:

Type `gdisk /dev/sdb` then `n`, `Enter`, `Enter`, `+5G`, `8e00`

Make it LVM ready and add it to the existing volume group (which you can find the name of by looking at the 'VG Name' field of `lvdisplay` command):

```
pvcreate /dev/sdb1
vgextend VolGroup00 /dev/sdb1
```
The new disk is now tacked onto the end of the LVM drive. No data is written to it though, because it isn't mounted. 

Find out the current extent number of the LVM volume to be backed up:

```
lvdisplay -v /dev/VolGroup00/lvolhome`
```

Create a snapshot volume using the extent number listed in the output. The `/dev/sda1` on the end isn't necessary unless you are trying to force it to write to a particular disk e.g., an inserted USB or an SSD:

```
lvcreate -l 22717 -s /dev/VolGroup00/lvolhome -n lvolhomesnap /dev/sda1
```
Copy the snapshot over to the backup device:

```
dd if=/dev/mapper/VolGroup00-lvolhomesnap of=/dev/mapper/VolGroup01-lvolbackup
```