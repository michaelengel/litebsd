LiteBSD
=======

This is my private copy of LiteBSD with some fixes for problems I encountered.
So far, everything is tested on qemu-system-mipsel and the WIFIRE PIC32MZ board.

As of now, the bugfixes are:

1. support for KTRACE
   - when enabling KTRACE in the kernel config, trying to trace a process
     results in a kernel panic due to trying to release a lock in 
     sys/kern_lock.c (l. 352). The current lock holder is set to LK_KERNPROC
     (pid -2), which - according to sys/lock.h - indicates that no process 
     holds an exclusive lock.

2. fix the execution of #! executables
   - The code implementing the execve system call is different to either the
     4.3 or 4.4BSD implementations (sys/kern_exec.c) I could find online.
     The existence of the "shebang" (#!) magic is tested in the getheader 
     function. When getheader detects this, it tries to set the real executable 
     to the one given after #!; however, this is not returned to execve from
     getheader (other BSD implementations implement the getheader functionality
     inside the execve function).

     This is important since, if #! was detected, execve jumps (yep, goto...)
     to its own start and re-executes with the new executable name (to be exact,
     a new struct nameidata) - which was, however, never set. I changed getheader 
     to return a pointer to the new nameidata for the real executable and moved
     the again label one line down to avoid re-initializing the nameidata from 
     scratch. 

     When doing this, the kernel crashed due to a double free, probably due to 
     the call to "FREE(ndp->ni_cnd.cn_pnbuf, M_NAMEI);" in step 2 in execve. 
     This is currently commented out.
     
     TODO: I will have to investigate if this is the real fix for the double free
     problem or if this causes a memory leak...


Enjoy,
    Michael Engel (engel@multicores.org)

----

Here's the original README:

LiteBSD is a variant of 4.4BSD operating system adapted for microcontrollers.
Currently, only the Microchip PIC32MZ family is supported as a target.
It is equipped with MMU with paging support, and 512kbytes of on-chip RAM.
These resources are enough to build a compact networked embedded system.

For more information, see [LiteBSD Wiki pages](https://github.com/sergev/LiteBSD/wiki).

Precompiled binaries for several PIC32MZ boards are available from
[Autobuild Download page](http://litebsd.org/wiki/autobuild.php).

If you are looking for the LiteBSD ports tree please see
[this repository](https://github.com/ibara/LiteBSD-Ports).
