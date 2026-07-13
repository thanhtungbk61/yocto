# Day 05 — Raspberry Pi Support

## Raspberry Pi3 Specs

Raspberry Pi 3
SoC: Broadcom BCM2837
CPU: 4× ARM Cortex-A53, 1.2GHz
GPU: Broadcom VideoCore IV
RAM: 1GB LPDDR2 (900 MHz)
Networking: 10/100 Ethernet, 2.4GHz 802.11n wireless
Bluetooth: Bluetooth 4.1 Classic, Bluetooth Low Energy
Storage: microSD
GPIO: 40-pin header
Ports: HDMI, 3.5mm analogue audio-video jack, 4× USB 2.0, Ethernet, Camera Serial Interface (CSI), Display Serial Interface (DSI)

---

## Meta Raspberrypi Layer

meta-raspberrypi

Hardware specific BSP overlay for the RaspberryPi device.

The core BSP part of meta-raspberrypi should work with different OpenEmbedded/Yocto distributions and layer
stacks, such as:
	• Distro-less (only with OE-Core).
	• Angstrom.
	• Yocto/Poky

Hardware Supported

Raspberry Pi
Raspberry Pi 2
Raspberry Pi 3
Raspberry Pi 4
Raspberry Zero
Raspberry Zero Wireless

Link: git://git.yoctoproject.org/meta-raspberrypi

Dependencies

URI 	: git://git.openembedded.org/meta-openembedded
Layers	: meta-oe, meta-multimedia, meta-networking, meta-python

---

## Build Image Raspberry Pi

Steps to generate an Yocto Image for Raspberry Pi3

Poky has no support for Broadcom BCM SoC.

Step1: Download the BSP Layer for Raspberry Pi3

$ git clone git://git.yoctoproject.org/meta-raspberrypi

Step2: Download meta-openembedded branch as meta-raspberrypi layer depends upon it

$ git clone git://git.openembedded.org/meta-openembedded

Step3: Once you complete cloning, there should be three folders: poky, meta-openembedded, meta-raspberrypi

$ ls source

Step4: Run the environment script to setup the Yocto Environment and create build directory

$ source poky/oe-init-build-env build_pi

Step5: Add meta-openembedded layers ( meta-oe, meta-multimedia, meta-networking, meta-python) and meta-raspberrypi layer to bblayers.conf

Step6: Set the MACHINE in local.conf to "raspberrypi3".
$ echo 'MACHINE = "raspberrypi3"' >> conf/local.conf

Step7: Final step is to build the image. To find out the available images:
$ ls ../meta-raspberrypi/recipes-*/images/

---

## Raspberry Pi Images

Images

rpi-hwup-image.bb: This is an image based on core-image-minimal

rpi-basic-image.bb: This is an image based on rpi-hwup-image.bb, with some added features (a splash screen)

rpi-test-image.bb: This is an image based on rpi-basic-image.bb, which includes some packages present in meta-raspberrypi

Enabling UART
RaspberryPi 3 does not have the UART enabled by default because this needs a fixed core frequency and enable_uart wil set it to the minimum

To enable it, set the following in local.conf

ENABLE_UART = "1"

$ bitbake rpi-hwup-image

---

## Raspberry Pi Boot Stage1 Rom

Booting Process in Raspberry Pi3

Broadcom BCM2837:
	CPU 	-	1.2GHz 64-bit quad-core ARMv8 Cortex-A53
	GPU 	-	Broadcom VideoCore IV
	SDRAM	-	1024 MiB

When you power the device
Stage 1 Booting - OnChip ROM
	GPU - On, CPU - Off, SDRAM - Off

	When the Pi is powered up the first thing that starts working is the GPU core and ARM CPU is held in reset

	The GPU core is the main part responsible for the first booting steps of the Pi. 

	Inside Broadcom’s SoC, there is a small boot ROM, its programmed by default to look for a file called
	bootcode.bin on the SD Card

	It loads bootcode.bin into L2 Cache memory

---

## Raspberry Pi Boot Stages2 3

Booting Process in Raspberry Pi3

Stage 2 Booting - bootcode.bin

	GPU - On, CPU - Off, SDRAM - On

	Executed on VideoCore GPU
	Enables SDRAM
	Loads the third stage bootloader start.elf into SDRAM

Stage 3 Booting - start.elf

	GPU - On, CPU - On, SDRAM - On

	start.elf is actually a full fledge proprietary operating system for GPU known as VCOS (VideoCore OS).
	It starts by reading config.txt  which contains configuration parameters for
		VideoCore (Video/HDMI modes, memory, console frame buffers etc) and
		Linux Kernel (load addresses, device tree, uart/console baud rates etc)
		kernel = kernel7.img
	It also loads a file cmdline.txt if it exists, and will pass its contents as kernel command line
	Finally enables the ARM CPU and loads the kernel image which is executed on the ARM CPU
	It also loads the dtb file

---

## Raspberry Pi Boot Summary

Booting in Raspberry Pi3

GPU Core
first stage bootloader, which is stored in ROM on the SoC
bootcode.bin
start.elf
config.txt
cmdline.txt
kernel7.img

---

## Flashing Raspberry Pi Image

Flashing images on SD Card

Images are present in tmp/deploy/images/raspberrypi3

$ lsblk

$ sudo dd if=rpi-hwup-image-raspberrypi3.rpi-sdimg of=/dev/sdb bs=4096 && sync

---

## Raspberry Pi Hardware Setup

Hardware setup

An SD card with image flashed

Raspberry Pi 3

A power adapter that can supply 5V

USB TTL-2303(PL2303) for serial communication

USB-TTL is connected to the J8 connector of Raspberry Pi3 in the following formation:

J8 Pin           USB TTL Function
-------		-----------------
6                GND Ground

8                RXL

10               TXL

![raspberry_pi_3_serial.png](day05_raspberry_pi_support/09_raspberry_pi_hardware_setup/raspberry_pi_3_serial.png)

---

## Raspberry Pi Zero W

Raspberry Pi Zero W and Yocto

	1GHz, Single-core CPU
	512MB RAM
	Mini HDMI and USB On-The-Go ports
	Micro USB power
	HAT-compatible 40-pin header
	Composite video and reset headers
	CSI camera connector
	802.11n wireless LAN
	Bluetooth 4.0

Steps for generating Yocto Image

Everything is same except MACHINE = 'raspberrypi0-wifi'

---

## Remote Ssh Access

Remotely access raspberry pi3

$ bitbake rpi-basic-image

dropbear is a SSH 2 server designed to be small enough to be used in small memory environments

---

## Rootfs Extra Space

IMAGE_ROOTFS_EXTRA_SPACE

Adds extra free space to the root filesystem image.

The variable specifies the value in kilobytes

Default value : 0

For example, to add an additional 4 GB of space, set the variable to IMAGE_ROOTFS_EXTRA_SPACE = "4194304"

---
