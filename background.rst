##########
Background
##########

********************************
Why choose mingw-w64 over mingw?
********************************

:term:`mingw-w64` is a fork of the :term:`mingw` project.

To date, numpy and scipy have used versions of the mingw compilers to create
32-bit binary distributions on Windows.  mingw-w64 has some major advantages
over mingw, including:

* support for compiling 64-bit extensions (as well as 32-bit);
* large file support;
* winpthread pthreads implementation, MIT licenced;
* more active development.

*********************
The mingw-w64 project
*********************

The mingw-w64 project releases the supporting files to allow the standard
:term:`gcc` distribution to compile code that can:

* make many standard Windows API calls;
* link against standard :term:`MSCRT` :term:`runtime library` files.

Among the files the project provides are:

* ``mingw-w64-headers`` |--| header files specific to the Windows
  implementation.
* ``mingw-w64-crt`` |--| code for C / C++ runtime libraries (providing
  functions not directly or reliably available from the MSCRT);
* ``mingw-w64-libraries`` |--| libraries for using POSIX threads (`winpthreads
  <http://locklessinc.com/articles/pthreads_on_windows/>`_), working with MSVC
  symbol name mangling (`libmangle
  <http://mingw-w64.sourceforge.net/libmangle>`_), Windows store compatibility
  (`winstorecompat
  <http://www.jbkempf.com/blog/post/2013/Technical-update-on-the-WinRT-port>`_),
  structured exception handling (`pseh <http://www.reactos.org/wiki/PSEH>`_);
* ``mingw-w64-tools`` |--| tools for: generating DLL export definition
  (``def``) files (`gendef
  <http://sourceforge.net/p/mingw-w64/wiki2/gendef>`_); working with the
  `Microsoft Interface Description Language
  <https://en.wikipedia.org/wiki/Microsoft_Interface_Definition_Language>`_
  (`genidl <http://manpages.ubuntu.com/manpages/precise/man1/genidl.1.html>`_,
  `widl <http://manpages.ubuntu.com/manpages/precise/man1/widl.1.html>`_);
  modifying `PE`_ flags in Windows executable files (`genpeimg`).

The project has official source releases on sourceforge: see `mingw-w64 source
releases`_.

In order to use mingw-w64 implementations to compile real code, you will need
a mingw-w64 compiler toolchain.

**********************
The compiler toolchain
**********************

To build software with mingw-w64 you will likely need:

* the compiler (gcc, compiled from standard gcc sources), augumented with *
  the mingw-w64 extensions, including Windows-specific headers and runtime
  libraries;
* a build of the GNU :term:`binutils` tools, containing utility programs such
  as ``ld``, ``ar``, ``dlltool``;

********************
C++ exception models
********************

The mingw-w64 compiler can be built with different ways of handling C++
exceptions. See `mingw-w64 exception handling`_ and `SO post on mingw-w64
exception handling`_ for background.

The exception handling modes are:

* Dwarf2: only supported on 32-bit Windows;
* setjmp / longjmp (SJLJ): "much slower than Dwarf2", but compatible on 32 and
  64 bits;
* Structured exception handling (SEH): see: `SEH and Mingw
  <http://www.programmingunlimited.net/siteexec/content.cgi?page=mingw-seh>`_

****************
Threading models
****************

The mingw-w64 compiler can also be built with different threading models. The
options are:

* POSIX threads (full C++11/C11 threading features);
* win32 threads (No C++11 multithreading features).

See these SO posts:

* `difference bewteen posix and win32 threads in mingw-w64
  <http://stackoverflow.com/questions/13212342/whats-the-difference-between-thread-posixs-and-thread-win32-in-gcc-port-of-windo>`_
* `posix vs win32
  <http://stackoverflow.com/questions/17242516/mingw-w64-threads-posix-vs-win32>`_

*************************************
A more complete software build system
*************************************

In practice, for building software, you will usually want a wider range of
tools than just the compiler and binutils utilities.  In particular you will
probably want tools like the ``bash`` shell, ``make``, and the `GNU build
system <https://en.wikipedia.org/wiki/GNU_build_system>`_.

MSYS2_ provides these.  msys2 is the successor of msys. Msys2 is necessary as
environment for the mingw build process on Windows.  -
http://sourceforge.net/p/msys2/wiki/MSYS2%20installation/

*********************************
Official mingw-w64 GCC toolchains
*********************************

'recommended' builds are available from the mingw-builds project at
http://mingw-w64.sourceforge.net/download.php#mingw-builds for example:

- http://sourceforge.net/projects/mingw-w64/files/Toolchains%20targetting%20Win64/Personal%20Builds/mingw-builds/4.8.2/threads-posix/seh/
- http://sourceforge.net/projects/mingw-w64/files/Toolchains%20targetting%20Win32/Personal%20Builds/mingw-builds/4.8.2/threads-posix/dwarf/

These are common combinations of exception and thread models. You can find
other combinations as well. Exception handling affects C++ development. Don't
ever link object code with different types of exception and/or thread
handling!

Threads concerning the question 'where to find mingw-w64 builds':

- http://article.gmane.org/gmane.comp.gnu.mingw.w64.general/7700
- http://article.gmane.org/gmane.comp.gnu.mingw.w64.general/8484

*********************************************************
Other sources of precompiled mingw-w64 compiled libraries
*********************************************************

Recent mingw-w64 based tools and library packages together with sources and
patches are available from archlinux as well as from the msys2 maintainers.

- http://sourceforge.net/projects/mingw-w64-archlinux/files/  (i686: Sjlj | x86_64: SEH)
- http://sourceforge.net/projects/msys2/files/REPOS/MINGW/  (i686: Dwarf | x86_64: SEH)

***************************************************
Building a mingw-w64 based GCC toolchain on Windows
***************************************************

"mingw-builds" is a set of scripts and patches for compiling the GCC toolchain
under Windows with the help of msys2 POSIX environment -
https://github.com/niXman/mingw-builds/tree/develop recent 'mingw-builds' GCC
toolchains can be downloaded from the mingw-w64 sf.net site:
http://sourceforge.net/projects/mingw-w64/files/Toolchains%20targetting%20Win64/Personal%20Builds/mingw-builds/

.. include:: links_names.inc
