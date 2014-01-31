---
layout: post
published: true
---

## Download the iso

Choose your nearest [mirror](https://www.archlinux.org/download/)to get the dual.iso in the fastest possible time.

## Copy ISO to USB pen

At one time extra steps were needed to make the ISOs UEFI bootable, but now a `dd` command will suffice:

```
dd bs=4M if=/path/to/archlinux.iso of=/dev/sdx && sync
```