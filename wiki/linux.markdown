---
title: "Linux"
layout: "wiki-page"
---

# Introduction to Linux as 4 KB Platform

Linux is at advantage by the fact that most standard Linux distributions ship a
lot more libraries than Win32 which you can assume being available in the system.
[Here](http://releases.ubuntu.com/19.04/ubuntu-19.04-desktop-amd64.manifest) is
the list of packages that come by default with Ubuntu 19.04.
Some of the useful ones are e.g. SDL which you can use as the base framework
for the application giving you graphics output and event handling. SDL also
provides sound output capabilities. Not all parties have SDL installed though,
as it doesn't come by default with Ubuntu! Also, usually, only the 64-bit
versions of the libraries are installed.

More resources:

* [PoroCYon's Linux Sizecoding wiki mirror](https://pcy.be/lsc-wiki.html)
  ([original](https://linux.weeaboo.software/) (down?), [local copy](lsc-wiki))
* [dnload documentation](https://github.com/faemiyah/dnload/)
* [source code of 작은 (small) by Limp Ninja & K2 & TiTAN
  ](https://gitlab.com/PoroCYon/linux-4k-intro-template), should be quite
  usable as an intro template.
* [source code of Nutrition Facts by Suricrasia Online](https://bitbucket.org/blackle_mori/nutrition-facts), for executable graphics
* [Kickers of ELF](http://www.muppetlabs.com/~breadbox/software/elfkickers.html)
  ([local copy](/html_articles/ELF Kickers.html)): A collection of useful tools.
* [A Whirlwind Tutorial on Creating Really Teensy ELF Executables for Linux](http://www.muppetlabs.com/~breadbox/software/tiny/teensy.html)
  ([local copy](/html_articles/A Whirlwind Tutorial on Creating Really Teensy ELF Executables for Linux.html)):
  introduction on messing with ELF headers and Linux syscalls. *The* article to
  start with if you want to do some ELF hacking.
* [HOWTO: 4k intros in GNU/Linux](http://twiren.kapsi.fi/linux4k.html), a short tutorial by Timo Wiren (glaze/biomassa^wAMMA)
* [Linux 4k Intro Coding Presentation Slides](http://ftp.kameli.net/pub/fit/misc/presis_asm06.pdf)
  from Assembly'2006, by Markku "Marq" Reunanen ([local copy](https://files.scene.org/view/resources/in4k/linux_4k_intro_coding_asm06.pdf)):
  contains some useful info, but is very outdated.

# Using static executables

[This](linux-static) page has a collection of tricks for how to
make statically linked Linux intros, plus some more niche tricks. However, as
GPU drivers require dynamic linking (they live in userspace), this info is less
relevant for more usual productions, and thus has been moved to a separate page
to have this page be less scatterbrained.

Similarly, the sizecoding.org wiki has [a page](http://www.sizecoding.org/wiki/Linux)
on how to write statically-linked Linux intros, typically for &lt;1k.

# Fighting against the C compiler

I'm assuming GCC or Clang here.

## Compiler flags etc

First of all, you want a few obvious (and less obvious) optimization plugins:

```
Optimization:
 -flto -fuse-linker-plugin -ffast-math
 -O2/-Os (always try both, -O2 sometimes generates smaller code than -Os, but
          the former also aligns branch targets to 16 bytes...)
 -march=core2 <--- usually seems to generate smaller code? not in all cases?
You might also want to disable all SSE/MMX/AVX stuff, as mixing these with
'regular' x86 will make the code less compressable. (Unless those instructions
are concentrated in one big block.) However, compilers like to use those for
non-'SIMD-purposes' very much, such as copying structs around.

Avoid ELF and ABI bloat:
 -fno-plt -fno-stack-protector -fno-stack-check -fno-unwind-tables
 -fno-asynchronous-unwind-tables -fomit-frame-pointer -fno-pic -fno-PIE -fno-PIC
 -ffunction-sections -fdata-sections
The last two switches causes every function/variable to be placed in its own
section, after which the linker can remove unused ones using the --gc-sections
flag.

Avoiding linker bloat:
 -Wl,--build-id=none -z norelro -wl,-z,noseparate-code -Wl,--no-eh-frame-hdr
 -no-pie -Wl,--no-ld-generated-unwind-info -Wl,--hash-style=sysv
 -Wl,-z,nodynamic-undefined-weak
 -Wl,--gc-sections <--- remove all unused functions/global variables/sections
Throw away C++ crap:
 -fno-exceptions -fno-rtti -fno-threadsafe-statics
```

A second trick, useful for dealing with custom linkers while still reaping
the benefits of some of the flags of GNU ld, is *incremental linking*: linking
multiple `.o` files into a new one that can be linked into an executable. That
way, you can still use things like LTO (which eg. smol (see below) isn't able
to deal with). You use it by passing `-Wl,-i` to GCC. For GCC 9 and later, you
also have to pass in `-flinker-output=nolto-rel` in order to actually apply the
LTO and output a normal object file.

## Code tricks

Instead of importing functions that implement syscalls (like `open`) or ones
that do basic memory or math stuff (like `memcpy` and `sin`), it's a better
idea to implement these yourself using inline assembly. For example, a `memcpy`
call can be turned into a single `rep stosb` instruction using the following
snippet:

```c
inline __attribute__((__force_inline__)) // make gcc/clang ALWAYS inline this function
static void ASM_memcpy(void* dst, const void* src, size_t size) {
	asm volatile /* volatile prevents reordering of the asm code between other
	                C code etc. */ (
		"rep stosb\n" // the asm code
		: // output variables (none), in the format "=format"(C_varname)[ASM_varname]
		:"S"(src),"D"(dst),"c"(size) // input variables, esi/rsi == src, edi/rdi == dst, ecx/rcx == size
		: // other, destroyed registers (none)
	);
}
```

A collection of a nubmer of syscalls implemented this way can be found
[here](https://bitbucket.org/blackle_mori/scalemark/src/master/sys.h)
([+ dependency](https://bitbucket.org/blackle_mori/scalemark/src/master/def.h)).

Other tricks for getting the code size down have been discussed at lenght in
other articles in this wiki.

## Removing more junk that comes with the compiler output

First of all, there's the *startup code*. Basically, the `main` function you
wrote isn't the *real* first thing that gets executed. The very first code
that actually runs in the process -- ignoring the dynamic linker for a moment
-- can be found in the compiler's `crt*` files. Those files fetch the `argc`,
`argv`, environment variables and the
[ELF auxiliary vector](https://www.youtube.com/watch?v=j72GExsDnDU&t=1015) from
the kernel, align the stack, initialize the internal libc components, and only
*then* it jumps to `main`. (For more details, visit the Linux Sizecoding Wiki.)

However, this code is rather large, and it's easy to bypass this: `-nostartfiles`.
But, if you use this flag without thinking, you'll get another error: "function
`_start` not found". This is the *real* entrypoint, so just rename your `main`
to `_start`. Then it will run, but your `argc` and `argv` and `environ` will be
garbage. Also, you might notice random crashes in random places that use SSE
instructions.

This means, you'll have to initialize and align the stack yourself. Fortunately,
this isn't too hard: `argc` and `argv` etc. are placed on the stack at process
entry as shown in the ASCII diagram in the midddle of
[this article](https://lwn.net/Articles/631631/). Then call `__libc_start_main`
with `main`, `argc`, `argv`, three null pointers, and finally the initial stack
value. An example of doing this can be found
[here](https://github.com/Shizmob/smol/blob/master/rt/crt1.c), except fetching
the stack pointer value is left as an exercise for the reader.
(Just `mov foo, esp/rsp` isn't a solution because the compiler inserts a
function prologue.)

## Making the linker output smaller

See the next section

# Dynamic ELF, using libraries

As 'vanilla' ELF files can get quite large because of the import and relocation
tables, a few tricks need to be used to decrease the file size.

## Approaches

### postprocessing GNU ld output

See the [page with older tricks](linux-static).

### dnload

[dnload](https://github.com/faemiyah/dnload/) is a tool by Trilkk/faemiyah that
takes C/C++ and GLSL source code and turns it into a packed binary. It works
out of the box, but only supports a fixed set of libraries and symbols. Other
tools are also more optimized.

### smol

[smol](https://github.com/PoroCYon/smol) is a size-optimizing linker that is
able to crunch the size of one's binary a bit further than dnload,
ELF-header-wise. It has been used successfully in [several
productions](https://demozoo.org/productions/tagged/smol/) and is the current
*publicly-availble* state-of-the-art tool.

### A note on other tools

Many other tools you might have heard about, such as bold, elfling, crinklux,
..., have fell prey to bitrot and are currently unusable, as circumstances
have changed: 32-bit ELFs are a dead end, as there are no libraries, and GCC
is emitting different relocation types by now so that programs like Bold that
have their own linking logic break as well. It's a sad state of affairs, but
it is what it is.

Other people and groups (notably epoqe with their own linker 'cold') have
their own tools that may outperform smol, but these are not (yet) publicly
available.

## details

For details on how these linkers work, go to the Linux Sizecoding Wiki (see the
top of this page).

# Using a synth

As most intro synths (4klang, Clinkster, Oidos) are self-contained, they can be
used without too much effort on Linux as well. Furthermore, 64klang2 is made to
work on Linux 'by default', and there are a number of other synths (such as
ghostsyn from faemiyah) that were made especially for Linux (or other Unices).

However, for the trio of 4klang, Clinkster and Oidos, there are a few
complications: they need some linking voodoo to get them to link properly,
and they're 32-bit only. Solutions for the former can be found
[here](https://gitlab.com/PoroCYon/4klang-linux), and for the latter, one can
use a dirty trick:

Make a standalone, static, 32-bit binary that only plays the music (as
explained in the section on library-less coding), embed that executable in the
main binary, and start that on launch. Sync the audio and video using raw
`read`/`write` syscalls to a pipe, as `read` will block when there is no data
available.

# Creating a GL context

As SDL etc. are sometimes not available, one might try to open a window using
libX11, and create an OpenGL context using GLX or EGL. However, using libX11
is often quite brittle, and these three APIs are very verbose.

But, there is a library called libgtk, and it has an API to create an OpenGL
context! So, we can just use that. An example for that can be found
[here](https://github.com/blackle/Linux-OpenGL-Examples/).

However, if you only need a single fullscreen that doesn't take any input and
only has to be shown directly to the screen, you can also use libclutter, which
has functions for that. See [here](https://www.pouet.net/prod.php?which=81604)
or [here](https://github.com/blackle/Clutter-1k/).

If you're lucky, the compo rules might allow the use of SDL, or even GLFW. Use
it! It makes initialization much easier and smaller, but, it mightn't be
available everywhere, so always have a GTK or Clutter backup version.

# Compression

Traditionally, shell droppers have been the most widely used compression tool.
These typically compress the intro binary using a system compression tool (like
xz or lzma) and embed that into a shell script that extracts and executes the
binary. These shell droppers to touch the filesystem, so they were gradually
superseded by assembly stubs that do the same thing. More information on these
can be found on the [other page](linux-static).

However, these still use off-the-shelf compression algorithms, while typical
Windows tools (like Crinkler, kkrunchy/rekkrunchy and Squishy) use custom,
PAQ-based compression algorithms. Few tools like these exist for Linux, but
[oneKpaq](https://github.com/PoroCYon/oneKpaq) is one of them. While [also
having been used
successfully](https://demozoo.org/productions/tagged/onekpaq/), it is not very
suitable for 64k productions, though.

