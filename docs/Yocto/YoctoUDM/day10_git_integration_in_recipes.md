# Day 10 — Git Integration in Recipes

## Git

### Git Remote Recipe Setup

Write a recipe for git remote repository

Yocto supports the ability to pull code from online git repositories as part of the build process.

Step 1: Set SRC_URI

SRC_URI = "git://<URL>;protocol=https"

Step 2: Set S environmental variable

S = ${WORKDIR}/git

Step 3: Set SRCREV environmental variable 

What is the use of SRCREV?

When fetching a repository, bitbake uses SRCREV variable to determine the specific revision from which to build

There are two options to this variable:

1) AUTOREV:

SRCREV = "${AUTOREV}"

When SRCREV is set to the value of this variable, it specifies to use the latest source revision in the repository

The build system accesses the network in an attempt to determine the latest version of software from the SCM

2) A specific revision (SHA1 hash when fetching from git)

If you want to build a fixed revision and you want to avoid performing a query on the remote repository every time BitBake parses your recipe

SRCREV = "d6918c8832793b4205ed3bfede78c2f915c23385"

### Specifying Git Branch

Specifying a different branch

You can specify the branch using the following form:

SRC_URI = "git://server.name/repository;branch=branchname"

If you do not specify a branch, BitBake looks in the default "master" branch.

BitBake will now validate the SRCREV value against the branch.

### Local Git Repository

Fetch from a local git source

git fetcher supports protocol parameter

protocol

The protocol used to fetch the files.

The default is "git" when a hostname is set. If a hostname is not set, the Git protocol is "file"

SRC_URI = "git:///home/user/git/myTest/;protocol=file"

### Private Repo Authentication

What about private repositories? 

Private repositories have the added complexity of requiring authentication before you can download (a.k.a clone) them. 

Luckily, Yocto supports the ssh protocol.

To make Yocto use a GitHub repository:

SRC_URI="git://git@github.com/group_name/repo_name.git;protocol=ssh"

To use SSH, you must generate an SSH keypair on your computer and add the public key to your GitHub account

### Git Tag Parameter

How to pass a tag to a git recipe

tag parameter of git fetcher

"tag": Specifies a tag to use for the checkout. To correctly resolve tags, BitBake must access the network

SRCREV is not needed in this case

### Patching Source Code

Patching the source for a recipe

Advantage of Yocto is that everything is build from source

It's fairly easy to make changes to anything that gets built

As part of building a recipe, OE creates a tmp/work/<architecture>/<recipe>/<version> directory, known as the "work directory". 

This is where all of the work done to build a recipe takes place.

A subdirectory contains the source of the recipe named <recipename>-<version> or "git" (depending on how the fetched source is provided)

The temptation is to simply make changes here and then recompile, but there are several reasons why that's not a good idea:

1.You can easily lose your changes if you're not careful e.g. running bitbake -c clean will wipe the directory out

2. You have to force compilation as the build system doesn't know that you've made any changes

bitbake -c compile <recipe> -f

### Creating Patches From Git

Patches

Patches can be easily created using Git

If you download the third-party source code as a Git repository, this is definitely the easiest solution

After downloading the repository, make the required changes to the code, and add these changes to the repository as a new commit

You can then tell Git to make a patch file.

If all the changes are contained within a single additional commit, you can use the following command:

~$ git show HEAD > my-patch.patch

These generated patches should be bundled with your recipe files.

Patches should always be in a sub-directory of where the recipe lives.

Yocto will automatically apply these patches when it needs to build your recipe.

---

## Challenge

Challenge

PV = "0.1+git${SRCPV}"
