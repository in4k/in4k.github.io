---
title: "vondehi"
layout: "wiki-page"
---

Vondehi is an in-memory unpacker for data compressed with `gzip` or `xz` (or
`xz` in LZMA1-mode). It basically performs these steps:

1. Set up a memfd using the `memfd_create` syscall. This file descriptor works
   like a regular file, except the backing storage is RAM.
2. Fork, pipe the payload data to `zcat` or `xzcat`, which outputs everything
   to the memfd from step 1.
3. Run `execveat` on the memfd.

Of course, the code itself is hand-optimized x86 assembly, and is very crazy.

