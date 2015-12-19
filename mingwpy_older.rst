##############################
Mingwpy retired bitbucket site
##############################

These lists refer to the older files at the `mingwpy bitbucket repo`_.

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

.. include:: links_names.inc
