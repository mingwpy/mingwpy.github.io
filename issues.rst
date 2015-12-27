######
Issues
######

******************************************
Finding the mingw-w64 runtime library code
******************************************

It is difficult to guarantee that a compiled Python extension will be able to
find the correct version of runtime DLLs |--| see: `Windows DLL notes`_.

One method to avoid this problem is to include any necessary runtime code in
the compiled extension.  This can be done by:

* Compiling all extensions (DLLs) with ``-static-libgcc``,
  ``-static-libstdc++`` etc flags;
* Using a gcc compiler toolchain that does static linking to runtimes by
  default.  This is a "static toolchain".  The `mingw-builds`_ set of scripts
  builds such a toolchain by passing the ``--static-gcc`` argument (see:
  `mingw-build build file
  <https://github.com/niXman/mingw-builds/blob/17ae841dcf6e72badad7941a06d631edaf687436/build#L218>`_;

Using these approaches, all the necessary object code from the GCC runtimes
will be statically linked into the binaries. As a consequence the binary size
will be increased in comparison to the standard toolchains / dynamic linking.
The advantage is, that there will be no dependency to external GCC runtime
libraries, so the deployment of Python extensions is greatly improved.

However, exception heavy C++ programs such as QT should be compiled with
shared runtimes to avoid problems with exception handling over DLL boundaries.

For building typically Python extensions a customized static GCC toolchain is
the best compromise IMHO.

**Note from njs:** This definitely sounds like the right thing to try
initially, *but* it's possible (likely?) that we'll run into the `same
problem that Steve Dower ran into with trying to set up MSVC to
statically link the runtime code
<http://stevedower.id.au/blog/building-for-python-3-5-part-two/>`_,
i.e., TLS keys are a limited resource, so if libgcc allocates any TLS
keys at startup, and every extension module has its own copy of
libgcc, then this puts a hard cap on how many Python extension modules
can be loaded into a single process. Which is really bad (consider
projects like Sage that make heavy use of Cython, and each Cython
source file creates a separate extension...). Or at least, it's really
bad IF it's a problem at all. In fact I have no idea whether libgcc
allocates any TLS keys at startup -- introspecting a handy copy of
libgcc*.dll shows that it does contain references to ``TlsAlloc``, but
I don't know whether they're called under ordinary circumstances. So
we should start with static linking, but have a fallback plan in case
it doesn't work.

Worst case, I think our fallback plan is to switch back to dynamic
linking, and require projects to use one of the standard dynamic
linking strategies that we've discussed elsewhere. Possibly this means
the mingwpy project will eventually want to start shipping a
mingw-w64-runtime wheel, similar to what we've discussed doing with
BLAS dlls, that projects could declare a dependency on.

******************************
MS / gcc ABI incompatibilities
******************************

There is a win32 default stack alignment incompatibility: GCC code provides
(and assumes) 16 (2^4) byte stack alignment since GCC4.6, but MSVC uses 4
(2^2) byte stack alignment.

Win64 X86_64 is not affected.  This issue is the major cause for segment
faults on 32bit systems.

The solution is to use the ``-mincoming-stack-boundary=2`` flag for compiling.

From:
https://docs.google.com/document/d/1lnWj0UhxJkeK0WyQdoW2opQKJfIseaN0qimFZbFOOYs

    Stack alignment discussion: GCC upstream agrees the default should be
    compatible with MSVC, however it’s not. GCC will likely not accept a patch
    to go back to the pre-4.6 stack alignment. Needs follow-up with Kai Tietz
    (he asked for a test case). Note: 32-bit only problem.

Some googling suggests that the case where this problem will arise specifically
is when you combine functions that contain SSE instructions (which are less
forgiving of bad alignment than most of the x86 ISA), then called from code
that doesn't guarantee the right stack alignment -- online hints suggest that
this tends to happen with windows threads. This is consistent with the
observation that OpenBLAS in particular blows up without this flag (b/c
OpenBLAS is a heavy user of SSE + threads).

(See: `1 <http://www.peterstock.co.uk/games/mingw_sse/>`_, `2 <http://www.sourceware.org/ml/pthreads-win32/2008/msg00056.html>`_)

It's likely that a sufficient test case to give to Kai would be a
program that spawns a bunch of threads running the following
function::

  DWORD WINAPI thread(LPVOID param) {
      int x;
      intptr_t address = (intptr_t) &x;
      if (address % 16 != 0) {
          printf("Thread stack %p is not 16-byte aligned!\n", &x);
      }
      sleep(10);
  }

and if this prints any output then we have our proof that
``-mincoming-stack-boundary=2`` is necessary for Windows/MSVC
compatibility.

**********************
Choice of MSVC runtime
**********************

All code in a single process should use one single version of the MSVC runtime
(see `MSDN article
<https://msdn.microsoft.com/en-us/library/ms235460.aspx>`_).

The Python that gets installed from downloading from https://python.org is
build with MSVC.  Therefore Python extensions for Python installed from these
installers must use the same MSVC CRT.

Specifically (see: `Python and MSVC versions
<https://matthew-brett.github.io/pydagogue/python_msvc.html#visual-studio-versions-used-to-compile-distributed-python-binaries>`_):

============== ============ =========
Python version VC++ version C runtime
============== ============ =========
2.7.6          9.0 / 2008   MSVCR90.DLL
3.2.3          9.0 / 2008   MSVCR90.DLL
3.3.5          10.0 / 2010  MSVCR100.DLL
3.4.0          10.0 / 2010  MSVCR100.DLL
3.5.0          14.0 / 2015  UCRTBASE.DLL / VCRUNTIME140.DLL
============== ============ =========

By default, mingw-w64 links to the MSVCRT.DLL.  This is a CRT dating from MSVC
4.2, but updated to contain the runtimes for MSVC 6.0, plus some more recent
(>6.0) API calls (see these comments on `using MSVCRT.DLL from Mingw-w64
<http://sourceforge.net/p/mingw-w64/wiki2/The%20case%20against%20msvcrt.dll>`_).

It is possible, using `spec files
<http://www.mingw.org/wiki/HOWTO_Use_the_GCC_specs_file>`_, to ask mingw-w64
to link to other versions of the MSVC runtimes.  This could induce bad
behavior if there is any API mismatch between the implementations in the
mingw-w64 headers (tuned to MSVCRT.DLL) and those in the newer C runtime.

What workarounds are necessary to use MSVC 9.0 / 2008 (Python 2.7)?

************************
The VS 14 / 2015 runtime
************************

Matters get more confusing for the latest (at time of writing) MSVC, version
14 (MSVS 2015).

Firstly, the CRT is now two files:

* ``ucrtbase.dll``;
* ``vcruntime140.dll``;

of which the first will be kept with a backwards-compatible API / ABI across
new VS releases.

Second, linking correctly to these new 2015 libraries requires careful choice
of the DLL import library.

(Quoting Nathaniel Smith on email):

    The good news though is that I think we actually do know how to do this --
    basically it's just, instead of linking directly to ucrtbase.dll, there
    are 15 "interface dlls" that export the various CRT symbols, so one has to
    link to the needed ones directly. (This is based on my fiddling around
    with the UCRT SDK, and Kai has apparently come to a similar conclusion.)
    So it'd be good to ask MS to confirm, but I'm ~90% sure this is right.

    Symbols and DLLs:
        https://gist.github.com/njsmith/08b1e52b65ea90427bfd

99% of the things we care about are in the safe unversioned
``ucrtbase.dll``. There remains some question of what to do with
``vcruntime140.dll``. The ideal would be that our toolchain simply
doesn't link to it at all. It looks like the symbols it provides are::

  [Ordinal/Name Pointer] Table
        [   0] _CreateFrameInfo
        [   1] _CxxThrowException
        [   2] _FindAndUnlinkFrame
        [   3] _IsExceptionObjectToBeDestroyed
        [   4] _SetWinRTOutOfMemoryExceptionCallback
        [   5] __AdjustPointer
        [   6] __BuildCatchObject
        [   7] __BuildCatchObjectHelper
        [   8] __C_specific_handler
        [   9] __C_specific_handler_noexcept
        [  10] __CxxDetectRethrow
        [  11] __CxxExceptionFilter
        [  12] __CxxFrameHandler
        [  13] __CxxFrameHandler2
        [  14] __CxxFrameHandler3
        [  15] __CxxQueryExceptionSize
        [  16] __CxxRegisterExceptionObject
        [  17] __CxxUnregisterExceptionObject
        [  18] __DestructExceptionObject
        [  19] __FrameUnwindFilter
        [  20] __GetPlatformExceptionInfo
        [  21] __NLG_Dispatch2
        [  22] __NLG_Return2
        [  23] __RTCastToVoid
        [  24] __RTDynamicCast
        [  25] __RTtypeid
        [  26] __TypeMatch
        [  27] __current_exception
        [  28] __current_exception_context
        [  29] __intrinsic_setjmp
        [  30] __intrinsic_setjmpex
        [  31] __processing_throw
        [  32] __report_gsfailure
        [  33] __std_exception_copy
        [  34] __std_exception_destroy
        [  35] __std_terminate
        [  36] __std_type_info_compare
        [  37] __std_type_info_destroy_list
        [  38] __std_type_info_hash
        [  39] __std_type_info_name
        [  40] __telemetry_main_invoke_trigger
        [  41] __telemetry_main_return_trigger
        [  42] __unDName
        [  43] __unDNameEx
        [  44] __uncaught_exception
        [  45] __uncaught_exceptions
        [  46] __vcrt_GetModuleFileNameW
        [  47] __vcrt_GetModuleHandleW
        [  48] __vcrt_InitializeCriticalSectionEx
        [  49] __vcrt_LoadLibraryExW
        [  50] _get_purecall_handler
        [  51] _get_unexpected
        [  52] _is_exception_typeof
        [  53] _local_unwind
        [  54] _purecall
        [  55] _set_purecall_handler
        [  56] _set_se_translator
        [  57] longjmp
        [  58] memchr
        [  59] memcmp
        [  60] memcpy
        [  61] memmove
        [  62] memset
        [  63] set_unexpected
        [  64] strchr
        [  65] strrchr
        [  66] strstr
        [  67] unexpected
        [  68] wcschr
        [  69] wcsrchr
        [  70] wcsstr

So, numbers 0 through 56 appear to me (= njs) to be internal machinery
used for C++ exceptions and `RTTI
<https://en.wikipedia.org/wiki/Run-time_type_information>`_. I'm
pretty sure this stuff is all irrelevant to us, because libgcc should
already be providing the equivalent machinery needed for any C++ code
that's compiled using gcc, and trying to achieve C++ ABI compatibility
between MSVC and gcc is outside the scope of this project.

Of the rest, ``set_unexpected`` and ``unexpected`` can be ignored,
because they're yet more C++-related nonsense that `apparently isn't
even used anymore
<https://msdn.microsoft.com/en-us/library/h46t5b69.aspx>`_), but the
others are real standard C functions, and a quick check reveals that
most of them are *not* available in ``ucrtbase.dll``, but only here in
``vcruntime140.dll``.

These can be broken down further into two categories:

1) Simple string functions: ``memchr``, ``memcmp``, ``memcpy``,
   ``memmove``, ``memset``, ``strchr``, ``strrchr``, ``strstr``,
   ``wcschr``, ``wcsrchr``, ``wcsstr``. These are crucial functions
   that we definitely need to support, but there's no particular
   advantage to using the implementation from
   ``vcruntime140.dll``. They can just be reimplemented inside
   mingw-w64. (Basic working versions are trivial; making them fast
   takes more work, but could be lifted from BSD libc or whatever.)

2) ``setjmp`` / ``longjmp``: Reimplementing these from scratch would
   be a huuuuge hassle. Fortunately, they're a very advanced and
   tricky feature that's rarely useful! ...But unfortunately, both
   numpy and scipy actually use them. Numpy's usage might be optional
   (it has fallback code for if they're not available, and the only
   cost would be that you couldn't hit control-C to interrupt a
   long-running inner-loop -- and this may not even work on Windows
   in the first place), but scipy uses them for actual flow control
   D-:. It's possible this could just be fixed in scipy if necessary.

   setjmp/longjmp may also be needed for exception handling to work in
   32-bit mingw-w64-compiled C++ code. Though I'm actually a bit
   confused on this point, since the copy of ``libgcc_s_sjlj-1.dll``
   that I have on my Debian box doesn't actually seem to contain any
   references to the ``setjmp`` or ``longjmp`` symbols in the CRT?

   On further investigation it looks like *maybe* the reason libgcc
   does not import setjmp/longjmp from the CRT is that it is built to
   use the gcc builtins ``__builtin_setjmp`` / ``__builtin_longjmp``
   instead? If these exist and are functional in mingw-w64 builds then
   that is an even better solution -- we just need to set up the
   headers so that user code that tries to call setjmp/longjmp are
   redirected to the builtins. Maybe. Need to check with mingw-w64
   folks about this.

*********************
Math precision issues
*********************

(Quoting Nathaniel Smith on email):

    mingw-w64's libm implementations are the borrowed from those used on BSDs,
    Linux, etc., and assume -- consistent with the ABI on those other
    platforms -- that the x87 FPU will be configured to use 80 bit precision
    for intermediate results. MSVC's ABI, though, configures the x87 FPU into
    64 bit precision mode, and we don't want to override that because who
    knows what would break.  The result is that when run in MSVC-compatibility
    mode, mingw-w64's libm code currently assumes that it has higher internal
    precision than it actually has, and doesn't necessarily produce the right
    answers. In particular the trig functions currently are just thin wrappers
    around the x87 fsin / fcos / etc. instructions. These are pretty sloppy
    and inaccurate, which doesn't matter too much if you're going to throw
    away all the low-order bits anyway... but if you're going to keep those
    bits then it becomes more of an issue.

    [...]

Investigating ``sleef`` library for needed functions.  Nathaniel suggested
libm implementation from bionic (Android's libc).

Bionic's libm is here: https://github.com/android/platform_bionic/tree/master/libm/x86
Basically what would be needed is just copying these files into mingw-w64 plus
build system updates.

***************
Disutils issues
***************

Mingw via Python distutils needs to link extension code to the correct Python
library, e.g. ``libpython27.dll``.  To do this, mingw needs a library
definition file ``libpython27.a``.  It is possible to create this file using
the mingw-w64 tools.

*Query from njs: can't we just use the python27.lib that's shipped
with CPython itself?*

numpy distutils currently needs patching to pass correct flags to compiler /
linker.

How to return correct flags to mingw-w64, from Python built with MSVC?

******************
Manifest resources
******************

Solution is to extend the GCC toolchain with the Manifest resource files and
ensure linkage with the help of the 'specs' file.

*If this is the solution, then what's the problem? Carl, could you add
some text here elaborating? (and then delete this comment :-)) -njs*

***********************
BLAS / LAPACK libraries
***********************

There is no silver bullet for the problem of finding fast, reliable BLAS and
LAPACK routines with a suitable license.

See :doc:`blas_lapack`.

******************
Partial to-do list
******************

* Implement linking to MSVC 2015 CRT libraries from mingw-w64;

  * work out correct method;
  * discuss with MS developers;
  * merge to mingw-w64;

* Implement high-precision libm;

  * decide on library;
  * merge to mingw-w64;

* Develop "runtime agility" for mingw-w64 - method of adapting to different
  MSVC CRT libraries dynamically.  Maybe 50-60K of developer work / 2 man
  months.  On back-burner for mingw-w64 project.

* Create buildbot / appveyor scripts to build mingwpy library;

* Create test rig for OpenBLAS maybe via numpy, implement on buildbots with
  different processors or gcc compile farm.

.. include:: links_names.inc
