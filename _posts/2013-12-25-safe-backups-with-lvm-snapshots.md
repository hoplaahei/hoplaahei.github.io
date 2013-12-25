---
layout: post
published: true
---

If you have free unpartitioned space on your LVM disk extend the logical device with:

````
lvextend
```
If there is no free space left for logical volumes then add a physical disk to the setup e.g., insert a USB pen and create a partition type LVM with hexcode 8e00 on it.

e.g., in gdisk:

Type `gdisk /dev/sdb` then `n`, `Enter`, `Enter`, `+5G`, `8e00`