########
Glossary
########

.. glossary::

    runtime library
        Library implementing routines to allow compiled code to interact with
        the runtime environment, such as the operating system.  For example
        the implementation of ``malloc`` and friends belongs in the compiler
        runtime library.

    MSVC
    Microsoft Visual C++
        The Microsoft standard C / C++ compiler. Part of :term:`Microsoft
        Visual Studio`.

    MSVS
    Microsoft Visual Studio
        The Microsoft toolchain for developing applications on Windows.
        Components include MSVC (C / C++) as well as C#, other languages, and
        an IDE.

    gcc
        `GNU compiler collection`_ When we mean the C or C++ compiler
        specifically, we write "gcc c" or "gcc c++".

    MSCRT
        Microsoft C :term:`runtime library` - see `MS runtime libraries`_ and
        `table of MSVC versions and CRTs`_.

    glibc
        The `GNU C runtime library <https://www.gnu.org/software/libc>`_

    mingw
        mingw_: "a contraction of 'Minimalist GNU for Windows', is a
        minimalist development environment for native Microsoft Windows
        applications.  It is a port of gcc to Windows that links against the
        MSCRT rather than the gcc libc.".  Also see the `mingw wikipedia
        page`_.

    mingw-w64
        mingw-w64_: "Mingw-w64 is an advancement of the original mingw.org
        project, created to support the GCC compiler on Windows systems. It
        has forked it in 2007 in order to provide support for 64 bits and new
        APIs."  See: `mingw-w64 history`_ and the `mingw-w64 wikipedia
        section`_.

    binutils
        GNU binutils_: a set of tools to work with compiler output, such as:
        ``ld`` (the GNU linker); ``as`` (the GNU assembler); ``dlltool``
        (makes files for building and working with DLLs); ``ar`` (works with
        archives of compiler object files).  See also the `binutils wikipedia
        page`_.

.. include:: links_names.inc
