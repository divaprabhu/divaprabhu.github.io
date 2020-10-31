---
layout: post
date: 2016-10-02
title: Installing OpenBSD To USB Drive Using VirtualBox
tags: ["virtualbox", "openbsd"]
---

You have installed OpenBSD on your MacBook but want to install on your USB
drive for on the go usage. This procedure was tested using macOS Sierra.
Installation is done using UEFI partition table and with full disk encryption.

*Note:* The process may take much longer than the regular hard disk or
VirtualBox install since we are writing to the disk directly from inside the
VirtualBox.

## Before proceeding you will need
* A USB drive that can be formatted without worrying about the data
* Access to a machine with virtual box installed
* OpenBSD installer ISO file
* An optional USB wireless adapter. I am using TP-LINK WN725n

## Setting up the partition
You can use a spare partition entirely for OpenBSD or partition it into two
partitions: One regular Apple HFS partition that can be read/written in OS X
and another OpenBSD install partition. I have a 64GB drive which I am going to
split into two partitions.
```
	$ diskutil partitiondisk disk2 2 GPT jhfs+ MYNAME 50% jhfs+ OpenBSD 50%
```
The partition now looks like below
```
$ diskutil list disk2
/dev/disk2 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *62.2 GB    disk2
   1:                        EFI EFI                     209.7 MB   disk2s1
   2:                  Apple_HFS MYNAME                  31.0 GB    disk2s2
   3:                  Apple_HFS OpenBSD                 30.7 GB    disk2s3
```
## Giving VirtualBox access to raw USB disk
Now we will create a VirtualBox image file that represents our USB drive. But
before proceeding, change the ownership of the device file to regular user. By
default it is owned by root and VirtualBox will have permission issues to access
it. Change the ownership to regular user.
```
	$ sudo chown youruserid /dev/disk2*
```
Now unmount the USB drive
```
	$ diskutil umount disk2s1
	disk2s1 was already unmounted
	$ diskutil umount disk2s2
	Volume MYNAME on disk2s2 unmounted
	$ diskutil umount disk2s3
	Volume OpenBSD on disk2s3 unmounted
```	
At any point during install, if VirtualBox gives permission or access issues,
repeat this step.

## Set up VirtualBox
Create an image that represents the USB drive
```
	$ VBoxManage internalcommands createrawvmdk -filename obsd.vmdk -rawdisk /dev/disk2
```
Now open up VirtualBox and create a virtual machine for OpenBSD. Here are the
quick notes

..- Create a new virtual machine. Select "Do not add virtual disk" at the
prompt. We can add it as SATA disk later
..- Once created, go to settings
Under System
..- Uncheck floppy and enable Hardware clock in UTC
..- In the processor tab, increase the processor to 2, so that OpenBSD will
install a multiprocessor kernel
Under Storage
..- Select the OpenBSD installer ISO file as the optical disk
..- Add a SATA controller and add a hard disk choosing the obsd.vmdk file we
created. If VirtualBox gives any error message, run chown and unmount commands
mentioned above and try again.
Under Networks
..- Enable network adapter
Under Ports
..- Go to USB and add the wireless USB adapter


## Install OpenBSD
Start the virtual machine and select (S)hell
Change the jhfs+ partition we created earlier to OpenBSD type(A6)
```
	# fdisk -e sd0
```
Create a soft raid partition
```
	# disklabel -E sd0
```
Create a device for sd1 which we will use as encrypted device
```
	# cd /dev
	# sh MAKEDEV sd1
```
Build the encrypted device
```
	# bioctl -c C -l sd0a softraid0
```
Enter the passphrase that will be used every time you boot to decrypt the drive.
Overwrite the first MB of the new device with zeroes
```
	# dd if=/dev/zero of=/dev/rsd1c bs=1m count=1
```
Exit from the shell and begin installation.  Do not worry about the wireless
adapter, we will get it to work later. We only need to get the VirtualBox
network adapter(usually em0) working at this point so we can update the firmware
later.  When selecting disks, make sure you select the encrypted drive which in
this case is sd1. Use the whole disk MBR partition.  Complete the installation
and reboot

## Post Installation
Once the installation is complete, reboot into the new installation using
VirtualBox. It will prompt for the passphrase OpenBSD should automatically
detect and install the firmware.

Now restart your computer and boot of the USB. You can refer to my previous
blog on how to configure the wireless adapter.

## References
1. [Virtualbox](https://www.virtualbox.org/manual/ch09.html)
2. [OpenBSD FAQ](href=https://www.openbsd.org/faq/faq14.html#softraid)
