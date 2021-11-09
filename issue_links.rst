##################################
Links for issues in Mingw64 builds
##################################

***************************
Slow math routines and bugs
***************************

https://sourceforge.net/p/mingw-w64/mailman/message/37360036/

[Mingw-w64-public] Patches to fix #515 and 916

https://sourceforge.net/p/mingw-w64/bugs/515/

#515 Incorrect sign output from asinh

https://sourceforge.net/p/mingw-w64/bugs/916/

#916 asinh incorrect for large values

https://sourceforge.net/p/mingw-w64/mailman/message/37311362/

2021-06-28 : [Mingw-w64-public] slow math routines and ucrt

https://sourceforge.net/p/mingw-w64/discussion/723797/thread/01b93e82/

2015-03-12 : [Mingw-w64-public] Exp implementation slow on x86_64

https://sourceforge.net/p/mingw-w64/mailman/message/37299080/

2021-06-09 [Mingw-w64-public] bug in sin(pi)

https://sourceforge.net/p/mingw-w64/bugs/904/

2021-06-16 : #904 Bug in sin(3.1415926535897931)

****************************
Thread and precision problem
****************************

https://sourceforge.net/p/mingw-w64/mailman/message/34896011/

2016-03-17 : [Mingw-w64-public] Floating-Point Operations Not Deterministic
When Excecuted Asynchronously

Also see:

https://sourceforge.net/p/mingw-w64/mailman/message/34913248/

::

    From: Carl Kleffner <cmkleffner@gm...> - 2016-03-07 11:51:17

    Just to be sure:

    _fpreset() maps to either
    __asm__ ("fninit")   // FPU  extended precision
    or
    (* __MINGW_IMP_SYMBOL(_fpreset))()  // FPU double precision

    Is this correct? depending on wether CRT_fp10.o is linked in (default for
    upstream mingw-w64) or CRT_fp8.o. My best guess is, that using mingw-w64 in
    conjunction with OpenCASCADE should link CRT_fp8.o *if* OpenCASCADE was
    compiled with MSVC.


https://stackoverflow.com/questions/67382785/why-does-mingw-w64-floating-point-precision-depend-on-winpthreads-version

https://sourceforge.net/p/mingw-w64/mailman/message/37274995/

(linking against `CRT_fp8.o` no longer works for generated threads).

::

    2021-05-04 : [Mingw-w64-public] Setting Floating-Point Operation Precision Changes With Gcc 10.3

    Liu Hao Liu Hao 2020-08-17
    winpthreads: Call `_fpreset()` for new threads This is necessary for math
    functions taking `long double` arguments to work reliably, as by default,
    new threads have their x87 precision set to 53 bits.

    Note the x87 control word can't be set in the TLS callback. It gets
    overwritten before the thread procedure begins execution. There is NO
    workaround for the win32 thread model, other than requiring users to call
    `_fpreset()` themselves.

    Reference: https://sourceforge.net/p/mingw-w64/mailman/message/37084134/

Also see: https://sourceforge.net/p/mingw-w64/mingw-w64/ci/295fafc

https://sourceforge.net/p/mingw-w64/mailman/message/33244358/

2015-01-15 : [Mingw-w64-public] gfortran, open mp and floating point precision

Also see:

https://sourceforge.net/p/mingw-w64/mailman/message/33332276/

::

    Re: [Mingw-w64-public] gfortran, open mp and floating point precision
    From: Carl Kleffner <cmkleffner@gm...> - 2015-02-02 22:29:27
    Attachments: Message as HTML

    The Windows ABI - see
    https://msdn.microsoft.com/en-us/library/ms235286.aspx - differes from the
    System V ABI - see http://www.x86-64.org/documentation/abi.pdf.

    MingW-W64 sets the x87 FPU control word register to extended precision.
    The 'standard' for Windows however is 'double precision'.  See
    https://msdn.microsoft.com/en-us/library/ms235300.aspx.

    On Windows newly started threads never inherit FPU register settings from
    the master thread and set the FPU registers to the standard value. Thus
    you can have different FPU settings in different threads. OpenBLAS i.e.
    has some extra code to propagate the FPU settings. Look for
    'CONSISTENT_FPCSR' at https://github.com/xianyi/OpenBLAS.

    Using extended precision with GCC can be tricky:
    https://gcc.gnu.org/wiki/FloatingPointMath and
    https://gcc.gnu.org/wiki/x87note. The latter link describes some gotchas
    when using extended precision with GCC.

    Cheers,

    Carl


https://sourceforge.net/p/mingw-w64/mailman/message/33032222/

::

    Carl Kleffner 2014-11-12

    [Mingw-w64-public] FPU control word on startup

    Hi,

    during tests with my mingw builds variant mingw-w64-for-python
    <https://bitbucket.org/carlkl/mingw-w64-for-python/downloads>; I stumpled
    upon a flaw in mingw-w64 FPU handling on startup. This is issued at
    mingw-w64 builds of numpy <https://github.com/numpy/numpy/issues/5194>;
    and test failures when building win32 binaries with mingw based on gcc 4.8
    <https://github.com/numpy/numpy/issues/4909>;. The reason for this
    behaviour: CRT_fp10.o (part of libmingw32.a) sets the FPU control word to
    extended precision on startup as per default. This can be overwritten by
    additionally linking CRT_fp8.o, but this is documented elsewhere
    (floath.h). As far I can see, this behaviour in mingw-w64 inherits from
    the Mingw32 codebase, but I'm not sure.

    There seems to be a consense, that 027f should be the standard for setting
    the FPU control word. Otherwise it gets into your way, see i.e.  Random and
    unexpected EXCEPTION_FLT_DIVIDE_BY_ZERO and EXCEPTION_FLT_INVALID__OPERATION
    <http://blogs.msdn.com/b/dougste/archive/2008/11/12/random-and-unexpected-exception-flt-divide-by-zero-and-exception-flt-invalid-operation.aspx>;

    Is there some code in mingw.w64 that depends on extended precision as a
    default value? Otherwise this should be fixed in the mingw-w64 codebase
    IMHO.

Also see:

https://sourceforge.net/p/mingw-w64/mailman/message/33071871/

::

    Re: [Mingw-w64-public] FPU control word on startup
    From: Carl Kleffner <cmkleffner@gm...> - 2014-11-23 16:03:14
    Attachments: Message as HTML

    What I'm talking about ist the following:

    CRT_fp10.o on startup executes FNINIT. According to
    http://www.intel.com/Assets/ja_JP/PDF/manual/253665.pdf:

    "When the x87 FPU is initialized with either an FINIT/FNINIT or
    FSAVE/FNSAVE instruction, the x87 FPU control word is set to 037FH, which
    masks all floating-point exceptions, sets rounding to nearest, and sets the
    x87 FPU precision to 64 bits."

    That is 80-bit double extended computation precision as default and
    definitly not the expected double computation precision. See also:

    http://sourceforge.net/p/mingw/bugs/904/
    https://ghc.haskell.org/trac/ghc/ticket/7289


https://sourceforge.net/p/mingw-w64/mailman/message/33071954/

And::

    Re: [Mingw-w64-public] FPU control word on startup
    From: Yaron Keren <yaron.keren@gm...> - 2014-11-23 16:40:32
    Attachments: Message as HTML

    It is a real incompatibility between VC and mingw. Visual C++ dropped
    support for 80 bit long doubles on x87 many years ago, in VC long doubles
    are 64 bit, which is why they use 0x27F. mingw still supports 80 long
    doubles on x87. As long this is the case, it is incosistent for mingw to
    set computation accuracy to 0x27f.

    The 'long double' size incompatibility requires mingw to implement various
    long-double functions including printf and scanf, independently of MSVCRT.

    So the decision is whether long doubles should be 64 or 80 bits. The
    control word is simply the result of this decision.

    BTW. see Kahan about long doubles (in Java, though),
    http://www.cs.berkeley.edu/~wkahan/JAVAhurt.pdf

And:

https://sourceforge.net/p/mingw-w64/mailman/message/33076023/

::

    Re: [Mingw-w64-public] FPU control word on startup
    From: Kai Tietz <ktietz70@go...> - 2014-11-24 20:30:10

    Hello,

    so from my POV decision is already made.  80-bit support is wanted, and
    needs to be active for being compatible with C99-math and testsuites and
    gcc for Windows.  This support for 64-bit goes back to 2005, and for
    32-bit even much earlier (1998 ?). We provide a way for users asking
    explicit for 64-bit precision by different object-files, as already noted
    in this thread (CRT_fp8, CRT_fp10,...).  This is the same as for txtmode.o
    vs. binmode.o, or CRT_noglob.o vs. CRT_glob.o changing behavior.

    Nevertheless I would welcome if someone would contribute a good page on
    our Wiki FAQ-section.

    Regards, Kai

*****************
AVX Win64 problem
*****************

https://sourceforge.net/p/mingw-w64/discussion/723797/thread/bc936130/

2018-04-07 : AVX / AVX2 Support on Windows x64 with MinGW64

Also see:

https://sourceforge.net/p/mingw-w64/mailman/message/34453505/

    Re: [Mingw-w64-public] AVX support is broken in 64-bit mode! Will there ever be a fix?
    From: Kai Tietz <ktietz70@go...> - 2015-09-12 10:39:57

    first thanks for interesting in this subject, but sadly I have disappoint
    you about it, I fear.  The cause for the inability to align to 32-byte
    alignment is neither to seek in gcc, nor in mingw-w64 itself.  It is a
    consequence of the SEH-information required by the x64 ABI (see here as
    reference either msdn, or ibm's IA64 exception-specification).  Sadly
    there is no way to express by it any kind of stack-alignment.  So it is
    impossible to write code, which brings AVX to proper stack-location within
    x64-ABI.  The ABI itself makes sure that on function-entry stack has a
    16-byte alignment, so 64-bit sse-registers can be stored aligned.  Another
    weakness in this prologue-description used by x64 is that register with
    more then 64-bit width can't be expressed.   This is a reason why the
    upper 8 bytes of a ymm-register are treated as volatile.

    Only way to work-a-round that is to do manual alignment of function,
    and making sure that you won't try to unwind over such a function.
    The mentioned prologue/epilogue script on stack-overflow article seems
    to do exactly this.

    Regards, Kai Tietz

https://sourceforge.net/p/mingw-w64/mailman/message/34486514/

    2015-09-24

    "Intel® Architecture Instruction Set Extensions Programming Reference"
    lists the SIMD instructions with explicit 32 byte alignment requirements:
    *Table 2-6. SIMD Instructions Requiring Explicitly Aligned MemoryRequire
    32-byte alignmentVMOVDQA   ymm, m256VMOVDQA   m256, ymmVMOVAPS   ymm,
    m256VMOVAPS   m256, ymmVMOVAPD   ymm, m256VMOVAPD   m256, ymmVMOVNTPS
    m256, ymmVMOVNTPD  m256, ymmVMOVNTDQ  m256, ymmVMOVNTDQA ymm, m25*

    See:
    https://software.intel.com/sites/default/files/m/d/4/1/d/8/Intro_to_Intel_AVX.pdf

    and:
    https://software.intel.com/en-us/intel-architecture-instruction-set-extensions-programming-reference
    (p. 2-10 Ref. # 319433-023)

    a wild guess: MSVC avoids these instructions ?

Follow up:

https://sourceforge.net/p/mingw-w64/mailman/message/34486922/

    From: Carl Kleffner <cmkleffner@gm...> - 2015-09-24 12:26:05

    Another link:
    https://connect.microsoft.com/VisualStudio/feedback/details/805738/visual-studio-2013-c-compiler-considers-unaligned-avx-data


https://sourceforge.net/p/mingw-w64/mailman/message/34453497/

2015-09-11 : [Mingw-w64-public] AVX support is broken in 64-bit mode! Will
there ever be a fix?

*************
Miscellaneous
*************

https://sourceforge.net/p/mingw-w64/mailman/message/32234485/

::

    2014-04-16 : [Mingw-w64-public] Bug fix for expm1 - how to proceed?

    ...

    When testing against the Python numpy library, we found a failing test
    for the mingw-w64 expm1 routine.  Specifically, we found that:

    double i = -0.0;
    return expm1(i);

    returns 0 instead of -0, as I believe the standard requires:

Not clear whether the above patch got merged.

https://sourceforge.net/p/mingw-w64/mailman/message/34874977/

2016-02-23 ::

    Re: [Mingw-w64-public] Any more commits before v5.x branches out? 

    I would like to add a patch for math/fpclassify.c

    ...

    Due to the bug https://sourceforge.net/p/mingw-w64/bugs/367   __fpclassify
    (double _x) never worked for doubles on 32 bit (__i386__  _X86_). It does
    work using the 64 bit path in the if clause at least for me. So this patch
    simply removes the differentiation between 32 bit and 64 bit.

Patch ran into trouble (see thread), not clear whether it got merged.

https://sourceforge.net/p/mingw-w64/discussion/723797/thread/43226a1d/

::

    Francesc Altet - 2010-03-25
    Crash with MingW-w64 and NumPy

    Hi,
    I have succeeded compiling the NumPy computational package for Python
    using mingw-w64.  However, when running the test suite, I consistently get
    a crash.  Here it is the backtrace using gdb

Later in that thread:

::

    So, it seems that it is a problem with char I/O.  Also, I've run into problems in many Unicode tests.  So I think the problem is more related with char/unicode code.
