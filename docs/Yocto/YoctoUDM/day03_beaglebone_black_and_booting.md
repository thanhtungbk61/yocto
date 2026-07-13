# Day 03 — BeagleBone Black and Booting

## Beaglebone Black Specs

Beagle Bone Black Specification

Texas Instruments AM335x (ARM Cortex-A8 CPU)

512MB DDR3 RAM

4 GB of on-board eMMC storage

3D graphics accelerator

NEON floating-point accelerator

2x PRU 32-bit micro-controllers

USB client for power & communications

USB host

Ethernet

HDMI (micro)

2x 46 pin headers with access to many expansion buses (I2C, SPI, UART and more)

A huge number of expansion boards, called capes

![beagle_bone_black.jpg](day03_beaglebone_black_and_booting/01_beaglebone_black_specs/beagle_bone_black.jpg)

---

## Build Image Beaglebone

Build Yocto Image for BeagleBone Black

BeagleBone is one of the reference boards of Yocto Project


$  source poky/oe-init-build-env build_bbb


Open build_bbb/local.conf file

        comment out the default selection, which is the qemux86_64
        uncomment the beaglebone selection

        MACHINE ?= "beaglebone-yocto"
        #MACHINE ??= "qemux86_64"

Trigger build
$  bitbake core-image-minimal

After the build is complete, you will have your images ready at tmp/deploy/images/beaglebone-yocto/

This folder contains
        first-level bootloader MLO,
        second-level bootloader u-boot,
        kernel image,
        device tree blobs,
        a root filesystem archive, and
        a modules archive.

---

## Booting Process Overview

Booting Process in Beaglebone black

The AM335x is complex piece of hardware, but has limited internal RAM (128 kB)

Because of this limited amount of RAM, multiple bootloader stages are needed

These bootloader stages systematically unlock the full functionality of the device so that all complexities of the device are available to the kernel

The AM335x has four distinct bootloader stages:


The four bootloader stages are:

1) ROM
2) SPL (or Secondary Program Loader)
3) U-BOOT
4) Linux Kernel

---

## Rom Bootloader Stage1

1st Stage Bootloader: ROM Code

The first stage bootloader is flashed in ROM on the device by Texas Instruments.

The ROM code is the first block of code that is automatically run on device start-up or after power-on reset (POR)

The ROM bootloader code is hardcoded into the device and cannot be changed by the user.

The ROM code has two main functions:

-Configuration of the device and initialization of primary peripherals
	Stack setup
	Configure Watchdog Timer 1 (set to three minutes)
	PLL and System Clocks configuration

- Ready device for next bootloader
	Check boot sources for next bootloader (SPL)
	Moves next bootloader code into memory to be run

The main purpose of the ROM code is to set up the device for the second stage bootloader.

By default, the ROM code in the Sitara AM3359 will boot from the MMC1 interface first (the onboard eMMC), followed by MMC0 (external uSD), UART0 and USB0.

If the boot switch (S2) is held down during power-up, the ROM will boot from the SPI0 Interface first, followed by MMC0 (external uSD), USB0 and UART0.

---

## Spl Bootloader Stage2

2nd Stage Bootloader: SPL

A fully featured version of U-Boot can be over 400KB, and the internal RAM on the AM335X is 128KB

Hence it is not possible to load this immediately

For this reason, a cut down version of U-Boot called U-Boot SPL (Second Program Loader) is loaded first,

once it has initialised the CPU, it chain loads a fully featured version of U-Boot (u-boot.img).

Name of SPL should be MLO

It should be located on active first partition of MMC, which must be formatted as FAT12/16/32

---

## Uboot Bootloader Stage3

3rd Stage Bootloader - U-Boot

U-BOOT allows for powerful command-based control over the kernel boot environment via a serial terminal

The user has control over a number of parameters such as boot arguments and the kernel boot command

In addition, U-boot environment variables can be configured. 

These environment variables are stored in the uEnv.txt file on your storage medium.

The built-in environment in u-boot loads a default am335x-boneblack.dts to pass to the kernel at boot.

In uEnv.txt you can explicitly specify a different DTS as well as the command line arguments to pass to the kernel

u-boot is also capable of obtaining network information via DHCP and loading it into environmental variables.

Finaly U-boot loads the kernel and a DTS into memory and boots the kernel with some command line arguments

The kernel initializes and mounts the root filesystem.

By default, the root filesystem is contained in the second partition (mmcblk0p2) of the microSD card, formatted for an ext3 file system.

---

## Bootloader Stages Reference

+--------+----------------+----------------+----------+
| Boot   | Terminology #1 | Terminology #2 | Actual   |
| stage  |                |                | program  |
| number |                |                | name     |
+--------+----------------+----------------+----------+
| 1      |  Primary       |  -             | ROM code |
|        |  Program       |                |          |
|        |  Loader        |                |          |
|        |                |                |          |
| 2      |  Secondary     |  1st stage     | u-boot   |
|        |  Program       |  bootloader    | SPL      |
|        |  Loader (SPL)  |                |          |
|        |                |                |          |
| 3      |  -             |  2nd stage     | u-boot   |
|        |                |  bootloader    |          |
|        |                |                |          |
| 4      |  -             |  -             | kernel   |
|        |                |                |          |
+--------+----------------+----------------+----------+


Reference: https://stackoverflow.com/questions/31244862/what-is-the-use-of-spl-secondary-program-loader

---

## Sd Card Partitioning

Creating partitions and formatting the SD card

1. Unmount any mounted partition, using the umount command:

$  umount /dev/sdb1

2. Launch the fdisk utitility and delete the previous partition(s); in our case, it is just one:

$  sudo fdisk /dev/sdb
Command (m for help): d
Selected partition 1

3. Create new partition called BOOT of 32 MB and type primary:

Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): 
Using default response p
Partition number (1-4, default 1): 
Using default value 1
First sector (2048-7774207, default 2048): 
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-7774207, default 7774207): +32M

4. Create a second partition to hold rootfs. We will give all the remaining space to this partition:

Command (m for help): n
Partition type:
   p   primary (1 primary, 0 extended, 3 free)
   e   extended
Select (default p): 
Using default response p
Partition number (1-4, default 2): 
Using default value 2
First sector (67584-7774207, default 67584): 
Using default value 67584
Last sector, +sectors or +size{K,M,G} (67584-7774207, default 7774207): 
Using default value 7774207

5. Make the first partition bootable by setting the boot flag:

Command (m for help): a
Partition number (1-4): 1

6. Set the first partition as WIN95 FAT32 (LBA):

Command (m for help): t Selected partition 1 Hex code (type L to list codes): c

7. We are done with the filesystem modification. So, let's write it by issuing the w command:

Command (m for help): w
The partition table has been altered!
Calling ioctl() to re-read partition table.
Syncing disks.

Tip
Do not forget to set the first partition as WIN95 FAT32 (LBA); otherwise, BeagleBone won't be able to boot from it. In this case, you might end up wasting time figuring out what's going wrong.

8. Format the first partition as FAT, using the following command. We will set the label as BOOT so that we know what directory it will be mounted to by udisks:

$  sudo mkfs.vfat -n "BOOT" /dev/sdb1

9. Format the second partition as an ext4 filesystem, using the following command. The label for this is set to ROOT, as it will contain the extracted image of rootfs.

$  sudo mkfs.ext4 -L "ROOT" /dev/sdb2

---

## Copy Images To Sdcard

### Copy Images Commands

Copying images to the card

We have formatted our card, according to the requirements. 

 Now, we are ready to populate images to it. 

The partitions are usually auto mounted under /media/$USER

If not, we can use the mount command to mount the partition to our desired location:

$ sudo mount /dev/sdb1 /media/$USER/BOOT
$ sudo mount /dev/sdb2 /media/$USER/ROOT

Now, follow these steps to copy images to the card:

1. Copy the u-boot MLO and u-boot bootloader images into the FAT32 partition:

$ sudo cp MLO /media/$USER/BOOT
$ sudo cp u-boot.img /media/$USER/BOOT

2. Copy the kernel image into the boot partition:


$ sudo cp zImage /media/$USER/BOOT 


3. Copy the .dtb file, am335x-boneblack.dtb, into the boot partition. This step is required only in the case of core-image-minimal. It is not required in our case, as we created core-image-sato, which already has this file placed at the desired location in rootfs:

$ sudo cp am335x-boneblack.dtb /media/$USER/BOOT 

4. As a root user, uncompress core-image-sato-beaglebone.tar.bz2 to the ext4 partition:

$ sudo tar -xf core-image-minimal-beaglebone-yocto.tar.bz2 -C /media/$USER/ROOT/
5. Unmount both partitions:

$ sudo umount /dev/mmcblk0p1
$ sudo umount /dev/mmcblk0p2

Remove the card from the host machine, and insert it into the SD card slot on BeagleBone Black.

### Gparted Partitioning

Gparted

#First partition
type: FAT32
size: around 30MB
label: BOOT
flags: boot
 
#Second partition
type: ext4
size: around 200MB, or rest of SD-card
label: ROOT

---

## Hardware Serial Setup

Hardware setup

An SD card with images flashed

BeagleBone Black

A power adapter that can supply 5V or a micro USB cable; we should use a 5V power adapter in order to avoid a decrease in the operating frequency

USB TTL-2303(PL2303) for serial communication

USB-TTL is connected to the J1 connector of BeagleBone in the following formation:

J1 Pin          USB TTL Function

1               GND Ground

4               RXL

5               TXL

![beaglebone_black_serial.jpg](day03_beaglebone_black_and_booting/10_hardware_serial_setup/beaglebone_black_serial.jpg)

---

## Minicom Serial Config

Serial setup

BeagleBone Black uses a serial debug port to communicate with the host machine. We will use minicom as a serial terminal client to communicate over the serial port. To set up minicom, perform the following steps:

1. Run this setup command as a privileged user:

$  sudo minicom -s

2. Press E to set the baud rate. Use the A and B keys to navigate the baud rate values. A corresponds to next and B to previous. Keep pressing B till you get 115200 8N1. Then, press Enter to choose this setting and go back to the previous menu.

3. Next, we need to press F and G to change enablement statuses of hardware flow control and software flow control. Both need to be set to No.

4. Choose Save setup as dfl to avoid reconfiguring every time and choose Exit to go to minicom. Don't exit from it if you want to observe whether there is any activity on the serial port.

---

## Booting Beaglebone

Booting BeagleBone

Now that we have everything set up, we are ready to boot.

We can just insert this card, and our board should boot from it. 

There might be only one issue if you have the eMMC (embedded MultiMediaCard) boot selected by default. 

You will have to disable it by booting up the board from the images you already have and renaming the MLO file from the eMMC partition

Alternatively, you can simply execute the following two commands on the u-boot prompt. 

 To stop at the u-boot prompt, simply press Enter after powering up the board before timeout:

# mmc dev 1
# mmc erase 0 512

The first command will select the eMMC card, and the second one will do the erasing so that BeagleBone doesn't try to boot from eMMC.

Insert our prepared SD card and power up BeagleBone. You should get an output similar to the following one on minicom:

Booting from mmc ...
## Booting kernel from Legacy Image at 82000000 ...
   Image Name:   Linux-3.14.0-yocto-standard
   Image Type:   ARM Linux Kernel Image (uncompressed)
   Data Size:    4985768 Bytes = 4.8 MiB
   Load Address: 80008000
   Entry Point:  80008000
   Verifying Checksum ... OK
## Flattened Device Tree blob at 88000000
   Booting using the fdt blob at 0x88000000
   Loading Kernel Image ... OK
   Loading Device Tree to 8fff5000, end 8ffff207 ... OK
Starting kernel …

Finally, we will land in our BeagleBone prompt:

Poky (Yocto Project Reference Distro) 1.6.1 beaglebone /dev/ttyO0
beaglebone login:

Enter root as user, and you are in as the root user:

root@beaglebone:~#

![bb_s2_switch.png](day03_beaglebone_black_and_booting/12_booting_beaglebone/bb_s2_switch.png)

---

## Challenge

Challenge

Do we have any other bootloader except u-boot for beagle bone black
