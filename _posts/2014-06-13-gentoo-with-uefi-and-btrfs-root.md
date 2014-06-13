---
layout: post
published: true
---

**NOTE: This guide assumes basic knowledge of \*nix such as how to enter terminal commands.             

Get the [minimal install cd](http://www.gentoo.org/main/en/where.xml) for your architecture (e.g, amd64).

Put it on a USB pen:

    dd if=install-ARCHITECTURE-minimal-DATE.iso of=/dev/sdX bs=4M && sync
    
... remember to replace the iso filename with the one you downloaded and the X of *sdX* with your drive letter. 

Find out what key to press to get the boot menu up on your BIOS, then reboot and select the USB pen from the menu. 