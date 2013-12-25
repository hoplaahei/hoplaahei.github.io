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
vgextend YourVolGroup /dev/sdb1
```