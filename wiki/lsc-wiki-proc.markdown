---
title: "Process creation"
layout: "wiki-page"
---

Main source: "How programs get run", on lwn
[1](https://lwn.net/Articles/630727/), [2](https://lwn.net/Articles/631631/).
**READ THIS FIRST**

There are a few details that are crucial for sizecoding stuff. On program entry:

* `PT_LOAD` phdrs allocate memory, or map data or code from the executable into
  memory.
* `PT_INTERP` makes the kernel load a second program and execute *that one*,
  after mapping the first one into memory.
* The kernel doesn't care about other phdrs.
* There is a minimum address for memory mapping, addresses lower than this
  value cannot be mapped into userspace memory. This config is available at
  `/proc/sys/vm/mmap_min_addr`, but can only be written to by root.
* The kernel maps pages, not bytes, so the size fields in a phdr are always
  aligned to the next page. Bytes that are not mapped from a file, or are
  "loaded" after the end of the file, are set to zero.
* Pretty much all registers that aren't a stack pointer or program counter are
  set to zero. *This is NOT true when doing dynamic linking*!
* On `x86_64` (and maybe `i386`?), [the stack is aligned to 16 bytes
  ](https://refspecs.linuxbase.org/elf/x86_64-abi-0.99.pdf). The
  `x86_64` calling convention says that the stac pointer mod 16 must be 8 when
  calling a function. [SIMD instructions sometimes require 16-byte alignment
  ](https://pcy.ulyssis.be/intelrefspec.pdf).
  Data on which SIMD instructions are working is sometimes stored on the stack.
  *This means that, if you do not manually realign the stack, crashes will
  happen when doing SIMD. __This code may be in libraries you're depending on,
  and depending on the distro, libraries may or may not be compiled with SIMD
  instructions!__* This can be fixed with one byte: `push rax`.
  * On `arm`, the stack seems to be [aligned to 8 bytes](https://wiki.debian.org/ArmEabiPort#Stack_alignment).
* Lots of interesting data is placed on the stack at program entry. See the
  second lwn article for details.
* For dynamic linking-related stuff on program entry, see [this
  page](lsc-wiki-rtld)
