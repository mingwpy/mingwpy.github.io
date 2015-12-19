####################
Mingwpy distribution
####################

***********
What is it?
***********

mingwpy is a mingw-w64 gcc toolchain customized for building Python
extensions.

See :doc:`issues` for the need of the various customizations.

************
Where is it?
************

Latest versions should be at the `mingwpy anaconda site`_.

There are older versions at the `mingwpy bitbucket repo`_.

*************
What's in it?
*************

See notes_ below.

See :doc:`mingwpy_older` for lists of files in the (older) distribution.

**************************************************
Customizations over standard mingw-builds releases
**************************************************

- two dedicated GCC toolchains for both 32bit (POSIX threads, Dwarf
  exceptions) and 64 bit (POSIX threads, SEH exceptions);
- statically built toolchain based on gcc and mingw-w64;
- languages: C, C++, gfortran, with :term:`LTO` enabled;
- customized 'specs' file for MSVCR90 linkage and manifest support (MSVCR100
  linkage coming soon);
- additional ftime64 patch to allow winpthreads and OpenMP to work with
  MSVCR90 linkage;
- OpenBLAS with windows thread support (OpenMP disabled) included.

.. _notes:

**************************
Notes for the distribution
**************************

libpython import files
======================

The ``libpython27.a`` import files have been generated with ``gendef`` and
``dlltool`` according to the recommendations on the mingw-w64 faq site. It is
essential to not use import libraries from anywhere, but create them with the
tools in the GCC toolchain. The GCC toolchains contain correct generated
``mscvrXXX`` import files by default.

OpenBLAS files
==============

The openblas DLL must be copied to numpy/core before building numpy. All Blas
and Lapack code will be linked dynamically to this DLL.  Because of this the
overall distro size gets much smaller compared to numpy-MKL or scipy-MKL. It is
not necessary to add numpy/core to the path!  (at least on my machine). To load
libopenblas.dll to the process space it is only necessary to import numpy |--|
nothing else. libopenblas.dll is linked against the msvcr90.dll, just like
python. The DLL itself is a fat binary containing all optimized kernels for
all supported platforms. DLL, headers and import files have been included into
the toolchain.

*************************
Compiling numpy and scipy
*************************

Compiling numpy
===============

#. <mingw>\bin and python should be in the PATH. Choose 32 bit or 64 bit architecture.
#. copy libpython27.a to <python>\libs check, that <python>\libs does not
   contain libmsvcr90.a
#. apply numpy.patch
#. copy libopenblas.dll from <mingw>\bin to numpy\core of course don't ever mix
   32bit and 64 bit code
#. create a site.cfg in the numpy folder with the absolute path to the mingw
   import files/header files. I copied the openblas header files, importlibs
   into the GCC toolchain.
#. create a mingw distutils.cfg file
#. test the configuration::

        python setup.py config_fc --verbose
        python setup.py build --help-fcompiler

#. build::

        python setup.py build --fcompiler=gnu95

#. make a exe installer::

        python setup.py bdist --format=wininst

#. make a wheel:

   Example for built 32-bit exe installer::

        wininst2wheel numpy-1.8.0.win32-py2.7.exe

#. install::

        wheel install numpy-1.8.0-cp27-none-win32.whl

#. test::

        python -c 'import numpy; numpy.test()'

Compiling scipy
===============

#. apply scipy.patch
#. Check fortran compiler with::

        python setup.py build --fcompiler=gnu95

   and a second time::

        python setup.py build --fcompiler=gnu95

#. Build exe installer::

        python setup.py bdist --format=wininst

#. install

#. test with::

        import scipy; scipy.test()

.. include:: links_names.inc
