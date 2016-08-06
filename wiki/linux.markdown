---
title: "Linux"
layout: "wiki-page"
---

# Introduction to Linux as 4 KB Platform

Linux is at advantage by the fact that most standard Linux distributions ship a lot more libraries than Win32 which you can assume being available in the system. Some of the useful ones are e.g. SDL and GLUT which you can use as the base framework for the application giving you graphics output and event handling. SDL also provides sound output capabilities.

More resources:

*   [Linux 4k Intro Coding Presentation Slides](http://ftp.kameli.net/pub/fit/misc/presis_asm06.pdf "http://ftp.kameli.net/pub/fit/misc/presis asm06.pdf") from Assembly'2006, by Markku "Marq" Reunanen ([Local copy](ftp://ftp.untergrund.net/users/in4kadmin/files/Linux_4k_Intro_Coding_asm06.pdf "ftp://ftp.untergrund.net/users/in4kadmin/files/Linux 4k Intro Coding asm06.pdf"))
*   [Kickers of ELF](http://www.muppetlabs.com/~breadbox/software/elfkickers.html "http://www.muppetlabs.com/~breadbox/software/elfkickers.html")
*   [A Whirlwind Tutorial on Creating Really Teensy ELF Executables for Linux](http://www.muppetlabs.com/~breadbox/software/tiny/teensy.html "http://www.muppetlabs.com/~breadbox/software/tiny/teensy.html")
*   [Haxxoring the elf format for 1k/4k stuff](http://www.pouet.net/topic.php?which=5392 "http://www.pouet.net/topic.php?which=5392"): A very interesting thread on Pouet.net
*   [GC Masher](http://ftp.kameli.net/pub/fit/misc/gcmasher11082005.tar.gz "http://ftp.kameli.net/pub/fit/misc/gcmasher11082005.tar.gz") - a tool to brute force through different compiler parameters in order to find a combination which produces smaller binary. ([Local copy](ftp://ftp.untergrund.net/users/in4kadmin/files/gcmasher11082005.tar.gz "ftp://ftp.untergrund.net/users/in4kadmin/files/gcmasher11082005.tar.gz"))
*   [Bold, the Byte Optimized Linker](http://www.alrj.org/projects/bold/ "http://www.alrj.org/projects/bold/") An minimalist Linux/x86_64 ELF linker
*   [ELF64 Anatomy](/index.php?title=ELF64_Anatomy "ELF64 Anatomy")

# 4K programming without libraries

Linux (and most other Unix-like systems) also has the advantage (compared to many other operating systems) that you don't need to use any libraries at all if you don't want to, because everyhing can be done through kernel-level system calls (the _int_ opcode in x86 assembly). This is a totally different approach than the previously mentioned one.

## Advantages to using libraries

*   Less than 80 bytes of headers are required (this applies to basically any system that uses the ELF32 binary format).
*   All library-related overhead in headers and code is avoided.
*   Many coders get more "hard-core kicks" from straightforward low-level programming than from the abuse of huge libraries. In this case, you can basically just include the header and then continue with pure assembly code and a couple of essential syscalls.
*   A surprising advantage is that the low-level interfaces are actually the most reliable ones on Unix-like systems. The kernel interface and the X11/GLX protocols are quite mature and do not tend to change every now and then, like many library binary interfaces do.

## Disadvantages

*   You have to do a lot of the low-level work by yourself.
*   Many of the things are rather complicated to implement and may require more bytes than a simple library access would do.

## How to do it

Accessing the sound device is usually a simple task (open the device file with _open_, set its parameters and monitor itse status with _ioctl_, and write out the pcm data with _write_).

The visual side is more complicated. Framebuffer devices are simple to access for pixel rendering, but they're not very often available on PC-based systems. Therefore, if you want to do decent visuals in a compatible way (and text terminal output is not enough for you), you must learn how to talk the X11 protocol in a low level. Basically, you establish a connection to localhost:6000, send the initial request, grab the id's you require from the response, open a window, etc. etc. It may be a bit long-winded, but you also get a low-level access to extensions such as XVideo and GLX.

*   [x11stub-20000828](http://www.pelulamu.net/pwp/x11stub-20000828.tar.gz "http://www.pelulamu.net/pwp/x11stub-20000828.tar.gz") - an old experiment by viznut/pwp on pixel-rendering with raw X11 calls. ([Local copy](ftp://ftp.untergrund.net/users/in4kadmin/files/x11stub-20000828.tar.gz "ftp://ftp.untergrund.net/users/in4kadmin/files/x11stub-20000828.tar.gz"))

# Self-compiling

Another approach usable for 4K intros on Unix platforms is self-compiling. That is, you actually distribute the intro as an executable source code package that decompresses, compiles, executes and deletes the actual intro. A shell script stub attached to the compressed source code of an SDL-based intro could perhaps be something like this:

```
a=/tmp/I;zcat<<X>$a;cc `sdl-config --libs --cflags` -o $a. $a;$a.;rm $a.;exit
```

## Advantages

*   You can use high-level languages and standard libraries without feeling lame.
*   The intro can be truly cross-platform one (rather independent of the actual Unix variant and processor architecture). Also rather easy to port or patch for other platforms.
*   You don't need to hassle with binary optimization.

## Disadvantages

*   The compilation time adds to the start-up time.
*   The code density in C source code is not as good as, say, x86 machine code.
*   The compilation may fail for a multitude of reasons that may depend on your distribution, headers, compiler version etc.
*   You need either to write the code in a highly compact manner (avoid newlines etc) or to write a postprocessor that minimizes the code for you.
*   You will have to release the source code (although in a rather obfuscated form).

# Compression

It's generally believed that the best 4K executable compression on Unix platforms is by a shell-script stub that drops the actual executable into a temporary file and starts it. A canonical Unix tool, gzexe, creates executables like this, but with quite a bad overhead. It's quite easy to come up with much shorter solutions.

Here's the stub presented by Marq/Fit in his Assembly 2006 seminar (56 bytes):

```
a=/tmp/I;tail -n+2 $0|zcat>$a;chmod +x $a;$a;rm $a;exit
```

Paul Sladen came up with the following shorter stub (you must pre `tac` the binary part, before concatenation, so that the transform is fully reversible; this may require appending a `\n` to the end of the file):

```
HOME=/tmp/$;cp $0 ~;tac $0|zcat>~;~;rm ~;exit
```

Using the current directory instead of `/tmp` saves a couple of bytes but may cause problems if you don't have a write access to that directory or if the competition rules prohibits usage of anything else than temporary directories for write access.

Gzip seems to be better in most cases for the 64 kB and under size class than Bzip2\. Idea for improvement: use the multi-line quoting syntax `<<ENDMARKER`); generally experiment as much as you can. You can also do the tempfile removal in the actual code, just after the startup. You may also be able to replace the "exit" command with your own code.

To achieve maximum compression with gzip use command line switches **-n** (skip embedding the filename) and **--best** (already the default); however there are other gzip _encoders_ around (just like there are lots of MPEG/MP3 encoders but the output an always be decoded by any decoder).

One gzip encoder that can often produce more compact (but compatible) results is the deflate encoder found originally in the [p7zip](http://p7zip.sourceforge.net/ "http://p7zip.sourceforge.net/"); this encoder is available on Debian/Ubuntu with `apt-get install advancecomp`. Compression tests have shown that compressing Linux 4K intros with the 7zip/advancecomp implementation saves 40 to 100 bytes in the final result.

You can also try optimising the deflate stream with [`kzip`](http://www.jonof.id.au/index.php?p=kenutils "http://www.jonof.id.au/index.php?p=kenutils") and then optimising the storage of the Huffman header on the front (single digit byte gains only) using Ben Jos Walbeehm's [`DeflOpt`](http://www.walbeehm.com/download/ "http://www.walbeehm.com/download/").
