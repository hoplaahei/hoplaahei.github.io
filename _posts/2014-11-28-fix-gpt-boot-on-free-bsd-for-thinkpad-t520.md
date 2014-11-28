---
layout: post
published: true
---

There is a long withstanding bug in the bioses for T520, T420 and W520 that prevents the booting of operating systems that use a GPT partition scheme with type `ee00` (protective MBR) as the first partition. [Many](http://forums.lenovo.com/t5/Linux-Discussion/Lenovo-Thinkpad-T520-doesn-t-boot-with-GPT-slices-on-FreeBSD-9/td-p/555317) users have this problem. 

I've learned about a hackish way to get GPT booting in legacy mode, and you may wish to skip straight to those steps, but carry on reading if you prefer a simpler solution that doesn't use GPT.

## Without GPT (simpler)

If you don't care about the [advantages](https://wiki.archlinux.org/index.php/GUID_Partition_Table#Advantages_of_GPT) of GPT, then the installer has the option to switch to BIOS partitioning, which should allow you to bypass the issue (though you will need to press `F1` at system boot, then goto `Startup` and change `UEFI/Legacy Boot` to `Both` or `Legacy Only`.

## With UEFI (without ZFS)

Another option is to boot from a UEFI installation medium and choose `UFS root` rather than `ZFS root` in the installer.

## The hackish way

None of the above solutions are acceptable to me. I want a ZFS root because I find it simpler to backup everything with ZFS, but I want to "have my cake and eat it" and stick with modern methods as well. For now I'll have to make do with legacy boot until they manage to get UEFI booting with a ZFS root, but I don't want the hassle of converting my partitions from MBR to GPT when they do eventually manage that. So the only other solution is to hack the GPT partition table for now to get it bootable.

Make sure your BIOS is UEFI enabled by pressing `F1` at system boot, choosing `Startup` and changing `UEFI/Legacy Boot` to `Both` or `UEFI Only`. If booting for a USB image, you also need to make sure `Config` -> `USB` -> `USB UEFI BIOS Support` is `Enabled`. Also, in the `Security` tab you must make sure `Secure Boot` is not enabled (it should be off by default). If you know how to compile C code there is even a [program](https://github.com/fpmurphy/UEFI-Utilities/blob/master/showlenovo/showlenovo.c) (needs gnu_efi package installed) to check if Secure Boot is enabled or disabled. Get one of the UEFI bootable live images and copy it to USB with `dd if=nameof.img of=/dev/yourdevice bs=1M && sync`. Now reboot and press `F12` to boot from the USB. Install the system as normal, being sure to select GPT partitioning over BIOS. When you reboot you will need to get back into the live USB and this time choose `Shell`.

You will need to replace `adaX` in the above command with e.g., `ada0` if it is your primary disk or `ada1` if it is the secondary. Run:

```
fdisk -p /dev/adaX > /tmp/parts
```

Open this file with an editor e.g., `ee /tmp/parts`:

```
    # /dev/ada0
    g c969021 h16 s63
    p 1 0xee 1 976773167
```

Your file likely won't look exactly like this, but it will look similar. Change `s63` to `s1`. Copy/paste the last line to make a 4th line. Change the `0xee` on the 3rd line to `0x00` and change `p 1` on the fourth line to 	`p 2`. The final product should look something like this:

```
    # /dev/ada0
    g c969021 h16 s63
    p 1 0x00 1 976773167
    p 2 0xee 1 976773167
```

Now run:

```
fdisk -f /tmp/parts /dev/adX
```

You've now modified the partition table so that partition one is an empty dummy partition, tricking the crappy Lenovo bios, and preventing it from borking at seeing an `ee00` protective MBR type partition as the first partition. 