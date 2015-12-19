############
Why mingwpy?
############

Python.org distributes binary installers for Python that are built with
:term:`MSVS`.

It is possible to install a version of MSVS that will build Python extensions
written in C.

So |--| why do we need mingwpy?

********************************************
MSVS can be painful to install and configure
********************************************

There is a free and relatively small MSVS version 9.0 download for compiling
Python 2.7 extensions at http://aka.ms/vcpython27. Quoting from
https://mail.python.org/pipermail/distutils-sig/2014-September/024885.html :

    [...] VC9 is no longer supported by Microsoft. This means there won't be
    any improvements or bug fixes coming, and there's no official support
    offered.

Compiling extensions for Python 3.5 needs MSVS version 14 (VS 2015).  The
"community edition" is a free download but takes a whopping 11GB of disk space
and takes a long time to install.

**************************************************
Scientific software often needs a Fortran compiler
**************************************************

See : `numerical software on Windows`_.

MSVS does not have a Fortran compiler. This means that anyone compiling scipy
or another Python package with Fortran code will need a compiler other than
(or in addition to) MSVS.

To our knowledge, the only freely-available Fortran compilers on Windows are
those from the :term:`gcc`, and specifically, ``g77`` / ``gfortran`` from
:term:`mingw` and :term:`mingw-w64`.

*******************************************************************
High performance numerical kernels aren't written with MSVS in mind
*******************************************************************

All of the competitive open-source linear algebra libraries
(e.g. OpenBLAS) contain crucial chunks of assembly language code, and
this code is written using gcc conventions that aren't supported by
MSVS.

***********************************************
Open-source depends on free access to compilers
***********************************************

Open source works by spreading the work of maintaining and distributing
packages across many people.  If it is hard to install the compilers then
fewer people will be able to test and fix bugs, meaning slower development and
more work in releasing packages.

************************************************
Cross-platform tools make for easier maintenance
************************************************

Scientific programmers overwhelmingly work on Unix, and are used to gcc c /
gfortran or clang. Few scientific programmers know MSVC. Using MSVC instead of
gcc / clang makes extra maintenance work and reduces the pool of developers
who can contribute.

************************************************
Open-source programmers prefer open-source tools
************************************************

Open-source programmers prefer to work with open tools.  Many of us have
switched to Python from languages like MATLAB for this reason.  It is
uncomfortable to have to use closed-source tools to build open-source
software.

.. include:: links_names.inc
