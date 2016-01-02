############################################
Funding request - MinGW-w64 static toolchain
############################################

Authors: Ralf Gommers, Matthew Brett, Nathaniel Smith, Olivier Grisel and Carl Kleffner

*****************
Executive summary
*****************

**Grant objective**: provide a modern, freely available, easy to install
compiler toolchain that allows building and distributing the scientific Python
stack on Windows.

**Problem statement**: the situation with building the scientific Python stack
on Windows is reaching breaking point.  There is no freely available compiler
toolchain that can be used to build 64-bit installers for Scipy or any Scipy
installers for Python 3.5. There are no Windows wheels for Numpy, Scipy and
many other projects on PyPi. Windows issues are a major burden for release
managers.

**Key benefits** for end users expected to be quickly realized after achieving
the objective of this grant are:

- make official 64-bit Windows binary installers for the core Scipy Stack;
- enable Numpy and Scipy Windows wheels on PyPi;
- clear path for other scientific Python projects to support Windows wheels on Pypi.

For developers the key benefits are:

- provide a usable free, easy to install compiler toolchain on Windows, thus
  increasing the pool of developers who can support Windows builds;
- provide a way forward for Python 3.5;
- move beyond Fortran 77.

**Funding request**: We request $5k in total from NumFOCUS, $1k of which to
come from Numpy/Scipy funds, as well as endorsement of a request for $5k to the
PSF working group for Python in Scientific Computing and Data Analysis.


*******
Context
*******

Python.org Windows installers are built with specific versions of Microsoft
Visual C++ (MSVC).  When we build extension modules with MSVC, we need the
specific version of MSVC with which Python was compiled.  Thus building
extensions for each of Python 2.7, 3.4 and 3.5 needs a different copy of MSVC
installed and configured `[10]`_ .  Installation and configuration is not always
easy.   Although it is possible to find freely available versions of MSVC for
these Pythons, there is no freely available Fortran compiler, making it
impossible to build Scipy and other packages depending on Fortran code.

For these reasons, Numpy and Scipy have been using a Windows GCC toolchain
based on MinGW for their public releases.  However, the MinGW project is now
dormant and the toolchain is aging.  Symptoms of this aging are that it does
not support 64-bit compilation or versions of Fortran beyond f77.

The obvious replacement for MinGW is the newer MinGW-w64 project.  This does
support 64-bit compilation and modern GFortran.  However, there are problems to
overcome in configuring MinGW-w64 to compile extensions with acceptable
mathematical precision and compatibility with code compiled with MSVC.

Carl Kleffner has been working on solving these and other problems for the last
2 years `[5]`_.  His current MinGW-w64-based toolchain is capable of building the
core packages of the Scipy stack (see `[6]`_, `[7]`_); there are rough edges however.
The aim of this proposal is to document this toolchain, and make it
user-friendly and maintainable, in order for the scientific Python community to
be able to switch to it for the Windows binaries of the releases of its core
projects.  There is significant work involved in achieving this
user-friendliness and maintainability to the level required for acceptance by
the core Scipy Stack projects.  Furthermore, significant work is needed to
support MSVC 2015 - needed to support Python 3.5 and up.  This effort requires
funding, to allow Carl Kleffner (the proposed contractor) to spend more time
on it than only spare hours in the evening.


***********************
Funding request details
***********************

The overall funding request for the planned work is split in 3 phases, for a
total amount of $10000. The request for the first phase is $1000; for the
second phase $4000 and for the third phase $5000.  There are several reasons
for this phasing:

1. Enable fast approval of the phase 1 and phase 2 requests.  The aim is to get
   started with the work as soon as possible (target: 1 Jan 2016).
2. Allow a split in funding sources.  The request for phase 1 is to NumFOCUS
   (funds donated to Numpy/Scipy), phase 2 also to NumFOCUS (non-project
   funds), while for phase 3 it will be to the PSF fund for scientific Python
   related grants.
3. Good progress during phases 1 and 2 will build confidence that the final
   goals of the larger phase 3 request will be met.

As stated above, the request to NumFOCUS now is to approve the phase 1 request
of $1000 and the phase 2 request of $4000.  In light of the significance of
this work for the whole Scipy Stack, the Numpy and Scipy projects are willing
to together contribute $1000 of the funds donated to them via NumFOCUS, which
covers phase 1.

The proposal is to execute the work on a contractor basis, with a set of
milestones to be achieved for each tranche of funds.  The contractor will be
Carl Kleffner.  The other people involved in the work, to support Carl
and act as community liaisons, are:

- Matthew Brett (MingwPy project - community & open sourcing aspects)
- Nathaniel Smith (lead interaction with upstream MinGW-w64)
- Ralf Gommers (support numpy.distutils / setuptools integration)
- Olivier Grisel (toolchain reproducibility & Windows CI)

**Main milestones** for phase 1:

1. Patches and a build script for the static toolchains are made available
   publicly. Build doesn’t have to be fully automated, but (with some help from
   Carl if needed) can be reproduced by at least one other person.

For phase 2:

2. MingwPy project is set up under the MingwPy Github org `[4]`_ and contains
   documented, maintainable build scripts for the toolchain itself.
3. Numpy and Scipy master branches can be built with that toolchain for Python
   2.7, win32 + win64, with at most some test failures due to limited numerical
   precision left (those will be addressed in phase 3).
4. Continuous integration is set up on one of Appveyor or TravisCI with the
   toolchain for Numpy, Scipy and scikit-learn.
5. Design documentation explaining the high-level design and choices made
   related to linking against MSVC runtimes, threading model, exception
   handling and floating point library support.

And for phase 3:

6. Numerical precision of toolchain is improved to the extent that the full
   Numpy and Scipy test suites pass.
7. Toolchain is installable as one or more wheels from PyPi with "pip install mingwpy".
8. A working prototyped solution for Python 3.5 is demonstrated.  The main
   principles of the solution are required to be acceptable to upstream MinGW-w64,
   however the prototype does not have to be in a state that's completely ready
   for integration upstream (this is more work, and is dependent on upstream).


**************************
Detailed problem statement
**************************

The current 32-bit Windows installers for Numpy and Scipy are built with MinGW
3.4.5 (see `[2]`_ for details). They’re compiled against three variants of ATLAS,
which differ in the instruction sets they support (no SSE, SS2 or SSE3). The
installers are .exe files with a GUI, and therefore cannot be used in batch
installed. Building wheels for these ATLAS based builds isn’t possible; the
wheel format doesn’t support selecting the right instruction set at install
time. Having no Windows wheels for Numpy and Scipy on PyPy has also blocked
many other projects from providing Windows wheels. The difficulty of building
Windows installers is a major burden on release managers for scientific Python
projects.

Development of core libraries like NumPy and SciPy is hobbled by the need to
work around bugs and limitations of old compilers (e.g., lack of support for
C99 and Fortran 95). Issues with the current outdated MinGW setup are the main
bottleneck in making releases, and regularly lead to delays of a month or more.
Windows CI isn’t available, therefore build issues and test failures are
usually detected too late.

Another important issue is that Cython, and related technologies that depend on
a working compiler, is hard to use in introductory classes because setting up
the Microsoft Visual Studio compilers is difficult and error-prone.


*********************
More on proposed work
*********************

The work to achieve the milestones listed above will land in the Github MingwPy
org `[4]`_; documentation is already being set up at `[9]`_.

The target for supported Python versions is 2.7 and 3.5.  Python 2.7 support
must work but is not required to be pushed to upstream MinGW-w64.  For Python
3.5 (and up), upstream MinGW-w64 support for the universal common runtime in
MSVC 2015 is necessary in order to have a situation that's maintainable longer
term.

Additional benefits
-------------------

There are a number of benefits in addition to the ones mentioned in the
*executive summary*. These are:

- Having a pip-installable compiler makes it significantly easier to work and
  teach with Cython.
- A pip-installable compiler also makes setting up Windows CI feasible for
  Scipy.
- Windows 32-bit builds will use the same MinGW-w64 based toolchain and
  therefore move from GCC 3.4.5 and g77 to GCC/GFortran >= 4.92. This allows
  the use of Fortran 90/95 code.
- OpenBLAS is currently the best option for a high performance open source
  BLAS/LAPACK implementation. The work in this proposal would enable using
  OpenBLAS instead of ATLAS.


********************
Technical background
********************

Why a static GCC toolchain?
---------------------------

GCC as well as the all necessary libraries for a tool chain can be compiled
with the *-disabled-shared* option at configure time. The GCC runtime object
code will be statically linked into the binaries. As a consequence no
dependencies to external GCC runtime libraries will be created. Deployment of
Python extensions is therefore greatly improved.

Considered alternatives
-----------------------

Two alternative options to improve the build situation on Windows were
considered:

1. Making GFortran work with MSVC directly.
2. A free solution involving Intel MKL and Intel compilers.

Option (1) is potentially easier to maintain than the proposed work (see
`[5]`_ for a report of some experimentation in this direction done in 2009).
However, this would rely on MSVC remaining freely available, which is not a
given.  It also would not provide all the benefits that this proposal brings:
the toolchain wouldn’t be pip-installable, and it wouldn’t support C99.
Finally: no one is working on this, while significant work has already been
done on the MinGW-w64 toolchain.

Option (2) has been explored, but isn’t feasible. MKL is now free, but ifort
(the Intel Fortran compiler) not yet. So that's still not a free toolchain.  In
addition, there are legal issues with distributing the binaries even if ifort
would become free.  There has been contact with Intel; our conclusion was that
there is little chance that a free solution will be made available by Intel in
the near future. (In fact, our contacts at Intel have encouraged us to move
ahead with the MinGW-w64 solution, and helped us locate BSD-licensed code to
improve the numerical precision issues that was previously contributed by Intel
to Android.)


Details on MinGW-w64 & the static toolchain
-------------------------------------------

There are several mutually incompatible 64-bit MinGW-based toolchains being
distributed. The most active and promising project is MinGW-w64 `[8]`_.
Therefore the static toolchain will be based on this project.

The static toolchains (separate ones for 32-bit and 64-bit) have to be built on
Windows, in the MSYS2 Posix shell. Cross-compiling on LINUX is under
examination, but isn’t a priority.

Customizations not present in the standard MinGW-w64 releases include:

- dedicated GCC tool chains for both 32-bit (win32 thread model, sljl
  exceptions) and 64 bit (win32 thread model, SEH exceptions)
- statically build tool chain based on gcc-4.9.2 and MinGW-w64 v4 (trunk)
- 'specs' files provided for MSVCR90/100 linkage and manifest support
- OpenBLAS >= 2.12 (optimized BLAS, LAPACK) with windows threads model support


**********
References
**********

[1] https://bitbucket.org/carlkl/mingw-w64-for-python/downloads

[2] https://github.com/numpy/numpy-vendor

[3] https://github.com/scipy/scipy/issues/2829

[4] https://github.com/mingwpy

[5] https://cournape.wordpress.com/2009/06/28/progress-for-numpy-on-windows-64-bits/

[6] https://github.com/numpy/numpy/issues/5194

[7] https://github.com/numpy/numpy/wiki/Mingw-static-toolchain

[8] http://sourceforge.net/projects/mingw-w64/

[9] http://mingwpy.github.io/

[10] https://matthew-brett.github.io/pydagogue/python_msvc.html


.. _[1]: https://bitbucket.org/carlkl/mingw-w64-for-python/downloads

.. _[2]: https://github.com/numpy/numpy-vendor

.. _[3]: https://github.com/scipy/scipy/issues/2829

.. _[4]: https://github.com/mingwpy

.. _[5]: https://cournape.wordpress.com/2009/06/28/progress-for-numpy-on-windows-64-bits/

.. _[6]: https://github.com/numpy/numpy/issues/5194

.. _[7]: https://github.com/numpy/numpy/wiki/Mingw-static-toolchain

.. _[8]: http://sourceforge.net/projects/mingw-w64/

.. _[9]: http://mingwpy.github.io/

.. _[10]: https://matthew-brett.github.io/pydagogue/python_msvc.html
