nos
===

An experimental and still-in-development implementation of a network operating system for the ATARI 8-bit computer and FujiNet device that accesses file system resources for up to eight remote mount points via the N: device. There is no support provided for traditional physical or emulated magnetic floppy drives (D: devices).

The operating system is well-suited for workflows where integration between a modern computer and an ATARI computer would be convenient.

After mounting a folder under, say, a TNFS server, files can be listed, renamed, deleted, etc using the command processor supplied by the operating system.

At least two critical features are still being developed:

1. A `COPY` file utility
2. Integration of the latest n-handler code

Other features on the wish-list:

1. A `TYPE` command to view plain text files
2. A `MOVE` command to relocate files between directories
3. A `SOURCE` command to execute commands stored within a text file
4. A `HELP` command
3. Wildcard support for `DEL`, `COPY`, `MOVE`
4. Date/time stamps in the listings produced by `DIR`

Command Processor
====================

FujiNet DOS provides a command-line interface. The prompt shows the current default network drive. FujiNet DOS commands are typed at this prompt.

Prompt:

	Nn:[] <-- cursor

where `n` is the current default device number (1-4).

The ATARI's full-screen editor can be used to correct or cobble a command using text snippets found elsewhere on the screen.

Command Summary
===============

|Command  |Purpose|
|:-------:|:------|
|NCD      |Mount or unmount a remote directory|32400
|NPWD     |Display the current mount point for a network drive|
|Nn:      |Change the current default network drive|
|DIR      |Display a list of files/directories found on a mounted network drive|
|DEL      |Delete a file found on a mount point|
|RENAME   |Rename a file or directory on a mount point|
|MKDIR    |Create a directory on a mount point|
|RMDIR    |Delete an empty directory on a mount point|
|RUN      |Execute machine code at a specified memory address|
|CAR      |Switch to a cartridge (if installed)|
|LOAD     |Load/Execute binary file|
|REENTER  |Attempt to jump to last LOADed program|
|NTRANS   |Translate the end-of-line character between the ATARI and the remote host's operating system|
|SOURCE   |Batch execute NOS/DOS commands in text file|
|PRINT    |Display text messages in batch file|
|REM      |Comment within batch file|
|@NOSCREEN|Disable echo of commands in batch file|
|@SCREEN  |Enable echo of commands in batch file|
|TYPE     |Show contents of text file|
|CLS      |Clear/Erase the screen|
|COLD     |Execute coldstart routine COLDSV ($E477)|
|WARM     |Execute warmstart routine WARMSV ($E474)|
|HELP     |Online help system|

Commands in Detail
==================

`NCD`
====

Mount or unmount a remote directory to a network device (N1: through N4:). The remote directory must be hosted using one of the supported FujiNet protocols (primarily TNFS).

**Mounting**

Usage:

	NCD [N[n]:]PROTO://path[/][:port]

Where `n` is an optional device number 1-4. If `n` is omitted, the current default device number is implied.
Where `PROTO` is a network protocol supported by the FujiNet, such as `TNFS`, `FTP`, `SMB`. File operations for various protocols are limited by what is supported by the protocol or FujiNet.

General examples:

	NCD TNFS://192.168.1.100/
	NCD N1:TNFS://192.168.1.100
	NCD N1:TNFS://192.168.1.100/action/myproj/
	NCD N2:FTP://ftp.pigwa.net/stuff/collections/
	NCD "N2:FTP://ftp.pigwa.net/stuff/collections/holmes cd/"

A required trailing path separator is appended if none is provided.

Example:

	NCD N1:TNFS://192.168.1.100/action/myproj

resolves to:

	NCD N1:TNFS://192.168.1.100/action/myproj/

After establishing a mount point using a protocol and absolute path, a subsequent relative path can be supplied.

Example:

	NCD N1:TNFS://192.168.1.100/
	NCD action/myproj

results in the path:

	N1:TNFS://192.168.1.100/action/myproj/

There is limited support for supplying a relative path that includes a reference to a parent directory.

Example:

	NCD N1:TNFS://192.168.1.100/action/myproj01/
	NCD ../myproj02/

results in the path:

	N1:TNFS://192.168.1.100/action/myproj01/../myproj02/

**Unmounting**

To release a remote mount point, enter the `NCD` command with only a device and no protocol and path.

Usage:

	NCD N[n]:

Where `n` is an optional device number 1-4. If `n` is omitted, the current default device number is implied.

Example:

	NCD N2:


`NPWD`
---
Query the current mount point for a network drive from the FujiNet device and display the results.

Usage:

	NPWD [N[n]:]

Where `Nn:` is optional and `n` detotes a device number 1-4. If `n` is omitted, the current default device number is implied.

General Examples:

	NPWD
	N1:TNFS://192.168.1.100/action/myproj01/   <-- FujiNet Response
	
	NPWD N2:
	N2:FTP://ftp.pigwa.net/stuff/collections/  <-- FujiNet Response


If there is nothing mounted to on the N: device then the response will be blank.

`Nn:` `(Change Drive)`
---
Change the current default network drive.

Usage:

	Nn:

Where `n` is required and `n` denotes a device number 1-4.

After changing the default network drive, the command prompt is updated to reflect the new default network drive. 

Note: No check is made regarding the status of the mount point for the new default network drive. That is, it is possible to switch to a network drive that has no assigned mount point.

`DIR`
---
Display a list of files/directories found on a mounted network drive. Filename and file size are returned.

Usage:

	DIR [N[n]:][path/][pattern]

Where `Nn:` is optional and `n` detotes a device number 1-4. If `n` is omitted, the current default device number is implied.

Where `[path]` is a relative path from the current working directory.

Where `[pattern]` is optional and can include a pattern string including wildcard characters [`*`]. 

Note: Regardless of the pattern string, directory entries are always returned.

Examples:

	DIR
	DIR *.BAS
	DIR N:
	DIR N2:
	DIR N2:*.BAS
	DIR N2:action/HELLO*.ACT
	DIR "N2:holmes cd/"

`DEL`
---

Delete a file found on a mount point.

Usage:

	DEL|ERASE [N[n]]:[path/]file

Only one file can be deleted per command. Pattern-matching using wildcard characters is not currently supported. 

Filenames are case-sensitive. If a filename contains spaces, enclose the entire path and filename with double-quotes.

    DEL MYFILE.AWP
    DEL N2:MyFile.AWP
    DEL N2:DOCS/MYFILE.AWP
    DEL "N2:DOCS/My File.AWP"
    ERASE "N2:DOCS/My File.AWP"

`RENAME`
---
Rename a file or directory on a mount point.

Usage:

	RENAME [N[n]:]OLDNAME,NEWNAME

Examples:

    RENAME ATARIWRITER.XEX,ATARIWRITER.COM
    RENAME AtariWriter.xex,ATARIWRITER.COM
    RENAME N2:AtariWriter.xex,ATARIWRITER.COM
    RENAME "Atari Writer.xex,ATARIWRITER.COM"
    RENAME "N2:APPS/Atari Writer.exe,ATARIWRITER.COM"


`MKDIR`
---
Create a new directory on a mount point.

Usage:

	MKDIR [N[n]:][/][path/]dirname

Where dirname is the name of the directory to be created.

Where path is an optional parent directory to dirname. 

`RMDIR`
---
Remove an empty directory on a mount point.

Usage:

	RMDIR [N[n]:]path

`CAR`
---
Switch to a cartridge (if installed).

Usage:

	CAR

Control is yielded to the warmstart vector defined within the cartridge. This should allow switching between the cartridge and DOS without destroying the cartridge's working memory. For example, if you are typing a BASIC program, you can switch between DOS and BASIC without losing work.

`LOAD`
---
Load a binary file into memory. Machine code vectored from INITAD ($02E2) or RUNAD ($02E0) is executed as appropriate.

	LOAD JUMPMANJR.XEX
	LOAD N2:ATARIWRITER.COM
	LOAD "N2:Atari Writer.xex"

`RUN`
---

Begin executing machine code at a specified memory address.

	RUN hexaddr

Where `hexaddr` is a four-character hexadecimal address. Leading the address with a `$` returns an error.

Example:
	
	RUN A000
	RUN 0600

`REENTER`
---

Attempt to re-enter a memory-resident program by jumping to the address loitering in `RUNAD` ($02E0). 

Usage:

    REENTER

For example, if an application has a `QUIT TO DOS` function, it may be possible to exit, perform a task in `DOS`, and then jump back into the application without having to re-load the application from the network.

The success of the `REENTER` command depends on the application. Also, be aware that the entry point for an application may clear any previous work in memory, so save any work before quitting to DOS.

`NTRANS`
---
Translate the end-of-line character between the remote host's operating system and the ATARI computer. This is useful for exchanging text files to or from a host computer.

Usage:

	NTRANS [N[n]:] mode

Where mode is required and either 0, 1, 2, or 3

| n  |Translation Mode|
|:--:|--------|
|0|No translation of ATARI EOL ($9B)|
|1|Convert CR --> ATARI EOL ($9B) (Incoming-only)|
|2|Convert LF --> ATARI EOL ($9B) (Incoming-only)|
|3|Convert CR/LF <--> ATARI EOL ($9B) (Both directions)|

The translation mode is assigned on the current default network drive.

Best results seem to arrive from using mode 3 and a text editor on the host machine that supports CR/LF.

Having translation enabled when working with binary files may result in corruption.

`CLS`
---
Clear / erase the screen.

Usage:

	CLS

`COLD`
---

Reboots the computer. Can be used to disable/enable ATARI BASIC on non-400/800 computers by holding (or not) the `OPTION` console key at the moment the command is launched.

Usage:

    COLD

`WARM`
---

Usage:

    WARM

Performs a warmstart.

`HELP`
---

Usage:

    HELP [TOPIC][/SUBTOPIC]

Online help system.
