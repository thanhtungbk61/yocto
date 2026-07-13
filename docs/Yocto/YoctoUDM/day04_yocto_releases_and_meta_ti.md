# Day 04 — Yocto Releases and meta-ti BSP Layer

## Yocto Releases

### Release Naming Convention

Yocto Project Releases

Naming Convention

Yocto Project releases follow the naming convention: Major.Minor.patch number

Eg. yocto-2.7.3

Major release number changes imply compatibility changes with previous releases

Minor release number changes imply significant changes up to, but not including compatibility changes

Minor rev number changes are for minor issues such as simple bugfixes, security updates, etc.

Poky Releases

Poky releases has code names as well as Major.Minor.patch numbering

Yocto Project/Poky Release, are released at the same time

### Yocto Releases Screenshot

![yocto_releases.PNG](day04_yocto_releases_and_meta_ti/1_yocto_releases/02_yocto_releases_screenshot/yocto_releases.PNG)

### Release Schedule Codenames

Yocto Project Releases

Major Releases: Every six months in April and October

Minor Releases: 
	No Particular schedule
	Driven by the accumulation of enough significant fixes or enhancements to the associated major release

Concept of codenames

Branches of metadata with the same codename are compatible with each other

### Milestone Point Releases

Types of Releases

a) Milestone Releases

Major Release is divided into milestone releases.

Milestone releases are used as:

	checkpoints for our progress

	preview for the public community

	to manage changes in the middle of a release 

Each milestone lasts about 4-8 weeks and has one planning week, several development weeks, stabilization weeks and one release week:

b) Point Releases

A point release is a minor release such as 1.0.x

Point releases are necessary to address the following:

	Issues building on current Linux distributions.
	fix critical bugs and security issues (CVEs) for the first 6 months after release
	fix security issues for the first year after release

![milestones.png](day04_yocto_releases_and_meta_ti/1_yocto_releases/04_milestone_point_releases/milestones.png)

### Release Lifecycle

Release Lifecycle

There are two possible lifecycles a release may follow:

Initial Release -> Stable -> Community -> EOL

	OR

Initial Release -> LTS	-> Community  -> EOL

Stable vs LTS Release Difference

Stable releases are maintained for seven months

LTS releases are maintained initially for two years

LTS releases are preannounced and known to be LTS in advance (usually every two years)

Community Support

Branch transitions to community support after the last stable dot release identified by the stable/LTS maintainer.

A call for a community support maintainer is sent to the mailing lists.
	A six (6) week waiting period.
	if there is no new maintainer, then the branch goes to into EOL status


Note:	Branches only have community support status if there is an active community member willing to step into the maintainer role for that series

### Layers And Branches

Layers and Branches

Layers being developed simultaneously by several different parties, the different flavors of Yocto and subsequent layers have to be split into branches in Git

Most BSP layers will only work with OpenEmbedded-Core of the same branch.

Mixing branches among your layers will end up in conflicts

For example, the so called bbappends, sometimes are tied to specific version number and will break the build if those versions are not found in the layers.

Which Branch to choose?

You should evaluate all the layers and find a compromise between:

    Getting the latest stable branch
    Getting the latest branch supported by all layers.

Note:  yocto branch you decided to use, could not be supported by your current host linux distribution.
Eg:  I want to use a layer which forces me to stick with Krogoth, but this branch is not tested with newer distributions such as Ubunutu 16.04 or 18.04.

---

## Meta Ti

### Bsp Layer Definition

#### Question

What is BSP Layer?

#### Answer

What is BSP Layer?

A collection of information(metadata) that defines how to support 
	a particular hardware device, 
	set of devices, or 
	hardware platform

### Bsp Naming Machines

BSP Layer

Naming Convention: meta-<bsp_name>

How to find out what all hardware devices are supported

conf/machine/*.conf list all the hardware devices supported by the BSP layer

### Meta Ti Layer

meta-ti

BSP Layer for Texas Instrument Hardware

$ cd sources

$ git clone git://git.yoctoproject.org/meta-ti

Hardware Configuration Supported

a) Beagle Bone Black

b) Beagle Board

c) Panda Board

d) OMAP boards

### Meta Ti Vs Meta Yocto Bsp

meta-ti vs meta-yocto-bsp

meta-yocto-bsp:

    provides "reference" BSPs for each of the supported architectures
    One for ARM (BeagleBone Black), one for MIPS, PPC and x86.
    it is based on the mainline kernel/bootloader
    does not support any advanced features or anything not in the upstream mainline kernel
    e.g. no capes, no power management, no hardware acceleration, no 3D, no PRU, etc.
    The purpose of this BSP is to have some basic out-of-box experience for the select hardware platforms within Poky to evaluate the Yocto Project and OpenEmbedded framework, but not the specific hardware platforms

meta-ti

    official Texas Instruments BSP that provides the latest WIP "staging" kernel and bootloader
    most of the advanced features and peripheral support for the wider range of latest TI platforms

### Adding Layers

Adding Layers

Two ways:

1. Manual: edit bblayers.conf file and add the new layer to BBLAYERS
2. Automatic: 

$ bitbake-layers add-layer <path-to-new-layer>

$ bitbake-layers add-layer ~/Yocto_Training/source/meta-ti/

### Build Steps Beaglebone

Steps for building

Step1 : Source the environment script

	$ source poky/oe-init-build-env

	Add the meta-ti layer

	$ bitbake-layers add-layer ~/Yocto_Training/source/meta-ti/

Step2 : Open local.conf and set Machine to beaglebone
	MACHINE='beaglebone'

Step3 : Also add INHERIT += "rm_work" to save disk space

Step4 : Generate an minimal image

	$ bitbake core-image-minimal

Step5 : Once the build finished, you will find the output images under $BUILDDIR/tmp/deploy/images/beaglebone

### Flashing With Wic

Flashing the image on the SD Card using wic

wic images are SD Card images and can be directly written into sd-card

core-image-minimal-beaglebone.wic.xz is compressed wic image.

It can be uncompressed using the unxz utility

$ unxz core-image-minimal-beaglebone.wic.xz

$ ls -lh core-image-minimal-beaglebone.wic

Flash it to the sd card

$ lsblk

$ sudo dd if=core-image-minimal-beaglebone.wic of=/dev/sdb bs=4096 status=progress && sync

---

## Challenge

Challenge

Generate an image for beaglebone using meta-ti, but remove meta-yocto-bsp layer
