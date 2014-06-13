---
layout: post
published: false
---

**NOTE: This guide assumes basic knowledge of Linux, such as how to enter terminal commands.**             

Don't bother with the Gentoo minimal install cd; it's limited to a console and setting up wifi is tedious. Many Gentoo users use SystemRescueCD instead (which, at the time of writing, is Gentoo based). It has instructions for [copying to a USB pen](http://www.sysresccd.org/Sysresccd-manual-en_How_to_install_SystemRescueCd_on_an_USB-stick). Or burn the downloaded .iso to the CD with e.g., ``` sudo cdrecord nameof.iso```

Now it's time to reboot with the USB/CD in. If you want UEFI boot, you will need to look up howto force a UEFI boot in your bios options (though it may default to UEFI anyway). Upon reboot there is a key to press (F12 in many post-boot screens) to setup your BIOS. There is also a key (often F1) to show a menu of devices to boot from. Choose the USB/CD drive.

Press enter to boot, and then type ```startx``` from command-line to run the graphical environment. Use network-manager icon in the taskbar to setup wifi.

Now skip straight to Part 1, [Chapter 4](http://www.gentoo.org/doc/en/handbook/handbook-amd64.xml?part=1&chap=4) of the Gentoo Handbook.