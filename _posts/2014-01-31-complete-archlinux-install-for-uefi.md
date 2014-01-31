---
layout: post
published: true
---

This guide requires knowledge of basic Linux Console commands.

## Download the iso

Choose your nearest [mirror](https://www.archlinux.org/download/) to get the dual ISO in the fastest possible time.

## Copy ISO to USB pen

In the past, extra steps were needed to make the ISOs UEFI bootable, but now a `dd` command will suffice:

```bash
dd bs=4M if=/path/to/archlinux.iso of=/dev/sdX && sync # where X is your device number
```

If unsure which `sdX` device to copy the ISO too, run `dmesg | tail` just after plugging it in. 