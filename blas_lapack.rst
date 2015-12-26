########################
BLAS / LAPACK on Windows
########################

Windows has no default BLAS_ / LAPACK_ library.  By "default" we mean,
installed with the operating system.

Numpy needs a BLAS library that has CBLAS_ C language wrappers.

Here is a list of the options that we know about.

*****
ATLAS
*****

The ATLAS_ libraries have been the default BLAS / LAPACK libraries for numpy
binary installers on Windows to date (end of 2015).

ATLAS uses comprehensive tests of parameters on a particular machine to chose
from a range of algorithms to optimize BLAS and some LAPACK routines.  Modern
versions (>= 3.9) perform reasonably well on `benchmarks
<http://cran.r-project.org/web/packages/gcbd/vignettes/gcbd.pdf>`_.
Each ATLAS build is optimized for a particular machine (CPU
capabilities, L1 / L2 cache size, memory speed), and ATLAS does not select
routines at runtime but at build time, meaning that a default ATLAS build can
be badly optimized for a particular processor.  The main developer of ATLAS is
Clint Whaley.  His main priority is optimizing for HPC machines, and he does
not give much time to supporting Windows builds.  Not surprisingly, ATLAS is
difficult to build on Windows, and is `not well optimized for Windows 64 bit
<http://math-atlas.sourceforge.net/atlas_install/node57>`_.

Advantages:

* Very reliable;
* BSD license;

Disadvantages:

* By design, the compilation step of ATLAS tunes the output library to the
  exact architecture on which it is compiling.  This means good performance
  for machines very like the build machine, but worse performance on other
  machines;
* No runtime optimization for running CPU;
* Has only one major developer (Clint Whaley);
* Compilation is difficult, slow and error-prone on Windows;
* Not optimized for Windows 64 bit

Because there is no run-time adaptation to the CPU, ATLAS built for a CPU with
SSE3 instructions will likely crash on a CPU that does not have SSE3
instructions, and ATLAS built for a SSE2 CPU will not be able to use SSE3
instructions.  Therefore, numpy installers on Windows use the "superpack"
format, where we build three ATLAS libraries:

* without CPU support for SSE instructions;
* with support for SSE2 instructions;
* with support for SSE3 instructions;

We make three Windows ``.exe`` installers, one for each of these ATLAS
versions, and then build a "superpack" installer that first checks the machine
on which the installer is running, to find what instructions the CPU supports,
and the installs the matching numpy / ATLAS package.

There is no way of doing this when installing from a binary wheel_, because
the wheel installation process consists of unpacking files to given
destinations, and does not allow pre-install or post-install scripts.

One option would be to build a binary wheel with ATLAS that depends on SSE2
instructions.  It seems that 99.5% of Windows machines have SSE2 (see:
`Windows versions`).  It technically not difficult to put a check in the numpy
``__init__.py`` file to give a helpful error message and die when the CPU does
not have SSE2::

    try:
        from ctypes import windll, wintypes
    except (ImportError, ValueError):
        pass
    else:
        has_feature = windll.kernel32.IsProcessorFeaturePresent
        has_feature.argtypes = [wintypes.DWORD]
        if not has_feature(10):
            msg = ("This version of numpy needs a CPU capable of SSE2, "
                    "but Windows says - not so.\n",
                    "Please reinstall numpy using a superpack installer")
            raise RuntimeError(msg)

*************************
Intel Math Kernel Library
*************************

The MKL_ has a reputation for being fast, particularly on Intel chips (see the
`MLK Wikipedia entry`_). It has good performance on `benchmarks
<http://www.wittwer.nl/wp-content/uploads/2009/08/blas_lapack.pdf>`_ across
the range, `except on AMD processors
<http://www.agner.org/optimize/blog/read.php?i=49#49>`_.

It is closed-source, but available for free under the `Community licensing
program <https://software.intel.com/sites/campaigns/nest>`_.

The `Intel license
<https://software.intel.com/en-us/articles/end-user-license-agreement>`_ does
allow us, the developers, to distribute copies of the MKL with our built
binaries, but carries the following extra license terms:

    H. DISTRIBUTION: Distribution of the Redistributables is also subject to
       the following limitations:
       [clauses i through iii omitted]

       [You] (iv) will provide the Redistributables subject to a license
       agreement that prohibits disassembly and reverse engineering of the
       Redistributables except in cases when you provide Your Product subject
       to an open source license that is not an Excluded License, for example,
       the BSD license, or the MIT license, (v) will indemnify, hold harmless,
       and defend Intel and its suppliers from and against any claims or
       lawsuits, including attorney's fees, that arise or result from Your
       modifications, derivative works or Your distribution of Your Product.

Clause iv above requires us to somehow get our users to agree to extra license
terms. This is presumably not practical using an automated installation method
like pip / pypi.  Clause v makes us, the developers responsible for legal cost
that could be very large.

See discussions about `MKL on numpy mailing
list
<http://numpy-discussion.10968.n7.nabble.com/Windows-wheels-using-MKL-td37097.html>`_
and `MKL on Julia issues <https://github.com/JuliaLang/julia/issues/4272>`_.

Advantages:

* At or near maximum speed;
* Runtime processor selection, giving good performance on a range of different
  CPUs.

Disadvantages:

* Closed source;
* License terms prevent automated installation;
* License terms probably not acceptable to numpy developers.

*********************
AMD Core Math Library
*********************

The ACML_ was AMD's equivalent to the MKL, with `similar
<http://eigen.tuxfamily.org/index.php?title=Benchmark-March2009>` or
`moderately worse
<http://www.wittwer.nl/wp-content/uploads/2009/08/blas_lapack.pdf>`_
performance.  As of time of writing (December 2015), AMD has marked the ACML
as "end of life", and suggests using the `AMD compute libraries`_ instead.

The ACML does not appear to contain a CBLAS_ interface.

Binaries linked against ACML have to conform to the `ACML license
<http://amd-dev.wpengine.netdna-cdn.com/wordpress/media/2013/12/ACML_June_24_2010_v2.pdf>`_
which, as for the MKL, requires software linked to the ACML to subject users
to the ACML license terms including:

  2. Restrictions. The Software contains copyrighted and patented
  material, trade secrets and other proprietary material. In order to
  protect them, and except as permitted by applicable legislation, you
  may not:

  a) decompile, reverse engineer, disassemble or otherwise reduce the
  Software to a human-perceivable form;

  b) modify, network, rent, lend, loan, distribute or create derivative
  works based upon the Software in whole or in part [...]

*********************
AMD compute libraries
*********************

AMD advertise the `AMD compute libraries`_ (ACL) as the successor to the ACML.

The ACL page points us to :ref:`blis` for BLAS and :ref:`libflame` for LAPACK.

.. _blis:

******************************************
BLAS-like instantiation software framework
******************************************

BLIS_ is "a portable software framework for instantiating high-performance BLAS-like dense linear algebra libraries."

It provides a `superset of BLAS
<https://github.com/flame/blis/wiki/FAQ#im-not-really-interested-in-all-of-these-newfangled-features-in-blis-can-i-just-use-blis-as-a-blas-library>`_,
along with a `CBLAS layer
<https://github.com/flame/blis/wiki/FAQ#what-about-cblas>`_.

It can be compiled into a BLAS library. As of writing (December 2015) `Windows
builds are experimental
<https://github.com/flame/blis/wiki/FAQ#can-i-build-blis-on-windows--mac-os-x>`_.
BLIS `does not currently do run-time hardware detection
<https://github.com/flame/blis/wiki/FAQ#does-blis-automatically-detect-my-hardware>`_.

As of December 2015 the `developer mailing list
<https://groups.google.com/forum/#!forum/blis-devel>`_ was fairly quiet, with
only a few emails since August 2015.

Advantages:

* portable across platforms;
* modern architecture.

Disadvantages:

* Windows builds are experimental;
* No runtime hardware detection.

.. _libflame:

********
libflame
********

`libflame`_ is an implementation of some LAPACK routines. See the `libflame
project page <http://www.cs.utexas.edu/~flame/web/libFLAME.html>`_ for more
detail.

libflame can also a full LAPACK implementation.  It is a sister project to
BLIS.

******
COBLAS
******

`COBLAS <https://github.com/rljames/coblas>`_ is a "Reference BLAS library in
C99", BSD license. A quick look at the code in April 2014 suggested it used
very straightforward implementations that are not highly optimized.

*******************************
Netlib reference implementation
*******************************

See `netlib BLAS`_ and `netlib LAPACK`_.

Most available benchmarks (e.g `R benchmarks
<http://cran.r-project.org/web/packages/gcbd/vignettes/gcbd.pdf>`_, `BLAS
LAPACK review`_) show the reference BLAS / LAPACK to be considerably slower
than any optimized library.

*****
Eigen
*****

_Eigen is "a C++ template library for linear algebra: matrices, vectors,
numerical solvers, and related algorithms."

Mostly covered by the Mozilla Public Licence 2, but some features covered by
the LGPL.  `Non-MPL2 features can be disabled
<http://eigen.tuxfamily.org/index.php?title=FAQ#Disabling_non_MPL2_features>`_

It is technically possible to compile Eigen into a BLAS library, but there is
currently no CBLAS interface.

See `Eigen FAQ entry discussing BLAS / LAPACK
<http://eigen.tuxfamily.org/index.php?title=FAQ#How_does_Eigen_compare_to_BLAS.2FLAPACK.3F>`_.

*********
GotoBLAS2
*********

GotoBLAS_ is the predecessor to OpenBLAS_.  It was a library written by
Kazushige Goto, and released under a BSD license, but is no longer maintained.
Goto now works for Intel. It was at or near the top of benchmarks on which it
has been tested (e.g `BLAS LAPACK review`_ `Eigen benchmarks
<http://eigen.tuxfamily.org/index.php?title=Benchmark>`_).  Like MKL and ACML,
GotoBLAS2 chooses routines at runtime according to the processor. It does not
detect modern processors (after 2011).

********
OpenBLAS
********

OpenBLAS_ is a fork of GotoBLAS2 updated for newer processors.  It uses the
3-clause BSD license.

Julia_ uses OpenBLAS by default.

See `OpenBLAS on github`_ for current code state.  It appears to be `actively
merging pull requests
<https://github.com/xianyi/OpenBLAS/pulls?direction=desc&page=1&sort=created&state=closed>`_.
There have been some worries about bugs and lack of tests on the `numpy
mailing list
<http://mail.scipy.org/pipermail/numpy-discussion/2014-March/069659.html>`_
and the `octave list
<http://article.gmane.org/gmane.comp.gnu.octave.maintainers/38746>`_.

It appears to be `fast on benchmarks
<https://github.com/tmolteno/necpp/issues/18>`_.

OpenBLAS on Win32 seems to be quite stable. Some OpenBLAS issues on Win64 can
be adressed with a single threaded version of that library.

Advantages:

* at or near fastest implementation;
* runtime hardware detection.

Disadvantages:

* questions about quality control.

.. include:: links_names.inc
