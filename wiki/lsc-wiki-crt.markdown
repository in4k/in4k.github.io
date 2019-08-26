---
title: "The C runtime"
layout: "wiki-page"
---

Even though C is a relatively low-level language, some assembly code is needed
to get the process in an acceptable state to the language standard.

First of all, `main` isn't the standard entry point, `_start` is. This function
is provided by your compiler, typically from a file called `crt*.o`. This
function:

1. aligns the stack
2. reads `argc`, `argv`, `environ` and [the auxiliary vector
   ](https://refspecs.linuxfoundation.org/LSB_1.3.0/IA64/spec/auxiliaryvector.html)
   from the stack (see the links to LWN in [Process creation](/explain/proc)
   for stack content details)
3. sets up TLS, locales, and other crap
4. calls `__libc_start_user`, which first calls the constructors of all global
   (C++) variables, then calls `main`, and `exit` automatically as
   well.

Of course, it's huge, but we can provide our own: step 3 can be skipped, 1 and 2
are really small to implement in assembly, and 4 is replaced by calling `main`.
Of course, the second step can be skipped as well when you don't need
args/environ.

However, when external libs need to read from the environment (eg. X11 needs
`$DISPLAY`), you still have to call `__libc_start_user`; it's a portable
interface that asks for `argc`, `argv`, `envp` and `main`. With smol, this
isn't exactly a hassle anymore.

For exiting the program when not using `__libc_start_main`, you can just use a
bare syscall, or `int3`.

### What are all those crtfoo.o-files for?

(Pilfered from [dev.gentoo.org](https://dev.gentoo.org/~vapier/crt.txt) -pcy)

> Some definitions:
> PIC - position independent code (-fPIC)
> PIE - position independent executable (-fPIE -pie)
> crt - C runtime
>
>
>
> crt0.o crt1.o etc...
>   Some systems use crt0.o, while some use crt1.o (and a few even use crt2.o
>   or higher).  Most likely due to a transitionary phase that some targets
>   went through.  The specific number is otherwise entirely arbitrary -- look
>   at the internal gcc port code to figure out what your target expects.  All
>   that matters is that whatever gcc has encoded, your C library better use
>   the same name.
>
>   This object is expected to contain the _start symbol which takes care of
>   bootstrapping the initial execution of the program.  What exactly that
>   entails is highly libc dependent and as such, the object is provided by
>   the C library and cannot be mixed with other ones.
>
>   On uClibc/glibc systems, this object initializes very early ABI requirements
>   (like the stack or frame pointer), setting up the argc/argv/env values, and
>   then passing pointers to the init/fini/main funcs to the internal libc main
>   which in turn does more general bootstrapping before finally calling the real
>   main function.
>
>   glibc ports call this file 'start.S' while uClibc ports call this crt0.S or
>   crt1.S (depending on what their gcc expects).
>
> crti.o
>   Defines the function prologs for the .init and .fini sections (with the _init
>   and _fini symbols respectively).  This way they can be called directly.  These
>   symbols also trigger the linker to generate DT_INIT/DT_FINI dynamic ELF tags.
>
>   These are to support the old style constructor/destructor system where all
>   .init/.fini sections get concatenated at link time.  Not to be confused with
>   newer prioritized constructor/destructor .init_array/.fini_array sections and
>   DT_INIT_ARRAY/DT_FINI_ARRAY ELF tags.
>
>   glibc ports used to call this 'initfini.c', but now use 'crti.S'.  uClibc
>   also uses 'crti.S'.
>
> crtn.o
>   Defines the function epilogs for the .init/.fini sections.  See crti.o.
>
>   glibc ports used to call this 'initfini.c', but now use 'crtn.S'.  uClibc
>   also uses 'crtn.S'.
>
> Scrt1.o
>   Used in place of crt1.o when generating PIEs.
> gcrt1.o
>   Used in place of crt1.o when generating code with profiling information.
>   Compile with -pg.  Produces output suitable for the gprof util.
> Mcrt1.o
>   Like gcrt1.o, but is used with the prof utility.  glibc installs this as
>   a dummy file as it's useless on linux systems.
>
> crtbegin.o
>   GCC uses this to find the start of the constructors.
> crtbeginS.o
>   Used in place of crtbegin.o when generating shared objects/PIEs.
> crtbeginT.o
>   Used in place of crtbegin.o when generating static executables.
> crtend.o
>   GCC uses this to find the start of the destructors.
> crtendS.o
>   Used in place of crtend.o when generating shared objects/PIEs.
>
>
>
> General linking order:
> crt1.o crti.o crtbegin.o [-L paths] [user objects] [gcc libs] [C libs] [gcc libs] crtend.o crtn.o
>
>
>
> More references:
>     http://gcc.gnu.org/onlinedocs/gccint/Initialization.html
