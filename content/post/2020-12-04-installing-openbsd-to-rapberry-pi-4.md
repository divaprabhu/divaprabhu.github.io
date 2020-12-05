---
layout: post
date: 2020-12-04
title: Installing OpenBSD to Raspberry Pi 4
tags: ["raspberry pi", "openbsd"]
---

This is a three part series where I will explain how I setup my Raspberry Pi4 as a Wireguard VPN server with ad blocking using Unbound. In this first part we will install OpenBSD to Raspberry Pi 4. I will assume you have a basic understading of installing and using OpenBSD.

## What do you need?
1. A microSD card (only a small card is needed)
2. A USB storage device where you will install the OS
3. A USB to TTL serial cable
3. USB wireless adapter that is supported by OpenBSD (Optional. Only needed if you don't want to use the ethernet)
4. A second USB storage to store the sets and wireless firmware. (Optional. Only needed if you don't want to use the ethernet)

## Installing the UEFI firmware
1. Plugin the microSD card and format it with fat32. Mount it at /mnt
```
	doas fdisk -iy sd?
	doas fdisk -e sd? # change the partition type to 0b and write
	doas newfs_msdos /dev/rsd?i
	doas mount /dev/sd?i /mnt
```
2. Download the latest version of the RPi 4 UEFI firmware zip file from [GitHub](https://github.com/pftf/RPi4/releases/latest). 
3. Unzip the file and copy the contents to /mnt
```
	mkdir pftf && cd pftf
	ftp https://github.com/pftf/RPi4/releases/download/v1.21/RPi4_UEFI_Firmware_v1.21.zip
	unzip RPi4_UEFI_Firmware_v1.21.zip && rm RPi4_UEFI_Firmware_v1.21.zip
	doas cp -R * /mnt
	cd .. && rm -rf pftf
	doas umount /mnt
```
4. You can now remove the SD card

## Installing miniroot USB
1. Download the [miniroot68.img] (https://cdn.openbsd.org/pub/OpenBSD/6.8/arm64/miniroot68.img)
2. Verify the checksum
3. Use dd to copy it to USB. Later while installing the OS, you can use the same usb drive as target for install.
```
	ftp https://cdn.openbsd.org/pub/OpenBSD/6.8/arm64/miniroot68.img
	ftp https://cdn.openbsd.org/pub/OpenBSD/6.8/arm64/SHA256
	sha256 -C SHA256 miniroot68.img
	doas dd if=miniroot68.img of=/dev/rsd?c bs=1m
4. You can now remove the SD card
5. If you cannot connect your pi to ethernet, download the required install sets from the [mirror] (https://cdn.openbsd.org/pub/OpenBSD/6.8/arm64/) in advance. You can copy it to the spare USB drive.

## Connect the serial cable and change boot order
1. Connect serial cable pins to your board. If you hold your pi with GPIO pins on your right hand side, then pin 3, 4 and 5 on the right side are GND, TX and RX respectively.
2. Connect the USB to your machine's USB port. Run **cu** command to connect to the console. If you are connecting from a Linux machine, you may have to replace cuaU0 by something like ttyUSB0
```
	cu -s 115200 -l /dev/cuaU0
3. Plugin the USB with miniroot image and power on the pi. You will see the booot manager. 
4. Change the boot order to boot from USB. Save and continue
5. You should now get to the OpenBSD installer

Install OpenBSD to the USB drive. Install the sets either from network or from USB. Once you reboot into your newly installed system and update the WiFI firmware for your adapter if needed

## Reference
1. [OpenBSD/arm64 installation instructions] (https://ftp.openbsd.org/pub/OpenBSD/6.8/arm64/INSTALL.arm64)
