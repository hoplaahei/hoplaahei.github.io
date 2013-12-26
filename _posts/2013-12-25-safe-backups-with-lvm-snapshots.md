---
layout: post
published: true
---

Create a logical LVM volume to hold the snapshots. If you have free unpartitioned space on your LVM disk then extend the existing logical device with:

````
lvextend
```
If there is no free space left then add another disk to your existing LVM setup e.g., insert a USB pen and create a partition type LVM with hexcode 8e00 on it.

e.g., in gdisk:

Type `gdisk /dev/sdb` then `n`, `Enter`, `Enter`, `+5G`, `8e00`

Make it LVM ready and add it to the existing volume group (which you can find the name of by looking at the 'VG Name' field of `lvdisplay` command):

```
pvcreate /dev/sdb1
vgextend VolGroup00 /dev/sdb1
```
The new disk is now tacked onto the end of the LVM setup. No data is written to it though because it isn't mounted. 

Find out the current extent number of the LVM volume to be backed up:

```
lvdisplay -v /dev/VolGroup00/lvolhome`
```

Create a snapshot volume using the extent number. The `/dev/sda1` on the end isn't necessary unless you are trying to force it to write to a particular disk e.g., an inserted USB or an SSD:

```
lvcreate -l 22717 -s /dev/VolGroup00/lvolhome -n lvolhomesnap /dev/sda1
```

