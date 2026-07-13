# Day 09 — Build Logs and Advanced Recipes

## Logs

### Build Log Location

Where do I find build logs?

Every build produces lots of log output for diagnostics and error chasing

Output of bitbake gets logged to tmp/log/cooker/<machine>/

$  cat tmp/log/cooker/<machine>/<timestamp>.log  | grep 'NOTE:.*task.*Started'

### Per Recipe Temp And Task Logs

Where do I find build logs?

For each individual recipe, there is a "temp" directory under the work directory

Within the build system this directory is pointed to by the T variable, so if you need to you can find it by using bitbake -e

$ bitbake -e <recipename> | grep ^T=

Each task that runs for a recipe produces "log" and "run" files in ${WORKDIR}/temp

You can find log files for each task in the recipe's temp directory

Log files are named log.taskname (e.g. log.do_configure, log.do_fetch, and log.do_compile)

For convenience, symbolic links are kept updated by BitBake, pointing to the last log files using the pattern log.<task>

We can run the scripts for every task with the pattern run.<task>.<pid> 

These files contain the commands which produce the build results

### Bitbake Logging Functions

Logging Information during task execution

The logging utilities provided by BitBake are very useful to trace the code execution path.

BitBake provides logging functions for use in Python and Shell Script code, as described

Python: For use within Python functions, BitBake supports several log levels, which are bb.fatal, bb.error, bb.warn, bb.note, bb.plain, and bb.debug

Shell Script: For use in Shell Script functions, the same set of log levels exists and is accessed with a similar syntax: bbfatal, bberror, bbwarn, bbnote, bbplain, and bbdebug

There is one subtle difference between the use of the logging functions in Python and Shell Scripting.

The logging functions in Python are directly handled by BitBake, seen on the console, and stored in the execution log that can be seen inside build/tmp/log/cooker/<machine>

When the logging functions are used in Shell Script, the information is outputted on the task's respective task log file, which is available in build/tmp/work/<arch>/<recipe name>/<software version>/temp

### Logging Levels

#### Logging Levels Reference

Logging

bb.fatal and bbfatal: These have the highest priority of logging messages as they print the message and terminate the processing. They cause the build to be interrupted.

bb.error and bberror: They are used to display an error but do not force the build to stop.

bb.warn and bbwarn: They are used to warn users about something.

bb.note and bbnote: These add a note to the user. They are only informative.

bb.plain and bbplain: These output a message.

bb.debug and bbdebug: These add debugging information that is shown depending on the debug level used.

#### Shell Logging Example

do_compile() {
 bbplain "*************************************"
 bbplain "* *"
 bbplain "* Example recipe created by bitbake-layers *"
 bbplain "* *"
 bbplain "*************************************"
}

### Debug Output Ddd Flag

Debug Output

You can see the debug output from Bitbake by using the -D option

The debug output gives more information about what BitBake is doing and the reason behind it

Each -D option you use increases the logging level.

The most common usage is -DDD

---

## Recipes

### C Program With Function

**`function.c`**

```c
#include <stdio.h>

int function(char *input)
{
	return printf("%s\n", input);
}
```

**`userprog.c`**

```c
#include <stdio.h>

int function(char *);

int main()
{
	function("Hello World\n");
	return 0;
}
```

### Simple C Program

**`userprog.c`**

```c
#include <stdio.h>

int main()
{
	#ifdef USE_SYSCALL
		write(1, "Hello System Call", 17);
	#else
		printf("Hello World");
	#endif
	return 0;
}
```

### C Program With Makefile

**`Makefile`**

```makefile
# compiler flags:
#  -g    adds debugging information to the executable file
#  -Wall turns on most, but not all, compiler warnings
CFLAGS  = -g -Wall -DUSE_SYSCALL

# the name to use for both the target source file, and the output file:
TARGET = userprog

all: $(TARGET)

$(TARGET): $(TARGET).c
	${CC} $(CFLAGS) -o $(TARGET) $(LDFLAGS)  $(TARGET).c

clean:
	rm  -f $(TARGET)
```

**`userprog.c`**

```c
#include <stdio.h>

int main()
{
	#ifdef USE_SYSCALL
		write(1, "Hello System Call", 17);
	#else
		printf("Hello World");
	#endif
	return 0;
}
```

### Oe Runmake Helper

oe_runmake

The default behavior of do_compile task is to run the oe_runmake function if a makefile(Makefile, makefile or GNUMakefile) is found. If no such file is found, the do_compile tasks does nothing

The default behavior of do_configure task is to run oe_runmake clean if a makefile (Makefile, makefile, or GNUmakefile) is found

oe_runmake vs make

oe_runmake function is used to run make.

oe_runmake
	passes EXTRA_OEMAKE settings to make

	displays the make command

	checks for errors generated via the call.

In OE environment you should not call make directly rather use oe_runmake when you need to run make.

oe_runmake is one of many helper functions defined by the base class

Try: check the definition of oe_runmake in base.bbclass

### Extra Oemake Variable

EXTRA_OEMAKE

If you need additional make options, you should store them in the EXTRA_OEMAKE variable

**`Makefile`**

```makefile
# compiler flags:
#  -g    adds debugging information to the executable file
#  -Wall turns on most, but not all, compiler warnings
CFLAGS  = -g -Wall -DUSE_SYSCALL

# the name to use for both the target source file, and the output file:
TARGET = userprog

all: $(TARGET)

$(TARGET): $(TARGET).c
	${CC} $(CFLAGS) -o $(TARGET)  $(TARGET).c $(LDFLAGS)

install:
	install -d $(DESTDIR)
	install -m 0755 $(TARGET) $(DESTDIR)

clean:
	rm  -f $(TARGET)

uninstall:
	rm $(DESTDIR)$(TARGET)
```

**`userprog.c`**

```c
#include <stdio.h>

int main()
{
	#ifdef USE_SYSCALL
		write(1, "Hello System Call", 17);
	#else
		printf("Hello World");
	#endif
	return 0;
}
```

### Makefile Without Clean Target

#### Question

What if i have a Makefile which don't have clean target?

#### Answer

What if i have a Makefile which don't have clean target?

CLEANBROKEN = "1"

If set to "1" within a recipe, CLEANBROKEN specifies that the make clean command does not work for the software being built.

Consequently, the OpenEmbedded build system will not try to run make clean during the do_configure task, which is the default behavior.

**`Makefile`**

```makefile
# compiler flags:
#  -g    adds debugging information to the executable file
#  -Wall turns on most, but not all, compiler warnings
CFLAGS  = -g -Wall -DUSE_SYSCALL

# the name to use for both the target source file, and the output file:
TARGET = userprog

all: $(TARGET)

$(TARGET): $(TARGET).c
	${CC} $(CFLAGS) -o $(TARGET)  $(TARGET).c $(LDFLAGS)

install:
	install -d $(DESTDIR)
	install -m 0755 $(TARGET) $(DESTDIR)

uninstall:
	rm $(DESTDIR)$(TARGET)
```

---

## Challenge

Challenge

Go to meta folder and analyze the various recipes related to git
