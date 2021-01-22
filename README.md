# HI-TECH Z80 C Compiler for CP/M

## Introduction

In this repository I intend to supply updates and enhancements to
the HI-TECH C Compiler for Z80 v3.09 running natively under CP/M or under
emulation using RunZ80, SIMH or ZXCC.

Each release is a consolidated milestone with various updates and
patches applied.

The latest release is V3.09-8 (see Modification History below).

If you only wish to download the latest binary distribution, download
it from

https://raw.githubusercontent.com/agn453/HI-TECH-Z80-C/master/htc-bin.lbr

I welcome bug reports/fixes and additional commentry and discussion.

You can raise an issue here on GitHub or contact me directly via e-mail
at <agn453@gmail.com> or by posting to the USENET newsgroup
forum at comp.os.cpm in the "Consolidating Updates for HI-TECH C for
Z80 CP/M v3.09" discussion thread - (accessible via Google Groups at
https://groups.google.com/forum/#!topic/comp.os.cpm/V9Qwoc3--Ak ).

The first release is a slightly modified (see below) v3.09.

## Background

The HI-TECH Z80 CP/M C compiler v3.09 is provided free of charge for any
use, private or commercial, strictly as-is. No warranty or product
support is offered or implied.

You may use this software for whatever you like, providing you acknowledge
that the copyright to this software remains with HI-TECH Software.

The *dist* folder contains the entire compiler including the original
distributed library source code in Huffman-encoded archive files (CPM.HUF,
GEN.HUF, FLOAT.HUF and STDIO.HUF). NB: I have not rebuilt the .HUF files -
so please use updated run-time library source files in the following folders.

The *cpm*, *gen*, *float* and *stdio* folders contain the library
source code that has been extracted from the .HUF archive files
using DEHUFF.COM and the updated/modified files.

The *doc* folder contains the documentation in ASCII text as HTCZ80.TXT.


## Modification History

In chronological order - so be sure to check the latest updates at the
end of this README file.

### Release v3.09

The original HI-TECH C linker (LINK.COM) has been renamed as LINQ.COM
and the main compiler front-end (C.COM) has been patched to reflect
this name change.  This was done to avoid a name clash with the Digital
Research supplied CP/M linker.  The original version is kept as C-ORIG.COM
and the patched version is dist/C309.COM

I've edited the documentation to remove the underlining and bolding of
text by over-printing and fixed some layout issues and typos.


### Release v3.09-1

As supplied in v3.09, the LIBF.LIB produces inaccurate floating point
trigonometry results - as reported by "Ed" in his posting to the
comp.os.cpm USENET Newsgroup.

```
Since we're on the topic of Hitech-C for CP/M, below is a problem I
discovered with the freeware release.

LIBF.LIB supplied with the freeware Hitech-C v3.09 produces inaccurate trig
results.  I found that by simply recompiling the library with the existing
source files from FLOAT.HUF, it fixed the problem.  Absolutely no changes
to the sources were required.
```

The re-compiled LIBF.LIB from the FLOAT.HUF library sources in *float*
has been copied to the distribution files in *dist*.  A CP/M submit file to
rebuild LIBF.LIB has been added to *float*.

A *test* folder has been created.  In it is a TESTTRIG.C program for testing
this release.


### Release v3.09-2

The late Jon Saxton (former SYSOP of the Australian Tesseract RCPM)
collected together a number of patches and updates - See Tesseract
volume 091.

These include contributions from John Elliot via
http://www.seasip.info/Cpm/software/index.html - in particular
for PIPEMGR which is a CP/M 3 RSX that implements piping and redirection.

John's contributions will be implemented in a later release.

This release includes updates to memset(), bios() and bdos() library
routines.  In addition the stat(), close() and sys_err() routines
are updated to remove references to bdoshl() and to include CP/M 3
error messages.

```
memset()

     The Hi-Tech C implementation of memset() is seriously broken and is
     almost guaranteed to give trouble in any program which uses it.

     The implementation simply does not agree with the function declaration.

bios()

    The original bios() routine used the function number to construct an offset
    into the BIOS jump vector and from that calculated the address of the
    appropriate BDOS routine.  Except for some character I/O routines, that
    method of calling the BIOS is guaranteed to crash CP/M 3.

    A new bios() function has been implemented which checks for CP/M 3 and if
    that is the current operating system then it accesses the BIOS via the
    sanctioned method, i.e. by invoking bdos(50) with an appropriately filled-
    out parameter block.

    The standard bios() routine takes 1, 2 or 3 parameters:

        short bios(int func, int bc, int de)

    and that is good enough for CP/M 2.2.  However CP/M 3 has functions which
    also take inputs in A and/or HL.  The routine is smart enough to figure
    out where to put the parameters.  All the C programmer needs to do is to
    supply the requisite arguments in the right order.

    A bug in the 2.2 bios() routine has been fixed.  The original code would
    always return an 8-bit result even though there are some functions which
    return 16-bit results.

bdos()

    All versions of CP/M 80 return from BDOS calls with BA = HL regardless of
    whether the function is returning an 8-bit or 16-bit result.  Hi-Tech C
    provided a bdos() call for 8-bit results and a bdoshl() call for 16-bit
    results.  John Elliott added a third, bdose(), for disk functions which
    could return extended error codes in H (and B) when running on CP/M 3.

    There is now a single bdos() function.  It always returns a 16-bit result
    in HL.  The bdoshl() routine still exists for compatibility with older
    source files but it is simply an alias for bdos().  John Elliott's
    bdose() routine no longer exists; bdos() also performs its functions.

    Some BDOS functions return 255 as an error flag.  The old bdos() code
    would sign-extend that to -1 (16 bit) but that is no longer done.

        if (bdos(fn, data) == -1)               /* This won't work */

        if (bdos(fn, data) == 255)              /* This works sometimes */

        if ((bdos(fn, data) & 0xFF) == 255)     /* This always works */

    Under CP/M 3 several functions return extended error information in
    the upper 8 bits of the return value.  (Those functions also set the
    standard errno global item.)  Always use the third form when checking
    for an error result.  The compiler is quite clever and doesn't make your
    program work harder ... it just uses the L register.
```


### Release v3.09-3

These are the Jon Saxton modified versions of John Elliots's patches.

```
Wildcard expansion

    Programs compiled with this update get wildcard expansion of the CP/M
    command line automatically.  There is no longer any need to call
    _getargs() explicitly.

    Enclosing an argument in quote marks (' or ") supresses expansion.
    This can be useful for programs like grep which may use ? and/or * in
    text search patterns or for program options containing a question mark:

        grep 'a.*end' *.h 2d:*.c
        grep "-?"

    The -R option passed to the Hi-Tech C compiler is no longer useful.  (It
    didn't work anyway.)

    Implementing automatic argument expansion meant altering the order of
    modules in LIBC.LIB.  That entailed rebuilding the entire library from
    scratch.  A script to do that is supplied.

Filename drive and user number prefixes

    The format of file name prefixes indicating drive letter and/or user
    number is now much more liberal.  If a file "sample.txt" is on drive
    E: in user area 12 then depending on the current drive/user the file
    may be accessed as:

        sample.txt              if current DU: is E12:
        12:sample.txt           if current disk is E:
        e:sample.txt            if current user is 12
        e12:sample.txt
        12e:sample.txt
        12:e:sample.txt         (Hi-Tech C format)

    Note that any of these forms is acceptable for program arguments, even
    those containng wildcard characters (?, *).

Exact file sizes (CP/M 3 and DOS+ v2.5)

    Exact file sizes are supported on CP/M 3.  Under CP/M 2.2 file sizes are
    always a multiple of 128 bytes but some versions of CP/M allow setting
    and reading a "last sector byte count" field in the FCB.  The concept was
    carried forward from CP/M's ancestor, ISIS, but DRI did not document the
    precise meaning of the count for CP/M so two distinct interpretations were
    possible:

    a.  LSBC represents the number of UNUSED bytes in the last sector;

    b.  LSBC contains the number of USED bytes but 0 means 128.

    Both usages have appeared in the small number of CP/M 3 utilities which
    support exact file lengths.  My own tools use the first interpretation;
    John Elliott's utilities use the second.

    The first interpretation is slightly simpler to implement because it
    avoids any special-case handling.  It is also historically correct.
    Neither of these considerations is particularly significant but we do need
    to have programs which work together and interpret the last sector byte
    count the same way.  After a lifetime in software development I have
    learned to avoid handling special cases as far as possible.  Accordingly
    I have modified John Elliott's routines to interpret the last sector byte
    count as the number of unused bytes in the last sector.

    With this interpretation the formula for calculating the number of bytes
    in a file is always (sectors * 128) - lsbc.

    (When running on CP/M 2 lsbc is always zero.)

strcasecmp()

    The case-insensitive string comparison function strcasecmp() has been
    implemented.  Its function prototype is in string.h.  It works just like
    strcmp() except that upper- and lower-case letters are treated as 
    identical.  There is also a strncasecmp() analogue of strncmp() which
    allows one to limit the comparison to a certain number of characters.

toupper() and tolower()

    The functions toupper() and tolower() were implemented as macros which
    added a case-shift value to the character parameter without checking to
    see if the parameter was a letter.  To use these routines correctly it was
    necessary to do a range check, e.g.

        if (isupper(c))
            c = tolower(c);

    These operations now invoke the correspondingly named routines in LIBC.LIB
    and it is no longer necessary to pre-test the character.

Support for read/write files

    The stdio functions now allow files to be opened in modes "r+" and "w+" - 
    ie, reading and writing are supported on the same file.
    Remember to fflush() or fseek() between a read and a write.
    This code is experimental.

PIPEMGR support (CP/M 3)

    Programs automatically check for PIPEMGR when they load, and if it
    is present the C streams stdin, stdout and stderr are routed to their
    PIPEMGR counterparts - no modification of the source is required.

    The special device files (CON: LST: PUN: and RDR:) are joined by RSX: 
    (PIPEMGR stdin/stdout streams) and ERR: (PIPEMGR stderr stream). If
    PIPEMGR is not present, these files behave like CON:

    The variable:

        extern char _piped;

    is nonzero if PIPEMGR is present.

    The command 

        FOO <CON:

    will behave differently if PIPEMGR is present. If it is not present,
    then you get line-based input from the Hi-Tech C library. If PIPEMGR
    is present then you get character-based input from PIPEMGR.

    The line-based input in the library will now return EOF if a control-Z
    is typed on a line by itself. Previously it would never return EOF on 
    line-based input.

    Unfortunately if you're using Simeon Cran's replacement ZPM3 then it
    does not allow control-Z to be typed on an input line.
    I'm not yet sure how to get round this.
  
    Remember that when PIPEMGR has parsed the command line, you may have a 
    number of blank (zero-length) arguments.

CP/M 3 extended error support

    To enable CP/M 3 extended error support, put the line

        bdos(0x2D,0xFF);

    near the beginning of your program. Then errors such as read-only files 
    will not cause the program to terminate abruptly; instead the call which
    caused the error will fail, and perror() will explain the reason. The 
    variable:

        extern int errno;

    will contain (16+x) for CP/M 3 extended error x.

File passwords (CP/M 3)

    NB: This feature is disabled.  You will need to change the
        PWDREC definition in the file BDOS.AS to true and rebuild
        and replace the module in LIBC.LIB to enable it.

    There is partial support for file passwords. To enable it, declare a 
    function:

        #include <cpm.h>

        char *mypass(struct FCB *fcb)
        {
        /* For the file specified by "fcb", a password needs to be entered.
           Return a pointer to it (8 characters, uppercase, packed with
           spaces) 
         */
        }

    and insert a line near the beginning of your program:

        _passwd = mypass;

    Then your routine ("mypass" in this example) will be called when
    CP/M reports a password error; it will be called repeatedly until
    the user enters the correct password or a it returns a NULL pointer
    (ie, the user has given up).

Checks for a ZCPR3 environment

    The variable

        extern void *_z3env;

    is 0 if one is not present, or its address if one is. It also passes it
    as a third parameter to main():

        int main(int argc, char **argv, void *z3env);

    The Z3 environment address must be passed to the program in HL.

Graceful exits

    Compiled programs exit gracefully if run on an 8080 or 8086 processor,
    or if there is not enough memory for the program and its static data.
    This is done in the CRTCPM.OBJ module.

CP/M 3 compatible error system

    exit(0) and _exit(0) set the CP/M 3 error code to 0.
    exit(n) and _exit(n) for non-zero n set the error code to 0xFF00 | (n&0x7F).

    The practical upshot is that exit(0) translates as "OK"; other values 
    translate as "error". You can use this in conjunction with CP/M 3's
    SUBMIT features.  If the next command in a .SUB file is preceded by
    a : character, it will be ignored:

        CPROG 
        :OTHER

    will only execute OTHER if CPROG exited with the value 0.

Extended getenv()

    Under CP/M 3, getenv() uses the search path set by SETDEF to locate the
    ENVIRON file. So if you like to keep ENVIRON on a drive that isn't A:, 
    programs will still find it.
```


### Release v3.09-3b

Updated the previous release to adjust the ordering of modules in LIBC.LIB -
programs should now link correctly.  Also added the missing documentation for
the debugger in doc/DEBUGMAN.TXT (this was previously available from the
now defunct hitech.com.au web site) and scanned PDFs of the 1989
vintage documentation.


### Release v3.09-4

Some of the string related function prototypes in STRING.H are missing from
the LIBC.LIB library.  Most of these are from an update to the MSX flavour
of HI-TECH C by Arnold Metsalaar.  I've cherry-picked the following routines
which are applicable to generic CP/M as well as MSX.

```
STRING.H
    The header file has been changed to reflect the available
    functions in LIBC.LIB.  There are still missing routines -
    namely strcoll() strcspn() strpbrk() and strspn() and these
    have been commented out for now.

strchr() and strrchr()
        char *strchr(char *s, int c)
        char *strrchr(char *s, int c)
    These functions - as well as index() and rindex() (which are identical)
    previously returned a NULL for no match.  The functions now return
    a pointer to the character string's ending NUL character.

stricmp() and strnicmp()
        char stricmp(char *s1, char *s2)
        char strnicmp(char *s1, char *s2, size_t n)
    Case-insensitive versions of strcmp() and strncmp() comparison routines.
    Can also be referenced as strcasecmp() and strncasecmp().

strstr(), strnstr(), stristr() and strnistr()
        char *strstr(char *t, char *s)
        char *strnstr(char *t, char *s, unsigned int n)
        char *strcasestr(char *t, char *s)
        char *strncasestr(char *t, char *s, unsigned int n)
    These extra functions locate the first occurrence of string s in
    string t.  The functions strnstr() and strcasestr() read at most
    n characters from the string t.  The functions strcasestr() and
    strncasestr() use case insensitive comparisons.
    All these functions return a pointer to the first character of
    the first occurence of string s in string t if found, and NULL
    otherwise.

strdup()
        char *strdup(char *s)
    Allocates a new buffer for and copies the string pointed to
    by s to it.  Returns a pointer to the copy of the string or NULL
    if the memory allocation failed. The memory block can be released
    using free().

strtok()
        char *strtok(char *s, char *tok, size_t toklen, char *brk)
    Copies characters from s to tok until it encounters one of the
    characters in brk or until toklen-1 characters have been copied
    (whichever comes first).  It then adds a NUL character to the
    end of the string.  This is a non-conforming POSIX function.

TIME.H
    Now includes a prototype for strftime() - see below.

strftime()
        size_t strftime(char *s, size_t maxs, char *f, struct tm *t)
    Converts a time value t to a string using the format string f
    into the string s of size maxs (including a terminating NUL).
    It acts as a sprintf() function for date/time values. The
    following are valid in the format string -

             %A      full weekday name (Monday)
             %a      abbreviated weekday name (Mon)
             %B      full month name (January)
             %b      abbreviated month name (Jan)
             %c      standard date and time representation
             %d      day-of-month (01-31)
             %H      hour (24 hour clock) (00-23)
             %I      hour (12 hour clock) (01-12)
             %j      day-of-year (001-366)
             %M      minute (00-59)
             %m      month (01-12)
             %p      local equivalent of AM or PM
             %S      second (00-59)
             %U      week-of-year, first day sunday (00-53)
             %W      week-of-year, first day monday (00-53)
             %w      weekday (0-6, sunday is 0)
             %X      standard time representation
             %x      standard date representation
             %Y      year with century
             %y      year without century (00-99)
             %Z      timezone name
             %%      percent sign

    the standard date string is equivalent to:

        %a %b %d %Y

    the standard time string is equivalent to:

        %H:%M:%S

    the standard date and time string is equivalent to:

        %a %b %d %H:%M:%S %Y

    strftime() returns the number of characters placed in the
    buffer, not including the terminating NUL, or zero if more
    than maxs characters were produced.
```

### Release v3.09-4-Z280

This is the first experimental release of the libraries that have been
built for a Zilog Z280 MPU.  Most of this work was retrieved from a
distribution of the UZI-280 operating system for the CPU280 system.
It was written by Stefan Nitschke and Alexander Schmid.

To use this on a system with a Zilog Z280 MPU, merge the files from the
*dist* folder (i.e. the Z80 version) with those from the *z280dist*
folder onto a drive in the search path (usually drive A in user area 0).
On CP/M 3 the *.COM, *.H, CRTCPM.OBJ and the *.LIB files should be
set with the system file attribute so they can be found from all user
areas.

The *z280dist* folder contains the following files -

```
C280.COM
    The compiler front-end for driving the HI-TECH Z80 C compiler
    for CP/M to produce code to run on a Zilog Z280 MPU.  It accepts
    command lines similar to the C.COM front-end from the *dist*
    folder - with one extra command-line switch to perform optimisation
    of generated assembly language to use the enhanced Z280 instructions.

        c280 -of2 ...

    will cause the OPTIMH.COM pass to perform this optimisation.

OPTIMH.COM and OPTIMH.C
    This is the Z280 assembly language optimiser (with source-code).
    The following instructions are optimised -

        call amul        -> multw hl,de

        call lmul        -> multuw hl,de

        call csv         -> push iy, push ix, lda ix,(sp+0)  (+6 bytes)

        jp cret          -> ld sp,ix pop ix pop iy ret (+6 bytes)

        ld  (hl),c
        inc hl           -> ldw (hl),bc
        ld  (hl),b       -> inc hl

        ld  (hl),e
        inc hl           -> ldw (hl),de
        ld  (hl),d       -> inc hl

        ld  c,(hl)
        inc hl           -> ldw bc,(hl)
        ld  b,(hl)       -> inc hl

        ld  e,(hl)
        inc hl           -> ldw de,(hl)
        ld  d,(hl)       -> inc hl

        ld  c,(ix+n)     or iy
        ld  b,(ix+n+1)   -> ldw bc,(ix+n)    (2 byte)

        ld  e,(ix+n)     or iy
        ld  d,(ix+n+1)   -> ldw de,(ix+n)    (2 byte)

        ld  l,(ix+n)     or iy
        ld  h,(ix+n+1)   -> ldw hl,(ix+n)    (2 byte)

        ld  (ix+n),b     or iy
        ld  (ix+n+1),c   -> ldw (ix+n),bc    (2 byte)

        ld  (ix+n),e     or iy
        ld  (ix+n+1),d   -> ldw (ix+n),de    (2 byte)

        ld  (ix+n),h     or iy
        ld  (ix+n+1),l   -> ldw (ix+n),hl    (2 byte)

        or  a
        sbc hl,bc        -> subw hl,bc       (1 byte)

        or  a
        sbc hl,de        -> subw hl,de       (1 byte)

        push  ix
        pop   de
        ld    hl,nn
        add   hl,de      -> lda hl,(ix+nn)   (3 byte)

        push  ix         -> ld  e,ixl
        pop   de         -> ld  d,ixh

        push  ix
        pop   hl         -> lda hl,(ix+0)

    In addition memory addressing with large offset -

        push  ix
        pop   de
        ld    hl,nn
        add   hl,de      -> lda hl,(ix+nn)   (3 byte)

        push	iy
        pop	de
        ld	hl,nn
        add	hl,de	-> lda	hl,(iy+nn)  (3 byte)

LIB280C.LIB and LIB280F.LIB
    These are the library modules for the Z280 C run-time library
    and floating point library that contain optimised Z280 routines.
    These will only execute on a Z280 MPU - and not on a Z80!
    They have been built from sources of the v3.09-4 release.

```

### Not actually a release - but including MSX-DOS 2 distribution

I'm making available the raw updates for MSX-DOS 2 for those interested in
the last known updates to run HI-TECH C for Z80 under MSX2.  You'll
find the individual files in the msx2dist folder and its sub-folders.

These were extracted from files produced by Arnold Metselaar in 2013 and
distributed as LZH archives on the MSX Banzai web site at
http://msxbanzai.tni.nl/dev/software.html

One of these archive files is known to be damaged - the one containing the
I/O library for MSX-DOS 2 (LDOS2_04.LZH) and I have replaced the source
code for only one of the modules with that from an earlier version obtained
from the Wayback machine at archive.org.  There's still 
a missing (_UTIME.AS) that I'll try to re-assemble from the binary
in the cc_01/LIBDOS2.LIB library. 

The msx2dist/original folder contains the as-downloaded .LZH archives
(including the previous versions from archive.org).

Also note that Yeongman Seo is currently working on a toolchain for developing
MSX-DOS/DOS 2 applications using the HI-TECH Z80 C compiler under a
Windows Win32 environment.  This is available on GitHub at

https://github.com/sharksym/CPMEMU_HI-TECH_C


### Fix to getenv()

The getenv() function was not correctly looking up the location of
the environment file under CP/M 3 for the "default" entry in the drive
search chain.  It will now locate the ENVIRON file if you have set-up
the default search chain with a default entry. For example

```
; Set search chain to current drive, RAMdisk, C: then A:
setdef * m: c: a:
```

in your CP/M 3 PROFILE.SUB file (or manually entered on the command line).

Updated dist/LIBC.LIB and z280dist/LIBC280.LIB too.


### General tidy-up and Overlay capability

Updates merged from @tsupplis

Ron Murray's overlay capabilities have been added, along with some
miscellaneous fixes - mainly to the CP/M BDOS definitions in CPM.H
and to fix an incompatibility with conflicting _TIME macros in both
TIME.H and CPM.H


### Release V3.09-5

Rebuilt libraries LIBC.LIB and LIBF.LIB (as well as LIB280C.LIB and
LIB280F.LIB for the Z280).  New snapshot as Release V3.09-5.


### Fix CRTCPM.OBJ start-up module

The CP/M start-up module CRTCPM.OBJ was not checking the CP/M version
prior to checking for the existence of the PIPEMGR RSX.  This caused
compiled programs under CP/M 2.2 to fail to initialise correctly.

A new version of CRTCPM.OBJ is available (and the source-code in
ZCRTCPM.AS has been updated to correct this).


### Add source for DEHUFF and ENHUFF

Andrey Nikitin has contributed the sources for the DEHUFF and ENHUFF
programs that were used by HI-TECH Software to distribute the library
source files.  I've placed the extracted source files and the resulting
binary produced by the latest compiler in the *huff* folder. Also,
the Huffman-encoded archive containing these sources has been placed
in the *dist* folder as HUFF.HUF.  There's also an original version
that produces some diagnostic errors in HUFFORIG.HUF.

These sources may be built using the HI-TECH C compiler or using the
gcc compiler under Linux or macOS.  I built them natively under CP/M
using -

```
c -O -v enhuff.c encode.c misc.c
c -O -v dehuff.c decode.c misc.c
```

(If you're using a Z280 use 'c280 -of2 -v ...')


### Change supported string length in printf() routine

Merged a minor change into stdio/DOPRNT.C from @tupplis so that various
printf() routines can now output strings greater than 255 characters.

Updated the dist/LIBC.LIB and z280dist/LIB280C.LIB libraries too
and created a new v3.09-6 release to consolidate the recent updates.


### Release V3.09-6

Updated the cpm/C.C source to include the V3.09-6 identifier and
supply a pre-compiled version in dist/C309-6.COM that can be copied
to the CP/M system path (usually drive A in user area 0) as C.COM

Also included a version of the start-up code for the Z280 under CP/M
as z280dist/CRTCPM.OBJ 
(with source code in z280dist/Z280CPM.AS).  This checks whether
the compiled binary is running on a Z280 MPU.


### PIPEMGR sources

I've included Jon Saxton's version of the PIPEMGR RSX for handling
pipes under CP/M 3.

This is a modified version of the original by John Elliot that's
detected and used by each of the releases of HI-TECH C available
from this repository.

You'll find the original PIPEMGR.ARC distribution from Tesseract
volume 089, a PIPEMGR.LBR containing the same files and individual
extracted files in the *pipemgr* folder.

Also included is the source and executable for TEE which has the
PIPEMGR RSX attached and behaves like its namesake in the Unix world,
along with missing files from the original Van Nuys tools
(PIPEMGR.H and PIPEMGR.C).

The rest of the Van Nuys tools are available from John Elliot's
original PIPEMGR page at

http://www.seasip.info/Cpm/software/Pipemgr/index.html


### Self-relocating .COM program PIPEMGR integration

@tsupplis provided updates to the self-relocating image start-up module
to support the improvements from the standard CP/M start-up (in CRTCPM.OBJ).
The source changes are in cpm/ZRRTCPM.AS and the compiled object replaces
dist/RRTCPM.OBJ

To build a self-relocating image, use the -A compile option.


### Minor issues

* Declaration for rename() missing from dist/UNIXIO.H

* Make CPU check in the start-up modules consistent.  Output
message for Z80 (or Z280 MPU) if the incorrect processor is
detected. For example -
```
This CP/M program requires a Z80 CPU.
```

* Include the modified floating-point version of float/DOPRNT.AS
to support outputting strings greater than 255 characters by the
various printf() routines.

* With @tsupplis provided fix for the compiler front-end dist/C309-7.COM
now handles input correctly from submit files - resolving issue #9.
Updated dist/SIGNAL.H, cpm/SIGNAL.C, cpm/C.C and stdio/GETARGS.C
(a mix of input from and output to the console device stream was
previously discarding input from CP/M 3 submit files). 

* The start-up modules have been corrected to properly
report an error when run under MS-DOS. (Previously the assembly language
routine was incorrectly being optimised - and an absolute jump was
changed to a relative branch).

* Updated both dist/LIBC.LIB and dist/LIBF.LIB object libraries
and the Z280 ones in z280dist/LIB280C.LIB and z280dist/LIB280F.LIB.

* Include dist/STDINT.H for some standard type definitions (and referenced
by the overlay support routine).


## MS-DOS cross-compiler available

Recently Microchip Technology Inc. (the current owners of the HI-TECH
intellectual property) also released the MS-DOS version of the HI-TECH
C Compiler v4.11 - a cross-compiler for building CP/M and ROM-able
programs for the Z80 using MS-DOS.  I've uploaded this version to
GitHub at

https://github.com/agn453/HI-TECH-Z80-C-Cross-Compiler


## Release v3.09-7

Thanks to @tsupplis, the following enhancements are now available
in the latest release v3.09-7.

* The compiler front end now supports building CP/M programs using overlays.
Just copy the dist/C309-7.COM to your system directory as C.COM, along
with the symbol table builder dist/SYMTOAS.COM and overlay support library
dist/LIBOVR.LIB.

To build a program with overlays, specify the -F option when compiling
the resident part of the main program (which writes a symbol file) and -Lovr
to use the ovrload() routine; and for each of the overlay segments compile
using the -Y option and specify the symbol file on the
command line.  For example (these files are in the *test* folder too) --

```
Firstly the simple main program

A>type TESTOVR.C
#include <cpm.h>
#include <stdio.h>
#include <overlay.h>

int main(int argc, char ** argv)
{
    intptr_t i;
    printf("overlay test\n");
    i=ovrload("testovr1",0);
    printf("overlay return is %d\n",i);
    i=ovrload("testovr1",0);
    printf("overlay return is %d\n",i);
    i=ovrload("testovr2",0);
    printf("overlay return is %d\n",i);
    i=ovrload("testovr1",0);
    printf("overlay return is %d\n",i);
    return 0;
}

A>type testovr1.c
void * yop(void *a) {
    printf("hello from overlay 01\n");
    return 0;
}

A>type testovr2.c
void * yop(void *a) {
    printf("hello from overlay 02\n");
    return 0;
}

A>c -ftestovrx.sym -o testovr.c -lovr                                         
A:C        COM  (User 0)                                                        
Hi-Tech C Compiler (CP/M-80) V3.09-7                                            
Copyright (C) 1984-87 HI-TECH SOFTWARE                                          
Updated from https://github.com/agn453/HI-TECH-Z80-C                            
                                                                                
A>c -y -o testovr1.c testovrx.sym                                             
A:C        COM  (User 0)                                                        
Hi-Tech C Compiler (CP/M-80) V3.09-7                                            
Copyright (C) 1984-87 HI-TECH SOFTWARE                                          
Updated from https://github.com/agn453/HI-TECH-Z80-C                            
TESTOVR1.C                                                                      
TESTOVRX.SYM

A>>c -y -o testovr2.c testovrx.sym                                             
A:C        COM  (User 0)                                                        
Hi-Tech C Compiler (CP/M-80) V3.09-7                                            
Copyright (C) 1984-87 HI-TECH SOFTWARE                                          
Updated from https://github.com/agn453/HI-TECH-Z80-C                            
TESTOVR2.C                                                                      
TESTOVRX.SYM                                                                    
                                                                                
A>testovr                                                                     
A:TESTOVR  COM                                                                  
overlay test                                                                    
hello from overlay 01                                                           
overlay return is 0                                                             
hello from overlay 01                                                           
overlay return is 0                                                             
hello from overlay 02                                                           
overlay return is 0                                                             
hello from overlay 01                                                           
overlay return is 0                                                             
                                                                                
A>
```

* There are additional testing programs added to the *test* folder along
with the output of a testing run in test/TEST.LOG that was generated on
a CP/M 3 system.

* Since wildcard expansion using _getargs() has been supported since
v3.09-3, it is no longer necessary to use the -R option.  This is now
deprecated and ignored if you use it on the command line.

* A new -H option may be specified to obtain help with the command-line
options.  It types out the system's OPTIONS file - so be sure this is
copied from the dist folder to the system directory (usually drive A:
in user are 0, and marked with the SYS and RO attributes).


### Minor updates

Addressing some minor changes from issue #18 -

* A description of the -E option was missing from the dist/OPTIONS help
message file.

* For CP/M emulators the options emitted by the compiler front-end have
been adjusted to be consistently in uppercase.

* Report an error if the OPTIONS file can't be found (for the -H help text).

I've re-issued this as a revised v3.09-7b release too.


### Release v3.09-8 (support for Z280 MPU)

The compiler front-end source in cpm/C.C has been updated so that it
can be compiled to produce a Z280 MPU version.  Co-existence with the Z80
version is now also possible due to the renaming of the various Z280 versions
start-up modules to C280CPM.OBJ, D280CPM.OBJ and R280CPM.OBJ.

The original source-code to the C280 front-end is lost (not included with the
UZI-280 distribution files) - so now Z280 users (like me) have the updated
version supporting overlays etc.  The original binary from the UZI-280
distribution is in the file z280dist/C280ORIG.COM

You'll find updated Z280 files in the *z280dist* folder.  The
z280dist/C280-8.COM is the new front-end and can be copied to the system
drive as C280.COM.  It tests for, and will only run on a Z280 MPU.
Be sure to copy the start-up object modules (?280CPM.OBJ files) and
the z280dist/LIB280*.LIB libraries to the system drive too.

This is also a new release milestone at v3.09-8.


### Minor fixes

* Minor change to allow size optimization (-O2 switch) for Z280.


### Easy binary download

Include an easy-download CP/M library file containing the binary
distribution (all the files you need from the *dist* folder to get it
running under CP/M).

Download this from

https://raw.githubusercontent.com/agn453/HI-TECH-Z80-C/master/htc-bin.lbr

and extract the files using one of the CP/M .LBR extraction tools (e.g.
NULU.COM).


### Minor update

* Revert change defining "Z80" symbol back to the lowercase form "z80"
in the main driver.  There are some inconsistencies with how this is used
in the other compiler modules - and in the library sourcefiles.


### Z280 optimizer speed/size

* Modify z280/OPTIMH.C and the front-end driver to select between size
and speed optimisations.  Previously some additional code was added to
unroll calls to the csv() routine and jumps to cret() into inline-code.
OPTIMH now accepts a -F option to include them (for faster code), else
it omits them for reduced code-size.  This is selected by using the -OF2
and -O2 option to the C280 front-end.


### Old files removed

Up until now, I've kept various replaced files in this repository
by renaming them to include a hash (or octothorpe) "#" character
in their names.  They have now been removed to avoid confusion.
GitHub is already tracking changes for us - and comparing commits
will show them from now on!  If you need to refer to them you can
download one of the previous releases.


--
Tony Nicholson
19-Jan-2021
