# Day 02 — Yocto Terminology and Build Directory

## Yocto Terminology

### Metadata Overview

Metadata

Poky = Bitbake + Metadata

Metadata is collection of
	• Configuration files (.conf)
	• Recipes (.bb and .bbappend)
	• Classes (.bbclass)
	• Includes (.inc)

### Recipes Definition

Recipes

Non-Yocto: A recipe is a set of instructions that describe how to prepare or make something, especially a dish

Yocto: A recipe is a set of instructions that is read and processed by the bitbake

Extension of Recipe: .bb

A recipe describes:
    where you get source code
    which patches to apply
    Configuration options
    Compile options (library dependencies)
    Install
    License

a software component


Examples of Recipes

dhcp_4.4.1.bb
gstreamer1.0_1.16.1.bb

### Configuration Files

Configuration Files

Files which hold
    global definition of variables
    user defined variables and
    hardware configuration information

They tell the build system what to build and put into the image to support a particular platform

Extension: .conf

Types

	• Machine Configuration Options
	• Distribution Configuration Options
	• Compiler tuning options
	• General Common Configuration Options
	• User Configuration Options (local.conf)

### Class Files

classes

Class files are used to abstract common functionality and share it amongst multiple recipe (.bb) files

To use a class file, you simply make sure the recipe inherits the class

Eg. inherit classname

Extension: .bbclass

They are usually placed in classes directory inside the meta* directory

Example of classes

cmake.bbclass - Handles cmake in recipes
kernel.bbclass - Handles building kernels. Contains code to build all kernel trees
module.bbclass - Provides support for building out-of-tree Linux Kernel Modules

### Layers Concept

Layers

A collection of related recipes.

or

Layers are recipe containers (folders)

Typical naming convention: meta-<layername>

Poky has the following layers:

meta, meta-poky, meta-selftest, meta-skeleton, meta-yocto-bsp

Why Layers

 Layers provide a mechanism to isolate meta data according to functionality, for instance BSPs, distribution configuration, etc.

You could have a BSP layer, a GUI layer, a distro configuration, middleware, or an application

Putting your entire build into one layer limits and complicates future customization and reuse. 

Example: meta-poky          -- Distro metadata
         meta-yocto-bsp     -- BSP    metadata

Layers allow to easily to add entire sets of meta data and/or replace sets with other sets.
meta-poky, is itself a layer applied on top of the OE-Core metadata layer, meta

### Layers Management

Layers

Which layers are used by Poky build system?

BBLAYERS variable present in build/conf/bblayers.conf file list the layers Bitbake tries to find

If bblayers.conf is not present when you start the build, the OpenEmbedded build system creates it from bblayers.conf.sample 
when you source the oe-init-build-env script

Command to find out which layers are present

$ bitbake-layers show-layers

Note: You can include any number of available layers from the Yocto Project 

Where to get other layers

https://layers.openembedded.org/layerindex/branch/master/layers/

### Compatible Layers

Yocto Project Compatible Layers

https://www.yoctoproject.org/software-overview/layers/

These layers are tested and are fully compatible with yocto project.

OpenEmbedded layer index contains more layers but the content is less universally validated

![yoctoproject_compatible.png](day02_yocto_terminology_and_build_dir/0_yocto_terminology/07_compatible_layers/yoctoproject_compatible.png)

### Image Definition

Image

An image is the top level recipe, it has a description, a license and inherits the core-image class

It is used alongside the machine definition

machine describes the hardware used and its capabilities

image is architecture agnostic and defines how the root filesystem is built, with what packages.

By default, several images are provided in Poky

Command to check the list of available image recipes

$ ls meta*/recipes*/images/*.bb

### Packages Definition

Packages

Non-Yocto: Any wrapped or boxed object or group of objects.

Yocto: A package is a binary file with name *.rpm, *.deb, or *.ipkg

A single recipe produces many packages. All packages that a recipe generated are listed in the recipe variable


$ vi  meta/recipes-multimedia/libtiff/tiff_4.0.10.bb
PACKAGES =+ "tiffxx tiff-utils"

---

## Poky Source

### Poky Source Treeub

Poky source tree

bitbake		-	Holds all Python scripts used by the bitbake command
			bitbake/bin is placed into the PATH environmental variable so bitbake can be found

documentation   -       All documentation sources for the Yocto Project documentation
			Can be used to generate nice PDFs

meta		-	Contains the oe-core metadata
			
meta-poky	-	Holds the configuration for the Poky reference distribution
			local.conf.sample, bblayers.conf.sample are present here

meta-skeleton	- 	Contains template recipes for BSP and kernel development


meta-yocto-bsp  -	Maintains several BSPs such as the Beaglebone, EdgeRouter, 
			and generic versions of both 32-bit and 64-bit IA machines.

scripts		-	Contains scripts used to set up the environment, development tools,
			and tools to flash the generated images on the target.

LICENSE		-	The license under which Poky is distributed (a mix of GPLv2 and MIT).

---

## Build Dir

### Conf

#### Bblayers Conf Guide

bblayers.conf

The bblayers.conf file tells BitBake what layers you want considered during the build.

By default, the layers listed in this file include layers minimally needed by the build system

However, you must manually add any custom layers you have created

E.g: BBLAYERS = "\
                                  /home/linuxtrainer/poky/meta \
                                 /home/linuxtrainer/poky/meta-poky \
                                 /home/linuxtrainer/poky/meta-yocto-bsp \
                                 /home/linuxtrainer/poky/meta-mylayer \
                                 "
This example enables four layers, one of which is a custom user defined layer named "meta-mylayer"

#### Bitbake Threading Config

BB_NUMBER_THREADS

	Determines the number of tasks that Bitbake will perform in parallel
	Note: These tasks are related to bitbake and nothing related to compiling
	Defaults to the number of CPUs on the system
	$ bitbake -e core-image-minimal | grep ^BB_NUMBER_THREADS=


PARALLEL_MAKE

	Corresponds to the -j make option
	specifies the number of processes that GNU make can run in parallel on a compilation task
	Defaults to the number of CPUs on the system
	$ bitbake -e core-image-minimal | grep ^PARALLEL_MAKE=

#### Local Conf Guide

conf

When you run source poky/oe-init-build-env, it will create a "build" folder in that directory
Inside this build folder, it will create "conf" folder which contains two files:

1. local.conf
2. bblayers.conf

local.conf

Configures almost every aspect of the build system
Contains local user settings

MACHINE: The machine the target is built for
	Eg: MACHINE = "qemux86-64"

DL_DIR: Where to place downloads
	During a first build the system will download many different source code tarballs,from various 
	upstream projects.These are all stored in DL_DIR
	The default is a downloads directory under TOPDIR which is the build directory

TMP_DIR:  Where to place the build output
	  This option specifies where the bulk of the building work should be done and
	  where BitBake should place its temporary files(source extraction, compilation) and output

Important Point:

local.conf file is a very convenient way to override several default configurations over all the Yocto Project's tools.

Essentially, we can change or set any variable, for example, add additional packages to an image file

Though it is convenient, it should be considered as a temporary change as the build/conf/local.conf file is not usually tracked by any source code management system.

#### Question

Where should i place the content of conf/local.conf as this file is part of build folder?

#### Answer

Where should i place the content of conf/local.conf as this file is part of build folder?

In general, everything in your local.conf should be moved to your own distro configuration

Finally, you should only set DISTRO to your own distro in local.conf

### Others

Other directories

downloads	-	downloaded upstream tarballs/git repositories of the recipes used in the build

sstate-cache	-	shared state cache

tmp		-	Holds all the build system output

			tmp/deploy/images/machine - Images are present here
			
cache		-	cache used by the bitbake's parser

---

## Build Workflow

Yocto/OpenEmbedded Build System Workflow

1. Developers specify architecture, policies, patches and configuration details.

2. The build system fetches and downloads the source code from the specified location
	supports downloading tarballs and source code repositories systems such as git/svn

3. extracts the sources into a local work area

4. patches are applied

5. steps for configuring and compiling the software are run

6. installs the software into a temporary staging area
	depending on the user configuration, deb/rpm/ipk binaries are generated

7. the build system generates a binary package feed that is used to create the final root file image.

8. finally generates the file system image and a customized Extensible SDK (eSDK) for application development in parallel

![YP-flow-diagram.png](day02_yocto_terminology_and_build_dir/3_build_workflow/YP-flow-diagram.png)

---

## Images

Images generated by Poky Build

The build process writes images out to the Build Directory inside the tmp/deploy/images/machine/ folder

1. kernel-image: 
		A kernel binary file
		The KERNEL_IMAGETYPE variable determines the naming scheme for the kernel image file.
		$ bitbake -e core-image-minimal | grep ^KERNEL_IMAGETYPE=

2. root-filesystem-image: 
		Root filesystems for the target device (e.g. *.ext3 or *.bz2 files).
		The IMAGE_FSTYPES variable determines the root filesystem image type
		$ bitbake -e core-image-minimal | grep ^IMAGE_FSTYPES=

3. kernel-modules:
		Tarballs that contain all the modules built for the kernel

4. bootloaders:
		If applicable to the target machine, bootloaders supporting the image.

symlinks
	symbolic link pointing to the most recently built file for each machine
	These links might be useful for external scripts that need to obtain  latest version of each file

---

## Savedisk Space

Saving Disk Space while building Yocto 

Yocto Build System can take a lot of disk space during build. But bitbake provides options to preserve disk space

You can tell bitbake to delete all the source code, build files after building a particular recipe by adding the following line in local.conf file

INHERIT += "rm_work"

Disadvantage: Difficult to debug while build fails of any recipe.

For example, if you want to exclude bitbake deleting source code of a particular package, you can add it in RM_WORK_EXCLUDE += "recipe-name"

E.g: RM_WORK_EXCLUDE += "core-image-minimal"

---

## Challenge

What is the difference between ipkg/deb/rpm etc?
