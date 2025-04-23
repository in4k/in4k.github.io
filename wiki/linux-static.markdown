---
title: "Linux (static executables and obsolete or niche tricks)"
layout: "wiki-page"
---

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
`/dev/vcsa` for tiles/'characters', or direct `/dev/dri/card0` access) are
simple to access for pixel rendering, but they're not very easy to access on
desktops or laptops. Therefore, if you want to do decent visuals in a
compatible way (and text terminal output is not enough for you), you must learn
how to talk the X11 protocol in a low level. Basically, you establish a
connection to localhost:6000, send the initial request, grab the id's you
require from the response, open a window, etc. etc. It may be a bit
long-winded, but you also get a low-level access to extensions such as XVideo
and GLX. However, if you want access to the GPU (and don't want to code your
own GPU drivers), this isn't exactly an option as the GPU drivers need to be
loaded from dynamic libraries.

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

See the [main Linux page](/wiki/linux.markdown)

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

### Using a special-purpose minimizing linker

See the [main Linux page](/wiki/linux.markdown)

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

