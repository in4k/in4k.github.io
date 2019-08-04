---
title: "PoroCYon's Linux Sizecoding Wiki Mirror"
layout: "wiki-page"
---

## Talks

### unlord (LCA 2019)

[pdf](https://people.xiph.org/~unlord/LCA2019.pdf); [video](https://www.youtube.com/watch?v=J5WX-wN_RKY)

### Shiz & PoroCYon (Revision 2019)

[pdf](https://pcy.ulyssis.be/pres/Lin.pdf); [video](https://www.youtube.com/watch?v=a03HXo8a_Io)

#### Errata

* `-march=haswell` should be **`-march=core2`**,
  the latter *usually, but not always,* generates smaller code
  ([source](https://github.com/faemiyah/dnload#compiler-flags))
* Ferris' "Squishy" packer isn't exactly '*general-purpose*'
  as compared to kkrunchy, but the approach is *more general* at least.
* runit and s6 are perfectly fine at supervising daemons.
* ryg/fr made kkrunchy, not kb/fr.

## Tools

### dynamic linkers

* ~~[elfling](https://github.com/google/elfling)~~ (currently broken)
* ~~[bold](http://www.alrj.org/pages/bold.html)~~ (currently broken)
* [dnload](https://github.com/faemiyah/dnload) (Linux, FreeBSD)
* [smol](https://github.com/Shizmob/smol) (glibc Linux, with non-glibc
  "compatibility mode")

### compressors

* shell dropping: `cp $$0 /tmp/M;(sed 1d $$0|lzcat)>$$_;exec $$_;`
* ~~[fishypack
  ](https://bitbucket.org/blackle_mori/cenotaph4soda/src/master/packer/?at=master): In-memory `memfd`/`execveat`-based decompressor~~ (Deprecated in favor of vondehi)
* [vondehi](https://gitlab.com/PoroCYon/vondehi): Next-generation in-memory `memfd`/`execveat`-based decompressor (See list of known bugs)
  * [autovndh](https://gitlab.com/snippets/1800243): script that bruteforces all possible gzip/lzma/xz/... options and automatically places a vondehi decompression stub at the beginning
* [oneKpaq](https://github.com/temisu/oneKpaq): Generic PAQ-based (de)compressor for 32-bit x86

### synths

* [ghostsyn](https://github.com/Juippi/ghostsyn)
* [4klang](https://www.pouet.net/prod.php?which=53398), [ForkedKlang
  ](https://www.pouet.net/topic.php?which=11312). Note that the replayer code
  is 32-bit only! You'll need some hacks to make it work on 64-bit.
* [Clinkster](https://www.pouet.net/prod.php?which=61592), replayer is 32-bit only as well.
* [Oidos](https://www.pouet.net/prod.php?which=69524), replayer is once more 32-bit only.
* [Axiom](https://github.com/monadgroup/axiom/) is "supposed" to work

## Example code

### Basic GL stuff

* [liner (intro template)](https://github.com/shizmob/liner)
* [more GL init stuff](https://github.com/blackle/Linux-OpenGL-Examples/)
* [even smaller GL init if you only need a single fullscreen fragment shader,
   still works with default Ubuntu](https://github.com/blackle/Clutter-1k/)

### Basic synth setup

* [4klang](https://gitlab.com/PoroCYon/4klang-linux/tree/master/4klang)
* [Clinkster](https://gitlab.com/PoroCYon/4klang-linux/tree/master/clinkster)
* [Oidos](https://gitlab.com/PoroCYon/4klang-linux/tree/master/oidos)
* [V2](https://gitlab.com/PoroCYon/4klang-linux/tree/master/v2)
* TODO:
  * Quiver?
  * More ad-hocish 4k synths
  * 64klang2
  * WaveSabre? (I heard MacSlow is/will be working on this one?)
  * the Brain Control one? ("Tunefish"?)
  * others
* Ghostsyn and Axiom should work by default (see [tools](/tools))

### Source code of a few intros

* [International Shipping by Suricrasia Online
  ](https://bitbucket.org/blackle_mori/international-shipping)
* [Ninjadev dabbles in intros](https://github.com/aleksanb/fourkay)
* [scaleMARK by Suricrasia Online](https://bitbucket.org/blackle_mori/scalemark)

### Other stuff

* [Tetris clone, in 2k](https://github.com/donnerbrenn/Tetris2k)

## Explanations

## Some notes about the target platform

We'll mostly target the Revision compo rules, as it's the biggest party, and
they seem to be the most strict.

* ***The distro is always the latest Ubuntu***
* ***CODE WILL NOT BE RAN AS ROOT***
* These packages are installed by default: [18.10
  ](http://releases.ubuntu.com/18.10/ubuntu-18.10-desktop-amd64.manifest),
  [19.04](http://releases.ubuntu.com/19.04/ubuntu-19.04-desktop-amd64.manifest).
* Note that only the 64-bit versions of these libraries are installed. *If
  you're running a 32-bit binary, it **MUST** be static!*
* No SDL. Or maybe [not anymore?](lsc-wiki-las-sdl-revision)

* [Syscalls](lsc-wiki-syscalls)
* [Creating small static binaries, ELF header
  hacks](https://www.muppetlabs.com/~breadbox/software/tiny/teensy.html) ([notes on doing the same, on ARM](lsc-wiki-tinyelf-arm))
* [Process creation](lsc-wiki-proc)
* [Dynamic linking](lsc-wiki-rtld)
* [C runtime](lsc-wiki-crt)

* [dnload](https://github.com/faemiyah/dnload/blob/master/README.rst):
  interesting read on all things sizecoding on mostly FreeBSD, but is
  applicable to Linux as well. Mostly correct and up-to-date, minus
  our smol hacks.
* [vondehi](lsc-wiki-vondehi)
* **smol**: see dynamic linking

### Interesting links

* [ELF stuff](https://www.cs.stevens.edu/~jschauma/631A/elf.html) (more than what's in the official docs)
* [Glibc ld.so internals](http://s.eresi-project.org/inc/articles/elf-rtld.txt)
* [GNU hash sections, in-depth](https://web.archive.org/web/20111022202443/http://blogs.oracle.com/ali/entry/gnu_hash_elf_sections)
* [ILT's series of articles on all things linking](https://www.airs.com/blog/archives/38)
* ["How we get to `main()`" (video)](https://www.youtube.com/watch?v=dOfucXtyEsU)
* ["Everything you always wanted to know about Hello World" (video)](https://archive.fosdem.org/2017/schedule/event/hello_world/)
* [link-time GNU hash stuff (src)](https://sourceware.org/git/?p=binutils.git;a=blob_plain;f=bfd/elf.c)
* [runtime GNU hash stuff (src)](https://sourceware.org/git/?p=glibc.git;a=blob_plain;f=elf/dl-lookup.c)
* ["Linkers and loaders" (book, pdf), very extensive](http://becbapatla.ac.in/cse/naveenv/docs/LL1.pdf)

## Misc

* [Collection of useful code to get started](https://weeaboo.software/lsc)
* Packages installed by default: [Ubuntu 18.04(.2)](http://releases.ubuntu.com/18.04.2/ubuntu-18.04.2-desktop-amd64.manifest), [Ubuntu 18.10](http://releases.ubuntu.com/18.10/ubuntu-18.10-desktop-amd64.manifest), [Ubuntu 19.04](http://releases.ubuntu.com/19.04/ubuntu-19.04-desktop-amd64.manifest)
* [Las' SDL explanation](lsc-wiki-las-sdl-revision)
* [Current discussion on doing ARM stuff](https://linux.weeaboo.software/explain/tinyelf-arm)

## Talk with us

Join `#lsc` on IRCnet! ([webchat](https://webchat.ircnet.net/?channels=lsc))
