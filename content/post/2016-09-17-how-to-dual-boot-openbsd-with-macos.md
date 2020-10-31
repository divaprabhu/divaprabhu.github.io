---
layout: post
date: 2016-09-17
title: How To Dual Boot OpenBSD With macOS
tags: ["openbsd", "macos", "dualboot"]
---

This blog outlines installation of OpenBSD 6.0 alongside macOS using the native boot loader.

## Pre-install Tasks
First download a disk image of 64-bit OpenBSD installer(installXX.fs) from
[OpenBSD site](https://mirror.esc7.net/pub/OpenBSD/6.0/amd64/install60.fs) and
write it to a USB drive.
```
    $ sudo dd if=Downloads/install60.fs of=/dev/disk2 bs=1m
```
Broadcom wireless chips used in MacBook lacks a driver in OpenBSD. I used
TP-LINK TL-WN725N USB wireless adapter.  Download the urtwn firmware from
[OpenBSD site](http://firmware.openbsd.org/firmware/6.0/urtwn-firmware-1.2.tgz) 

Now we will mount the EFI partition and copy the firmware file downloaded to it.
```
    $ sudo mkdir /Volumes/efi
    $ sudo mount -t msdos /dev/disk0s1 /Volumes/efi
    $ sudo cp urtwn-firmware-1.2.tgz /Volumes/efi/
```
Now we will create two HFS partitions using the Disk Utility application in
macOS. Launch the utility. Select the main hard disk(not the partition). Do not
select FAT or exFAT. It did not allow me to create anything lesser than 2GB.
Follow the same steps as in 10 and create the second partition large enough to
hold OpenBSD.

## Disable SIP
macOS includes a security feature called SIP(System Integrity Protection). More
information can be obtained [rodsbooks](http://www.rodsbooks.com/refind/sip.html)
We will disable SIP as mentioned that link. Restart your system and press and
hold Command+R when you here the chime. Open a terminal window and type
```
    # csrutil disable.
```
## Install OpenBSD
Plugin your OpenBSD installer drive and the wireless USB adapter. Restart the
system. Press and hold option key when you hear the chime. Boot from EFI Boot. 
The fonts will be very small. But this will be only during install.

Follow the installation instruction from openbsd
[FAQ](https://www.openbsd.org/faq/faq4.html)

## Post Installation - on macOS
Run mount at the terminal and you should see the 2GB partition you created
mounted under /Volumes directory. We will first create a directory structure
similar to macOS boot.efi which is located at (/Volumes/Macintosh
HD/System/Library/CoreServices/boot.efi)
```
    $ cd /Volumes/OpenBSDBoot
    $ sudo mkdir System mach_kernel
    $ cd System
    $ sudo mkdir -p Library/CoreServices
```      
Now mount the EFI partition and copy the OpenBSD boot.efi. Since I am using 64
bit version, I will be copying bootx64.efi.
```
    $ sudo mkdir /Volumes/efi
    $ sudo mount -t msdos /dev/disk0s1 /Volumes/efi
    $ sudo cp /Volumes/efi/EFI/boot/bootx64.efi /Volumes/OpenBSDBoot/System/Library/CoreServices/
```              
Set the OpenBSD boot partition as one of the startup disk
```
    $ sudo bless --folder /Volumes/OpenBSDBoot --file /Volumes/OpenBSDBoot/System/Library/CoreServices/bootx64.efi
```                
Unplug the USB drive if it is still plugged in. Reboot the system and press and
hold option key when you here the chime. Select EFI boot and you are set to log
in to OpenBSD

## Post Install steps on OpenBSD
### Network Set Up
Once you boot into openbsd, login as root user mount the EFI file system and
copy the firmware.
```
    $ mount -t msdos /dev/sd0i /mnt
    $ cp /mnt/urtwn-firmware-1.2.tgz .
```        
### Install the firmware
```
    $ fw_update urtwn-firmware-1.2.tgz
```        
### Update the hostname.if file with wireless network details
```
    $ cat /etc/hostname.urtwn0
    nwid mywifissid
    wpakey mywifipassword
    dhcp
```
Connect to network with
```
    $ sh /etc/netstart urtwn0
```
From the subsequent boot, network should come up by itself

## References
1. [rodsbooks](http://www.rodsbooks.com/refind/sip.html)
2. [How to Dual Boot OSX and Ubuntu](http://courses.cms.caltech.edu/cs171/materials/pdfs/How_to_Dual-Boot_OSX_and_Ubuntu.pdf)
3. [OpenBSD FAQ](https://www.openbsd.org/faq/index.html)
