#######
Roadmap
#######

The intent of the MingwPy developers is to expand this website and the MingwPy
project (code to build and distribute the compiler toolchain), and start using
it in the first half of 2016 as the default method to distribute Windows wheel
binaries for Numpy, Scipy and other scientific Python projects.

For more details, please see this proposal (which was accepted by both NumFOCUS
and the PSF in early Jan 2016):

.. toctree::
   :maxdepth: 1

   proposal_december2015

Status update June '16
----------------------

**Progress on milestones in the proposal**

Phase 1 of the proposal has been finished.  The compiler toolchain has now been
re-built by several people based on the public build scripts and documentation.

Phase 2 is about halfway to being finished:

- The MingwPy GitHub project is set up [2]_ and mailing list [3]_ activity is
  picking up.
- Technical design documentation is gradually growing [4]_.
- Numpy and Scipy wheels (both 32-bit and 64-bit) can be built with MingwPy
  [6]_, albeit with a modified numpy.distutils.  Once the numpy.distutils
  patches have been integrated in Numpy master this milestone is done.
- Only continuous integration work still has to start.

One item from phase 3 is almost finished already: MingwPy itself can now be
installed as wheels for Python 2.7, 3.3 and 3.4 from an Anaconda channel [6]_.
Once those wheels have proven themselves in daily use for a while they will be
uploaded on PyPI as well.

**Other developments and observations**

There is also interest in MingwPy from outside the Scientific Python community.
Tony Kelman, who is maintainer of the Windows builds for Julia, has been active
in providing feedback and new ideas.  Julia uses MinGW-w64 for all its Windows
builds, and it looks like there's quite some synergy here (also on OpenBLAS
builds and improvements for example).  Other examples reported on the mailing
list or GitHub were use of MingwPy for building Kivy [10]_, for a Blender
plugin written with C99 variadic macros (which MSVC doesn't support), and for
building NuPIC (machine learning platform from Numenta).

Some topics that people have shown interest in and are being discussed on the
MingwPy issue tracker or mailing list, but are outside the scope of the
currently funded proposal are:

- Cross-compiling from Linux
- Interoperability with other MinGW-w64 builds.
- And most importantly: having UCRT support in MinGW-w64 itself.  This is of
  (moderate) interest to the MinGW-w64 developers themselves, but critical for
  MingwPy because of the move of CPython to MSVC 2015.

The received funding from the PSF and NumFOCUS and the progress that MingwPy
has shown, seems to have also encouraged related activities.  A good example is
PEP 513 [7]_ and Manylinux [8]_, which has enabled Linux wheels on PyPI - this
effort was driven by people from the scientific Python community and several
MingwPy devs made major contributions here.  Another important step forwards is
that Windows wheels for Numpy [9]_ are now on PyPI.  Those are a stopgap
solution based on MSVC until Numpy/Scipy wheels built with MingwPy will be
pushed to PyPI, but have already proven useful to users.


.. [1] https://mingwpy.github.io/proposal_december2015.html
.. [2] https://github.com/mingwpy
.. [3] https://groups.google.com/forum/#!forum/mingwpy
.. [4] http://mingwpy.github.io/
.. [5] https://pypi.python.org/pypi/numpy
.. [6] https://pypi.anaconda.org/carlkl/simple/mingwpy/
.. [7] https://www.python.org/dev/peps/pep-0513/
.. [8] https://github.com/pypa/manylinux
.. [9] https://github.com/numpy/windows-wheel-builder
.. [10] https://kivy.org/docs/installation/installation-windows.html
