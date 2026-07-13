# Day 01 — Embedded Linux and Yocto Introduction

## Four Embedded Linux Elements

Toolchain: The compiler and other tools needed to create code for your target device. Everything else depends on the toolchain.

Bootloader: The program that initializes the board and loads the Linux kernel.

Kernel: This is the heart of the system, managing system resources and interfacing with hardware.

Root filesystem: Contains the libraries and programs that are run once the kernel has completed its initialization.

One more element can be collection of programs specific to your embedded application which make the device do whatever it is supposed to do, be it weigh groceries, display movies, control a robot, or fly a drone.

## Yocto Terminology

What is Yocto?

Yocto is the smallest SI metric system prefix

Like 'm' stands for milli ( 10^-3 ).

So is yocto ( y = 10^-24 )

## Yocto Project Overview

What is Yocto Project
Yocto project provides open source, high quality infrastructure and tools to help developers create their own custom Linux distributions for any hardware architecture.

History
Founded in 2010,  in an effort to reduce work duplication, provide resources and information catering to both new and experienced users.

collaboration of
	many hardware manufacturers
	open source operating systems vendors and
	electronics companies

Yocto is also a project working group of the Linux Foundation

### Yocto Project History

Advantages of Yocto Project

1. Widely Adopted Across the Industry
	Semiconductor, operating system, software, and service vendors exist whose products and services adopt and support the Yocto Project.
	Eg. Intel, Facebook, arm, Juniper Networks, LG, AMD, NXP, DELL

2. Architecture Agnostic
	supports Intel, ARM, MIPS, AMD, PPC and other architectures
	chip vendors create and supply BSPs that support their hardware
	if you have custom silicon, you can create a BSP that supports that architecture
	the Yocto Project fully supports a wide range of device emulation through the Quick EMUlator (QEMU)

3. Images and Code Transfer Easily
	Yocto Project output can easily move between architectures without moving to new development environments.

4. Flexibility
	Through customization and layering, a project group can leverage the base Linux distribution to create a distribution that works for their product needs.

5. Ideal for Constrained Embedded and IoT devices
	Unlike a full Linux distribution, you can use the Yocto Project to create exactly what you need for embedded devices
	You only add the feature support or packages that you absolutely need for the device

6. Uses a Layer Model
	The Yocto Project layer infrastructure groups related functionality into separate bundles.
	You can incrementally add these grouped functionalities to your project as needed
	Allows you to easily extend the system, make customizations, and keep functionality organized.

---

## Yocto Advantages

What is Yocto Project

To understand the outcome provided by the Yocto Project, we can use the analogy of the computing machine

Input: Set of data that describes what we want, that is our specification 
		(Kernel Configuration, Hardware Name, Packages/Binaries to be installed)

Output: Linux Based Embedded Product 
		(Linux Kernel, Root File System, Bootloader, Device Tree, Toolchain)

---

## Yocto Input Output Model

Setting up Build Machine

Prerequisites

1. 50 Gbytes of free disk space

2. Runs a supported Linux distribution (i.e. recent releases of Fedora, openSUSE, CentOS, Debian, or Ubuntu). 

3. 
	Git 1.8.3.1 or greater

	tar 1.27 or greater

	Python 3.4.0 or greater.

Packages and package installation vary depending on your development system.

(*) Install the required packages for Yocto to Work from
        https://www.yoctoproject.org/docs/latest/ref-manual/ref-manual.html#ubuntu-packages

$ sudo apt-get install gawk wget git-core diffstat unzip texinfo gcc-multilib \
    build-essential chrpath socat cpio python python3 python3-pip python3-pexpect \
    xz-utils debianutils iputils-ping python3-git python3-jinja2 libegl1-mesa libsdl1.2-dev \
    pylint3 xterm

sudo apt-get install gawk wget git-core diffstat unzip texinfo gcc-multilib \
    build-essential chrpath socat cpio python3 python3-pip python3-pexpect \
    xz-utils debianutils iputils-ping python3-git python3-jinja2 libegl1-mesa libsdl1.2-dev \
    pylint3 xterm

## Build Machine Setup

Poky

Poky is a reference distribution of Yocto Project. The word "reference" is used to mean "example" in this context.

Yocto Project uses Poky to build images (kernel, system, and application software) for targeted hardware.

At the technical level it is a combined repository of the components
	• Bitbake
	• OpenEmbedded Core
	• meta-yocto-bsp
	• Documentation

Note: Poky does not contain binary files,it is a working example of how to build your own custom Linux distribution from source.

What is difference between poky and Yocto

The exact difference between Yocto and Poky is Yocto refers to the organization ( like one would refer to 'canonical', the company behind Ubuntu ), and Poky refers to the actual bits downloaded ( analogous to 'Ubuntu' )

---

## Poky Reference Distribution

Metadata 

Non Yocto:  A set of data that describes and gives information about other data

Yocto World: 
	Metadata refers to the build instructions
	Commands and data used to indicate what versions of software are used
	Where they are obtained from
	Changes or additions to the software itself ( patches ) which are used to fix bugs or customize the software for use in a particular situation

Metadata is collection of
	• Configuration files (.conf)
	• Recipes (.bb and .bbappend)
	• Classes (.bbclass)
	• Includes (.inc)

---

## Metadata Definition

OpenEmbedded Project

From http://www.openembedded.org/wiki/Main_Page

OpenEmbedded offers a best-in-class cross-compile environment. It allows developers to create a complete Linux Distribution for embedded systems.

What is the difference between OpenEmbedded and the Yocto Project?

The Yocto Project and OpenEmbedded share a core collection of metadata called openembedded-core. 

However, the two organizations remain separate, each with its own focus

OpenEmbedded provides a comprehensive set of metadata for a wide variety of architectures, features, and applications

	Not a reference distribution
	Designed to be the foundation for others

The Yocto Project focuses on providing powerful, easy-to-use, interoperable, well-tested tools, metadata, and board support packages (BSPs) for a core set of architectures and specific boards.


OpenEmbedded-Core(oe-core)

The Yocto Project and OpenEmbedded have agreed to work together and share a common core set of metadata(recipes, classes and associated files): oe-core

---

## Openembedded Core

Bitbake

Bitbake is a core component of the Yocto Project.

It basically performs the same functionality as of make.

It's a task scheduler that parses python and shell script mixed code

The code parsed generates and runs tasks, which are basically a set of steps ordered according to code's dependencies.

It reads recipes and follows them by fetching packages, building them and incorporating the results into bootable images.

It keeps track of all tasks being processed in order to ensure completion, maximizing the use of processing resources to reduce build time and being predictable.

---

## Bitbake Build System

meta-yocto-bsp

A Board Support Package (BSP) is a collection of information that defines how to support a particular hardware device, set of devices, or hardware platform

The BSP includes information about the hardware features present on the device and kernel configuration information along with any additional hardware drivers required.

The BSP also lists any additional software components required in addition to a generic Linux software stack for both essential and optional platform features.

The meta-yocto-bsp layer in Poky maintains several BSPs such as the Beaglebone, EdgeRouter, and generic versions of both 32-bit and 64-bit IA machines.

Machines supported
Texas Instruments BeagleBone (beaglebone)
Freescale MPC8315E-RDB (mpc8315e-rdb)
Intel x86-based PCs and devices (genericx86 and genericx86-64)
Ubiquiti Networks EdgeRouter Lite (edgerouter)

Note: To develop on different hardware, you will need to complement Poky with hardware-specific Yocto layers.

---

## Meta Yocto Bsp Machines

Others

meta-poky, which is Poky-specific metadata

Documentation, which contains the Yocto Project source files used to make the set of user manuals.

Conclusion

Poky includes 
	some OE components(oe-core)
	bitbake
	demo-BSP's
	helper scripts to setup environment
	emulator QEMU to test the image

![poky-reference-distribution.png](day01_embedded_linux_and_yocto_intro/10_meta_yocto_bsp_machines/poky-reference-distribution.png)

---

## Poky Structure Conclusion

Workflow of Yocto Project

Step 1: Download the Poky Source code

$ git clone git://git.yoctoproject.org/poky

Step 2: Checkout the latest branch/release (zeus)

$ git checkout zeus 

Step 3: Prepare the build environment

Poky provides you a script 'oe-init-build-env', which should be used to setup the build environment

script will set up your environment to use Yocto build system,including adding the BitBake utility to your path

$ source poky/oe-init-build-env [ build_directory ]

Here build_directory is an optional argument for the name of the directory where the environment is set

in case it is not given , it defaults to "build"

The above script will move you in a build folder and create two files ( local.conf, bblayers.conf ) inside conf folder

Step 4: Building Linux Distribution

$ bitbake <image_name>
$ time bitbake core-image-minimal

core-image-minimal

This is a small image allowing a device to boot, and it is very useful for kernel and boot loader tests and development

---

## Yocto Workflow Steps

Command to run the generated image in QEMU

Quick Emulator ( QEMU ) is a free and open source software package that performs hardware virtualization.

The QEMU based machines allow test and development without real hardware.

Currently emulations are supported for:
        • ARM
        • MIPS
        • MIPS64
        • PowerPC
        • X86
        • X86_64


Poky provides a script 'runqemu' which will allow you to start the QEMU using yocto generated images.

The runqemu script is run as:

   runqemu <machine> <zimage> <filesystem>

where:

   <machine> is the machine/architecture to use (qemuarm/qemumips/qemuppc/qemux86/qemux86-64)
   <zimage> is the path to a kernel (e.g. zimage-qemuarm.bin)
   <filesystem> is the path to an ext2 image (e.g. filesystem-qemuarm.ext2) or an nfs directory

Full usage instructions can be seen by running the command with no options specified.

Exit QEMU

Exit QEMU by either clicking on the shutdown icon or by typing Ctrl-C in the QEMU transcript window from which you evoked QEMU.

---

## Qemu Emulation

Steps to generate ARM image and run in QEMU

When you set up the build environment, a local configuration file named local.conf becomes available in a conf subdirectory of the Build Directory

The defaults are set to build for a qemux86-64 target

Edit ./build/conf/local.conf

Set

MACHINE = "qemuarm"

$ source poky/oe-init-build-env

$ bitbake core-image-minimal

$ runqemu core-image-minimal

---

## Arm Image Qemu

Nographic

You can launch QEMU without the graphic window by adding nographic to the command line

$ runqemu qemuarm nographic

---

## Package Installation

To add a particular package in your root file system

Open your local.conf file and add the recipe name below

IMAGE_INSTALL += "recipe-name"

E.g. IMAGE_INSTALL += "usbutils" for lsusb

or IMAGE_INSTALL_append = " usbutils"

---

## Core Image Sato

core-image-sato

This is the X11 Window-system-based image with a SATO theme and a GNOME mobile desktop environment

$ bitbake core-image-sato

---

## Challenge Qemu Mips

### Challenge

Challenge

What changes you make to generate an image for qemu mips?

Once generated run the image in qemu emulator

---
