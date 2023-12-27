# x51_tools

Development/research repo for Powkiddy X51 console.
Most of this information should also apply to other ATM7051 devices.


### Technical specs

- **CPU**: Actions ATM7051
- **RAM**: 128MB DDR3
- **Screen**: 5 inches 800x480/816x480
- **Ports**: USB-C power/data, headphone jack 3.5mm, mini HDMI, microSD slot, 2 x USB
- **Buttons**: volume wheel, power, reset, function, gamepad buttons
- **Speaker**: 2 speakers
- **Battery**: 3000 mAh


### Hardware

- **PCB**: LLW7051-047 V7.0 2023-08-02
- **LCD**: no references, 50 to 40 pin adapter
- **CPU**: Actions ATM7051H
- **Flash**: [Zetta ZDND2G08U3D-IA](http://www.zettadevice.com/upload/file/pdf/DS_Zetta_2Gb_PPI_NAND_Flash%20(Rev_A).pdf)
- **RAM**: [Nanya NT5CB128M16BP-CG](https://www.datasheets360.com/pdf/2902671752543230045)
- **Audio codec**: Actions ATT3002


### Software information

- **Emulation software version**: 2.0.00.230822.1453 / 047V3r_st7701jc_V2.0
- **uname**: Linux (none) 3.10.52 #1 SMP PREEMPT Tue Aug 22 14:52:53 CST 2023 armv7l GNU/Linux
- **/proc/version**: Linux version 3.10.52 (sjg@Server100) (gcc version 4.8.3 (OpenWrt/Linaro GCC 4.8-2014.04 r49389) ) #1 SMP PREEMPT Tue Aug 22 14:52:53 CST 2023


### Default system structure

Factory system on flash:

	Byte offset	Name		Device		Mountpoint
	0		bootloader
	524288		uboot
	4194304		misc		/dev/nand0p1
	12582912	recovery	/dev/nand0p2
	23068672	config		/dev/nand0p3
	23199744	system		/dev/nand0p4	/
	56754176	upgrade		/dev/nand0p5	/var/upgrade
	61997056	res		/dev/nand0p6	/usr/local
	93454336	udisk		/dev/nand0p7	/mnt/udisk

- **misc** is a FAT format filesystem containing boot files:
	battery_low.bmp.gz
	boot_logo.bmp.gz
	kernel.dtb
	ramdisk.img
	readme.txt
	uenv.txt
	uImage

- **recovery** is a FAT format filesystem containing alternative boot files:
	battery_low.bmp.gz
	boot_logo.bmp.gz
	kernel.dtb
	ota_initrafms.img
	readme.txt
	uImage

- **config** is a broken FAT format filesystem containing config files for emulation software.

- **system** contains a squashfs rootfs.

- **upgrade** is an empty filesystem mounted on /var/upgrade

- **res** is an squashfs filesystem containing resources for emulation software.

- **udisk** is a FAT format filesystem containing some additional emulation software configs.


### ADB access

Connect a USB male-male cable between PC and USB PORT2 on device. Then connect a USB-C cable to device and PC, accept connection with A button.
ADB is enabled by default, with a root shell.


### Firmware backup

Use dd to backup /dev/nand0 and/or /dev/nand0p*


### Charge mode

Following the same procedure for ADB access but with the device turned off will get it in "charge mode". System is booted in the same way, but the screen will be off and the CPU governor will be set to minimum.
You can turn on backlight (display will remain black as there is no graphical application running) with this command:

	echo 0 >/sys/class/backlight/owl_backlight/bl_power


### chroot

As "chroot" command is available, you can use it to chroot into different root file systems:

	mkdir /tmp/share
	mount /dev/mmcblk0p1 /tmp/share
	mkdir /tmp/rootfs
	mount -o loop /tmp/share/other_rootfs.squashfs /tmp/rootfs
	mount -o bind /tmp /tmp/rootfs/tmp
	mount -o bind /dev /tmp/rootfs/dev
	mount -o bind /sys /tmp/rootfs/sys
	mount -o bind /proc /tmp/rootfs/proc
	mount -o bind /tmp/share /tmp/rootfs/userdata
	chroot /tmp/rootfs /bin/bash


### Actions fw packages

Firmware packages for modern Actions devices are distributed on a single file with ".fw" extension, being this file a proprietary Actions format.
Current "Pad Firmware Modify Tool 1.23" from Actions seems to not be compatible with non-Android packages as the one this device needs.
There are several unknown files with "3DUfw" header inside package, but misc, recovery, system and res file systems seems to be unencrypted inside this file, so theoretically a custom fw file could be generated (needs further investigation, maybe some kind of checksum/hash is involved).
