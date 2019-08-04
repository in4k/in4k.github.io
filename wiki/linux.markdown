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

* [Linux 4k Intro Coding Presentation Slides](http://ftp.kameli.net/pub/fit/misc/presis_asm06.pdf)
  from Assembly'2006, by Markku "Marq" Reunanen ([local copy](https://files.scene.org/view/resources/in4k/linux_4k_intro_coding_asm06.pdf)):
  contains some useful info, but is very outdated.
* [Kickers of ELF](http://www.muppetlabs.com/~breadbox/software/elfkickers.html)
  ([local copy](/html_articles/ELF Kickers.html)): A collection of useful tools.
* [A Whirlwind Tutorial on Creating Really Teensy ELF Executables for Linux](http://www.muppetlabs.com/~breadbox/software/tiny/teensy.html)
  ([local copy](/html_articles/A Whirlwind Tutorial on Creating Really Teensy ELF Executables for Linux.html)):
  introduction on messing with ELF headers and Linux syscalls. *The* article to
  start with if you want to do some ELF hacking.
* [Haxxoring the elf format for 1k/4k stuff](http://www.pouet.net/topic.php?which=5392):
  A very interesting thread on Pouet.net. Mostly 32-bit stuff, but the principles
  apply to 64-bit as well.
* [GC Masher](http://ftp.kameli.net/pub/fit/misc/gcmasher11082005.tar.gz) - a
  tool to brute force through different compiler parameters in order to find a
  combination which produces smaller binary. ([local copy](https://files.scene.org/view/resources/in4k/gcmasher11082005.tar.gz))
* [Bold, the Byte Optimized Linker](http://www.alrj.org/projects/bold/) A
  minimalist Linux/x86_64 ELF linker. *Currently broken because it can't deal
  with the relocation types currently emitted by GCC and Clang.*
* [ELF64 Anatomy](/index.php?title=ELF64_Anatomy "ELF64 Anatomy")
* [Technicalities for very small (1k/4k) Raspberry Pi intros](http://www.pouet.net/topic.php?which=10915)
* [dnload documentation](https://github.com/faemiyah/dnload/)
* [PoroCYon's Linux Sizecoding wiki mirror](https://pcy.be/lsc-wiki.html)
  ([original](https://linux.weeaboo.software/) (down?), [local copy](lsc-wiki))

# 4K programming without libraries

Linux (and most other Unix-like systems) also has the advantage (compared to
many other operating systems) that you don't need to use any libraries at all
if you don't want to, because everyhing can be done through kernel-level system
calls (the `int` opcode in x86 assembly, or `syscall` in 64-bit mode). This is
a totally different approach than the previously mentioned one.

## Advantages compared to using libraries

* Less than 80 bytes of headers are required (this applies to basically any
  system that uses the ELF32 binary format).
* You can use 32-bit assembly code, because you aren't relying on any
  libraries that are only available as 64-bit.
* All library-related overhead in headers and code is avoided.
* A surprising advantage is that the low-level interfaces are actually the most
  reliable ones on Unix-like systems. The kernel interface and the X11/GLX
  protocols are quite mature and do not tend to change every now and then, like
  many library binary interfaces do.

## Disadvantages

* You have to do a lot of the low-level work by yourself.
* Many of the things are rather complicated to implement and may require more
  bytes than a simple library access would do.
* All the GPU drivers are userspace, so you need some way of loading dynamic
  libraries if you want to do OpenGL stuff.

## How to do it

The exact mechanics of how to make a syscall can be found in the "Whirlwind
Tutorial on Creating Teensy ELF Executables".

Audio output can be achieved easily by creating a pipe (using the `pipe`
syscall), `fork`ing, `dup2` in the read end of the pipe to `stdin` and `execve`ing
the child to `/usr/bin/aplay`. Then, the parent process can simply write PCM
data to the pipe, and sound will appear.
[Here](https://www.pouet.net/prod.php?which=70107) is a prod that does just this.

The visual side is more complicated. Framebuffer devices (`/dev/fd*` for pixels,
`/dev/vcsa` for tiles, or direct `/dev/dri/card0` access) are simple to access
for pixel rendering, but they're not very easy to access on desktops or laptops.
Therefore, if you want to do decent visuals in a compatible way (and text
terminal output is not enough for you), you must learn how to talk the X11
protocol in a low level. Basically, you establish a connection to
localhost:6000, send the initial request, grab the id's you require from the
response, open a window, etc. etc. It may be a bit long-winded, but you also
get a low-level access to extensions such as XVideo and GLX. However, if you
want access to the GPU (and don't want to code your own GPU drivers), this
isn't exactly an option as the GPU drivers need to be loaded from dynamic
libraries.

* [x11stub-20000828](http://www.pelulamu.net/pwp/x11stub-20000828.tar.gz):
  an old experiment by viznut/pwp on pixel-rendering with raw X11 calls.
  ([Local copy](ftp://ftp.untergrund.net/users/in4kadmin/files/x11stub-20000828.tar.gz))
  Currently known to segfault on some systems.

# Self-compiling

Another approach usable for 4K intros on Unix platforms is self-compiling. That
is, you actually distribute the intro as an executable source code package that
decompresses, compiles, executes and deletes the actual intro. A shell script
stub attached to the compressed source code of an SDL-based intro could perhaps
be something like this:

```
a=/tmp/I;zcat<<X>$a;cc `sdl-config --libs --cflags` -o $a. $a;$a.;rm $a.;exit
```

## Advantages

* You can use high-level languages (Python is usually available in many places).
* The intro can be truly cross-platform one (rather independent of the actual
  Unix variant and processor architecture). Also rather easy to port or patch
  for other platforms.
* You don't need to hassle with binary optimization.
* Instead of using C, you can also use Python and its `ctypes` library. See eg.
  [this prod](https://www.pouet.net/prod.php?which=81604). This way you can
  avoid the overead in C coding, but you'll have to mess with the `ctypes`
  library bugs to make it work in 64-bit mode.

## Disadvantages

* The compilation time adds to the start-up time.
* The code density in C source code is not as good as, say, x86 machine code.
  However, C source code (plain text) compresses better than x86 assembly.
* The compilation may fail for a multitude of reasons that may depend on your
  distribution, headers, compiler version etc. *Standard Ubuntu systems do
  **NOT** come with development headers!*
* You need either to write the code in a highly compact manner (avoid newline
  etc) or to write a postprocessor that minimizes the code for you.

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

This is arguably the easiest and most stable option, but isn't the most
effective, size-wise. However, ld doesn't like to output small binaries, so you
have to bully it a little bit:

1. Use a custom linker script that merges the `.text` (code) and
  `.rodata` (readonly data) sections, discards `.gnu.warning`, `.note.*`,
  `.gnu_debuglink`, `.comment`, `.gnu.attributes`, `.eh_frame*`, `.gcc_except*`,
  `.gnu_extab*`, `.exception*`, etc. Get the default linker script using `-Wl,-v`,
  then modify it. See the GNU ld docs for the syntax.
2. Remove symbol version data using `objcopy -R .gnu.version -R .gnu.version_r`.
3. Run [noelfver](https://gitlab.com/PoroCYon/noelfver) on the output of `objcopy`.
   This actually disables the use of the versioning info.
4. `strip -s`. This removes symbol and debug data etc, but leaves the section
   headers in. Also removes remaining versioning data now that it is no longer
   used thanks to *noelfver*.
5. `sstrip -z` to also remove the section headers (from ELF Kickers)
6. `chmod -x`

### dnload

[dnload](https://github.com/faemiyah/dnload/) is a tool by Trilkk/faemiyah that
takes C/C++ and GLSL source code and turns it into a packed binary. It works
out of the box, but only supports a fixed set of libraries and symbols.

### smol

[smol](https://github.com/Shizmob/smol) (['staging' branch](https://github.com/PoroCYon/smol))
is an experimental, currently-work-in-progress size-optimizing linker that is
able to crunch the size of one's binary a bit further than dnload,
ELF-header-wise. It doesn't come with batteries included, and its options can
get a bit confusing, so getting it to work takes some effort.

### A note on other tools

Many other tools you might have heard about, such as bold, elfling, crinklux,
..., have fell prey to bitrot and are currently unusable, as circumstances
have changed: 32-bit ELFs are a dead end, as there are no libraries, and GCC
is emitting different relocation types by now so that programs like Bold that
have their own linking logic break as well. It's a sad state of affairs, but
it is what it is.

## details

For details on how these things could be implemented, go to the Linux
Sizecoding Wiki.

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
is often quite brittle, and these three APIS are very verbose.

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

A quick and easy way to compress your executable is to use a *shell script
dropper*: a small shell script that unpacks and runs itself, followed directly
by your compressed intro.

Using a shell dropper, you can compress your intro using only tools of which
decompressors are installed by default -- gzip, xz, lzma, bzip2, etc. Gzip
(`gzip -cnk9` for compression, `zcat` for decompression) usually isn't very
good -- zopfli might get you slightly better compression results, but xz (`xz`
for compression, `xzcat` for decompression, see manpage for flags) is usually
much better. However, standard xz files have large headers compared to gzip.
Therefore, it's a better idea to use xz in LZMA1 mode (`xz --lzma1`), which has
slightly worse compression, but much smaller headers. With these small sizes,
LZMA1 usually ends up performing slightly better than xz because of this.

The smallest shell dropper I've come across is this one, which takes 42 bytes:

```
cp $0 /tmp/M;(sed 1d $0|xzcat)>$_;$_;exit
<compressed data>
```

Of course, this assumes that `/tmp` is executable. One could change the `/tmp/M`
to just `M` to put it in the current directory, but that isn't guaranteed to
be writable either!

However, there is a known 'fix' for this: using an in-memory dropper written in
assembly. [vondehi](https://gitlab.com/PoroCYon/vondehi) is currently the best
of those. It basically does the same things as the shell script, except
everything is performed in RAM. Simply take the output of the NASM file and
append your compressed data.

If you want to tune the compression parameters to get a slightly smaller output
file, there's a script called [autovndh.py](https://gitlab.com/snippets/1800243)
that bruteforces all options, and optionally also prepends a vondehi stub as well.

This still isn't the best compression ratio possible, however. As Crinkler on
Windows performs much better because of its use of a PAQ-style algorithm (as
opposed to LZMA), the tooling isn't finished yet. Work has been done towards
reaching a Crinkler equivalent, as there are some obsolete tools that do this,
as well as a current work-in-progress project called
[XLINK](https://github.com/negge/xlink) by unlord/xylem. The latter isn't
usable yet, though.

