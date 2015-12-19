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

Latest versions should be at:
https://bitbucket.org/carlkl/mingw-w64-for-python/downloads

*************
What's in it?
*************

See notes_ below.

#. patches used::

    numpy.patch
    scipy.patch

#. 64 bit GCC toolchain::

    amd64/
        mingw-w64-toolchain-static_amd64-gcc-4.8.2_vc90_rev-20140131.7z
        libpython27.a

   #. numpy-1.8.0 linked against OpenBLAS::

       amd64/numpy-1.8.0/
           numpy-1.8.0.win-amd64-py2.7.exe
           numpy-1.8.0-cp27-none-win_amd64.whl
           numpy_amd64_fcompiler.log
           numpy_amd64_build.log
           numpy_amd64_test.log
           _numpyconfig.h
           config.h

   #. scipy-0.13.3 linked against OpenBLAS::

       amd64/scipy-0.13.3/
           scipy-0.13.3.win-amd64-py2.7.exe
           scipy-0.13.3-cp27-none-win_amd64.whl
           scipy_amd64_fcompiler.log
           scipy_amd64_build.log
           scipy_amd64_build_cont.log
           scipy_amd64_test._segfault.log
           scipy_amd64_test.log

#. 32 bit GCC toolchain::

    win32/
        mingw-w64-toolchain-static_win32-gcc-4.8.2_vc90_rev-20140131.7z
        libpython27.a

   #. numpy-1.8.0 linked against OpenBLAS::

       win32/numpy-1.8.0/
           numpy-1.8.0.win32-py2.7.exe
           numpy-1.8.0-cp27-none-win32.whl
           numpy_win32_fcompiler.log
           numpy_win32_build.log
           numpy_win32_test.log
           _numpyconfig.h
           config.h

   #. scipy-0.13.3 linked against OpenBLAS::

       win32/scipy-0.13.3/
           scipy-0.13.3.win32-py2.7.exe
           scipy-0.13.3-cp27-none-win32.whl
           scipy_win32_fcompiler.log
           scipy_win32_build.log
           scipy_win32_build_cont.log
           scipy_win32_test.log

**************************************************
Customizations over standard mingw-builds releases
**************************************************

- two dedicated GCC toolchains for both 32bit (POSIX threads, Dwarf exceptions)
  and 64 bit (POSIX threads, SEH exceptions)
- statically build toolchain based on gcc-4.8.2 and mingw-w64 v 3.1.0
- languages: C, C++, gfortran, LTO
- customized 'specs' file for MSVCR90 linkage and manifest support (MSVCR100 linkage coming soon)
- additional ftime64 patch to allow winpthreads and OpenMP to work with MSVCR90 linkage
- openblas-2.9rc1 with windows thread support (OpenMP disabled) included

.. _notes:

**************************
Notes for the distribution
**************************

libpython import files
======================

The ``libpython27.a`` import files have been generated with gendef and dlltool
according to the recommendations on the mingw-w64 faq site. It is essential to
not use import libraries from anywhere, but create it with the tools in the GCC
toolchain. The GCC toolchains contains correct generated mscvrXX import files
per default.

OpenBLAS files
==============

The openblas DLL must be copied to numpy/core before building numpy. All Blas
and Lapack code will be linked dynamically to this DLL.  Because of this the
overall distro size gets much smaller compared to numpy-MKL or scipy-MKL. It is
not necessary to add numpy/core to the path!  (at least on my machine). To load
libopenblas.dll to the process space it is only necessary to import numpy -
nothing else. libopenblas.dll is linked against the msvcr90.dll, just like
python. The DLL itself is a fat binary containing all optimized kernels for all
supported platforms. DLL, headers and import files have been included into the
toolchain.

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
