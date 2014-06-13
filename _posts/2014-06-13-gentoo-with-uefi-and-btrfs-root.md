---
layout: post
published: false
---

**NOTE: This guide assumes basic knowledge of Linux, such as how to enter terminal commands.**             

Don't bother with the Gentoo minimal install cd; it's limited to a console and setting up wifi is tedious. Many Gentoo users use SystemRescueCD instead (which, at the time of writing, is Gentoo based). It has instructions for [copying to a USB pen](http://www.sysresccd.org/Sysresccd-manual-en_How_to_install_SystemRescueCd_on_an_USB-stick). Or burn the downloaded .iso to the CD with e.g., ``` sudo cdrecord nameof.iso```

Now it's time to reboot with the USB/CD in. If you want UEFI boot, you will need to look up howto force a UEFI boot in your bios options (though it may default to UEFI anyway). Upon reboot there is a key to press (F12 in many post-boot screens) to setup your BIOS. There is also a key (often F1) to show a menu of devices to boot from. Choose the USB/CD drive.

Press enter to boot, and then type ```startx``` from command-line to run the graphical environment. Use network-manager icon in the taskbar to setup wifi.

Now skip straight to Part 1, [Chapter 4](http://www.gentoo.org/doc/en/handbook/handbook-amd64.xml?part=1&chap=4) of the Gentoo Handbook.

I had some problems using ```gpart``` on the live iso. 

``` Fatal error: /dev(/dev/sdb): seek failure.```

I thought my disk might be mechanically damaged, but the same error appeared for the brand new ssd I'd just installed on /dev/sda, so I'm more inclined to believe this is a bug in gpart. Although, I suppose there could be something wrong with the SATA connection on the motherboard itself. Anyway, just a heads up in case you get the same errors as me. 

I decided to use gdisk instead.

```
Command (? for help): o
This option deletes all partitions and creates a new protective MBR.
Proceed? (Y/N): y

Command (? for help): n
Partition number (1-128, default 1): 
First sector (34-250069646, default = 2048) or {+-}size{KMGTP}: 
Last sector (2048-250069646, default = 250069646) or {+-}size{KMGTP}: 1050623
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300): EF00
Changed type of partition to 'EFI System'

Command (? for help): n
Partition number (2-128, default 2): 
First sector (34-250069646, default = 1050624) or {+-}size{KMGTP}: 
Last sector (1050624-250069646, default = 250069646) or {+-}size{KMGTP}: +9G
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300): 8200
Changed type of partition to 'Linux swap'

Command (? for help): n
Partition number (3-128, default 3): 
First sector (34-250069646, default = 19924992) or {+-}size{KMGTP}: 
Last sector (19924992-250069646, default = 250069646) or {+-}size{KMGTP}: 
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300): 
Changed type of partition to 'Linux filesystem'

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/sdb.
The operation has completed successfully.
```