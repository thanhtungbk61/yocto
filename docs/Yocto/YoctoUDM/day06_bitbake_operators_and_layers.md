# Day 06 — BitBake Operators and Layer Creation

## Operators

### Basic Assignment

Basic Variable Setting (=)

VARIABLE = "value"

The following example sets VARIABLE to "value".

Eg. MACHINE = "raspberrypi3"

This assignment occurs immediately as the statement is parsed

It is a "hard" assignment

If you include leading or trailing spaces as part of an assignment, the spaces are retained:

VARIABLE = " value"
VARIABLE = "value "

Note: You can also use single quotes ('') instead of double quotes when setting a variable's value
Benefit:

VARIABLE = 'I have a " in my value'

### Check Variable Value

#### Question

How to check the value of variable?

#### Answer

How to check the value of variable?

For configuration changes, use the following:

$ bitbake -e

This command displays variable values after the configuration files (i.e. local.conf, bblayers.conf, bitbake.conf and so forth) have been parsed.

For recipe changes, use the following

$ bitbake recipe -e | grep VARIABLE="

### Line Joining Backslash

Line Joining

BitBake joins any line ending in a backslash character ("\") with the following line before parsing statements

The most common use for the "\" character is to split variable assignments over multiple lines

BBLAYERS ?= " \
  /home/linuxtrainer/Yocto_Training/source/poky/meta \
  /home/linuxtrainer/Yocto_Training/source/poky/meta-poky \
  /home/linuxtrainer/Yocto_Training/source/poky/meta-yocto-bsp \
  "

$ bitbake -e | grep ^BBLAYERS

### Soft Assignment Question Eq

Setting a default value(?=)

?= is used for soft assignment for a variable

What's the benefit?

Allows you to define a variable if it is undefined when the statement is parsed,

If the variable has a value, then the soft assignment is lost

Eg:

MACHINE ?= "qemuarm"

If MACHINE is already set before this statement is parsed, the above value is not assigned

If MACHINE is not set, then the above value is assigned

Note: Assignment is immediate

What happens if we have multiple ?=
If multiple "?=" assignments to a single variable exist, the first of those ends up getting used

### Weak Default Assignment

Setting a weaker default value (??=)

Weaker default value is achieved using the ??=  operator

Difference between ?= and ??=

Assignment is made at the end of the parsing process rather than immediately.

When multiple "??=" assignments exist, the last one is used

Eg.

MACHINE ??= "qemux86"
MACHINE ??= "qemuarm"

If MACHINE is not set, the value of MACHINE = "qemuarm"
If MACHINE is set, before the statements, then the value will not be changed

It is called weak assignment, as assignment does not occur until the end of the parsing process.

Note: "=" or "?=" assignment will override the value set with "??="

### Variable Expansion Syntax

Variable Expansion

Variables can reference the contents of other variables using a syntax that is similar to variable expansion in Bourne shells

A = "hello"
B = "${A} world"

$ bitbake -e | grep ^A=
$ bitbake -e | grep ^B=

The "=" operator does not immediately expand variable references in the right-hand side

Instead, expansion is deferred until the variable assigned to is actually used

A = "${B} hello"
B = "${C} world"
C = "linux"

$ bitbake -e | grep ^A=

What happens if C is not defined in above?
the string is kept as is

### Immediate Expansion Colon Eq

Immediate Variable Expansion (:=)

The ":=" operator results in a variable's contents being expanded immediately, rather than when the variable is actually used

A = "11"
B = "B:${A}"
A = "22"
C := "C:${A}"
D = "${B}"


A = "11"
B := "B:${A}"
A = "22"
C := "C:${A}"
D = "${B}"

### Append Operators

Appending operators

+=

A = "hello"
A += "world"

.=

A = "hello"
A .= "world"

Difference between += and .= is space is automatically added in += 

These operators take immediate effect during parsing

### Prepend Operators

Prepending Operators

=+

A = "world"
A =+ "hello"

=.

A = "world"
A =. "hello"

Same as previous, =+ adds an additional space

### Override Append Prepend

Appending and Prepending (Override Style Syntax)

You can also append and prepend a variable's value using an override style syntax.

When you use this syntax, no spaces are inserted.

A = "hello"
A_append = " world"

B = "test"
B_append = "world"

C = "full"
C_prepend = "house"

### Override Removal

Removal

You can remove values from lists using the removal override style syntax

Specifying a value for removal causes all occurrences of that value to be removed from the variable.

FOO = "123 456 789 123456 123 456 123 456"
FOO_remove = "123"

$ bitbake -e | grep ^FOO=

### Override Vs Append Advantages

Override Style Operation Advantages

An advantage of the override style operations "_append", "_prepend", and "_remove" as compared to the "+=" and "=+" operators is that the override style operators provide guaranteed operations

IMAGE_INSTALL += "usbutils"

IMAGE_INSTALL_append = " usbutils"

---

## Create Layer

### Layer Fundamentals

What is a layer?

A layer is a logical collection of related recipes.

Types of Layers: oe-core, BSP Layer, application layer

Layer name starts with meta-, but this is not a technical restriction.

Eg. meta-mycustom

Why create a meta layer?

Despite most of the customization can be done with the local.conf configuration file, it is not possible to:

	Store recipes for your own software projects

	Create your own images

	Consolidate patches/modifications to other people’s recipes

	Add a new custom kernel

	Add a new machine


Most important point: Do not edit POKY/UPSTREAM Layers, as it complicates future updates
Advantage: This allows you to easily port from one version of Poky to another

### Layer Content And Priority

Depending on the type of layer, add the content:

	If the layer is adding support for a machine, add the machine configuration in conf/machine/

	If the layer is adding distro policy, add the distro configuration in conf/distro/

	If the layer introduces new recipes, put the recipes you need in recipes-* subdirectories of the layer directory.

Recipe directories inside layers

By convention, recipes are splitted into categories

The most difficult part is deciding in which category your recipe will go

By checking what was already done in the official layers should give you a good idea of what you should do

Layer Priority:

	Each layer has a priority, which is used by bitbake to decide which layer takes precedence if there are recipe files with the same name in multiple layers

	A higher numeric value represents a higher priority.

### Manual Layer Creation

Creating Layer

There are two ways to create your own layer.
1. Manually
2. Using script

Manually:

Step 1 : Create a directory for the layer. For example: 'meta-mylayer'

Step 2 : Create a conf/layer.conf
		You can simply copy meta-oe's one and just change "openembedded-layer" to something appropriate for your layer; you may also want to set the priority as appropriate.

Step 3 : Update bblayers.conf file with the new layer

### Bitbake Layers Tool

Creating layer using tool

You can create your own layer using the bitbake-layers create-layer command

$ bitbake-layers create-layer --help

The tool automates layer creation by setting up a subdirectory with a layer.conf configuration file, a recipes-example subdirectory that contains an example.bb recipe, a licensing file, and a README

$ bitbake-layers create-layer ../source/meta-mylayer

Default priority of the layer is 6

$ bitbake-layers add-layer ../source/meta-mylayer

$ bitbake-layers show-layers

### Layer Conf Reference

Layer Configuration File layer.conf

# The configuration and classes directory is appended to BBPATH
BBPATH .= ":${LAYERDIR}"

# The recipes for the layers are appended to BBFILES
BBFILES += "${LAYERDIR}/recipes-*/*/*.bb ${LAYERDIR}/recipes-*/*/*.bbappend"

# The BBFILE_COLLECTIONS variable is then appended with the layer name
BBFILE_COLLECTIONS += "skeleton"
BBFILE_PATTERN_skeleton = "^${LAYERDIR}/"
# The BBFILE_PRIORITY variable then assigns a priority to the layer. 
BBFILE_PRIORITY_skeleton = "1"

# This should only be incremented on significant changes that will
# cause compatibility issues with other layers
LAYERVERSION_skeleton = "1"

LAYERDEPENDS_skeleton = "core"

LAYERSERIES_COMPAT_skeleton = "zeus"

### Yocto Layer Compatibility

yocto-check-layer

The yocto-check-layer script provides you a way to assess how compatible your layer is with the Yocto Project

You should use this script if you are planning to apply for Yocto Project Compatible Program

$ source oe-init-build-env

$ yocto-check-layer your_layer_directory

---

## Challenge

Challenge

Find out in meta folder which one is used a lot: += or _append
