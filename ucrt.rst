How to correctly link to UCRT (and why it works that way)
=========================================================

The license that Microsoft imposes on the Windows runtime SDK makes it
impossible for the gcc/mingw-w64 developers to use the official CRT
.lib files. They have to recreate everything from scratch, and in
particular, they need to jerry-rig their own mechanism for linking to
ucrtbase.dll.

Traditionally, the way this has worked is that someone grabs the .dll,
dumps out the list of symbols it contains, and then uses that list to
make a new .lib file. And we almost went ahead and started
distributing software linked to ucrtbase.dll like this... except for a
stray mention in a `blog post by Steve Dower
<http://stevedower.id.au/blog/building-for-python-3-5/>`_ suggesting
that because the UCRT uses "API sets", it's actually forbidden to link
to ucrtbase.dll directly and one *must* use the official .lib file, or
else one loses all compatibility guarantees.

I think we can all agree that it would suck if in a few years MS
pushed an update to ucrtbase.dll, and suddenly all mingw-w64-compiled
applications (like: git, R, NumPy, ...) started crashing on
startup. So we want to get this right.

The combination of MS legal terms that mean we can't use the .lib
file, plus MS technical requirements that mean we must use the .lib
file, leaves us in a bit of a bind :-). The simplest way out of this
is to figure out what the *actual* compatibility guarantees are, and
make sure that our jerry-rigged mechanism only uses stable
interfaces. Below is my attempt to figure this out -- and, the good
news is, we sent this text to the folks at MS who maintain the UCRT
and they signed off on it :-).


Background
----------

Maintaining a shared library ABI in the long term is very
difficult. It helps a lot if there's way to provide a new,
incompatible version of a single ABI symbol without breaking old
programs.

For the UCRT to be supportable in the long term, we need to (a) have a
single DLL containing the full CRT, and (b) make it possible for
references to 'malloc' in a program compiled at time 1 to refer to one
symbol in this DLL, while references to 'malloc' in a program compiled
at time 2 refer to a *different* symbol inside this same DLL. So the
traditional symbol lookup key of (DLL name, symbol name) isn't going
to work -- we need (ucrtbase.dll, malloc, time 1) to resolve to a
different symbol than (ucrtbase.dll, malloc, time 2).

Useful comparison: glibc -- which is Linux's equivalent of the UCRT --
faces the identical problem, and they solve it via ELF "symbol
versioning", where symbol lookup actually is extended to act on the
3-tuple (DLL name, symbol name, symbol version), and at build time the
linker takes references like (glibc, malloc) and tags them with
version information. But the PE format doesn't have symbol versions,
so Windows needs a different solution.

Fortunately, it turns out that we can cobble together a solution out
of features that PE has supported for years.


API set implementation
----------------------

The essential idea is that we are going to mangle the symbol version
into the dll name (as contrasted with the ELF approach, which
essentially mangles the symbol version into the symbol name). So
instead of referring to e.g. (ucrtbase.dll, malloc), any binaries we
compile now will contain references to something like
(ucrtbase-v1.dll, malloc); and then in the future if the symbol
changes incompatibly, then we can leave the old implementation
available in ucrtbase-v1.dll and update our SDK so that new binaries
will refer to something like (ucrtbase-v2.dll, malloc); that way both
sets of binaries can keep working indefinitely.

There are two problems we'll have to overcome to make this work,
though. First and most obviously, as written this makes it look like
we've reverted back to the bad old days where there were multiple
incompatible CRT dlls. To avoid that, we want to have multiple DLL
*names*, but we want both versions of malloc to ultimately be
implemented by a single unversioned, updated-in-place ucrtbase.dll. So
under the hood, ucrtbase.dll will have malloc_v1 and malloc_v2, and
then we need to maintain a table of replacements so whenever we load a
binary that refers to (ucrtbase-v1.dll, malloc), then what that binary
will actually get is (ucrtbase.dll, malloc_v1), and similarly
(ucrtbase-v2, malloc) will get resolved to (ucrtbase.dll,
malloc_v2). And this table of replacements of course needs to be
distributed and updated along with ucrtbase.dll. Once all that's in
place, then we'll be free to rearrange ucrtbase.dll any way we like in
the future, so long as we also update the replacement table at the
same time.

Fortunately, it turns out that the PE format already has exactly this
kind of replacement table mechanism: `symbol forwarding
<https://blogs.msdn.microsoft.com/oldnewthing/20060719-24/?p=30473>`_. Normally,
a DLL's export table has a bunch of entries like "if you're looking
for m my 'malloc' symbol, then see offset 0x00abef80 within this
file". But there's also another option: we can create a DLL called
'ucrtbase-v1.dll', whose exports table just has a bunch of entries
saying "if you're looking for 'malloc', then go load ucrtbase.dll
instead and find its symbol called 'malloc_v1'". And then we can
distribute and update this shim DLL alongside ucrtbase.dll.

Second, we have to decide how to lay out our shim DLL or DLLs. In the
above example, we for simplicity assumed that we would have just one
shim DLL for the entire UCRT. In this approach, every time we modify
any symbol, we'd have to create a whole new shim DLL and bump the
version number for all the symbols together. This wouldn't be
disastrous, but might get expensive in the long run: in a large
program we might eventually have to load ucrtbase-v1, ucrtbase-v2,
ucrtbase-v3, ... each of which would contain ~1300 export entries that
need to be read and processed, even though most of them are
identical. At the other extreme, you have the ELF-style approach where
every symbol gets its own independent version number. For ELF this
works fine because the version numbers are simply stored in a table
within the shared object; for us it would be rather expensive, because
we'd have to create a whole new DLL for each and every symbol, which
means that starting even simple programs would require us to locate
and load ~1300 separate DLLs. In practice, the UCRT splits the
difference: following the natural assumption that any change which
breaks 'malloc' is also likely to break 'free' and 'realloc', it
divides the ~1300 symbols into 14 groups by theme ("heap", "stdio",
"time", etc.), and each group gets its own explicitly-versioned shim
DLL with a name like api-ms-win-crt-heap-l1-1-0.dll.  (This also fits
with conventions established for the shim DLLs used for the core
Windows APIs starting in Windows 7 -- though those are implemented via
a different mechanism internally -- and I suppose it might eventually
make it easier to define slimmed-down versions of the CRT that only
provide a subset of these symbols.)

And finally, we need a convenient way for users to link to these
things, because no-one wants to have to explicitly list all of the
api-ms-win-crt-* libraries in a giant linker command line. This turns
out to be easy as well. Conventionally, every shared library foo.dll
has a corresponding foo.lib, and when you want to link your code then
foo.lib is what you actually give to the linker. And conventionally,
foo.lib just contains a bunch of entries that tell the linker: "if you
want symbol XX, then you should find it foo.dll". However, there isn't
actually any rule that all the entries in foo.lib have to refer to
foo.dll -- this is just a convention. ucrt.lib breaks this convention
-- it has an entry for 'malloc' that says it can be found in
api-ms-win-crt-heap-l1-1-0.dll, an entry for 'rename' that says it can
be found in api-ms-win-crt-filesystem-l1-1-0.dll, and so
forth. (Basically ucrt.lib is what you'd get if you generated a
conventional .lib file for each of the shim .dlls, and then
concatenated them together.) The end result is that the user links to
ucrt.lib, and then everything magically gets sorted out. Which is why
none of this is documented and why I had to go work it out from first
principles :-).


Conclusions
-----------

After all that, the conclusion is pretty simple:

- Correctly linking to the UCRT via "API sets" does not require any
  new features in the toolchain; it's just some clever usage of old,
  well-supported features.

- The only "secret sauce" contained in the ucrt.lib file is the list
  of (DLL name, symbol name) pairs; given these, we can
  straightforwardly construct a new .lib file from scratch, and then
  any binaries that we link using our new .lib file will be linked to
  the UCRT in a safe, forwards-compatible manner.

The list of (DLL name, symbol name) pairs for SDK version
10.0.10240.0, x86-32 + x86-64, is available here:
https://gist.github.com/njsmith/08b1e52b65ea90427bfd
