## Explanations

* [Target platform]($docroot$lsc-wiki/target.html)

* [Syscalls]($docroot$lsc-wiki/explain/syscalls.html)
* [Creating small static binaries, ELF header
  hacks](https://www.muppetlabs.com/~breadbox/software/tiny/teensy.html) ([notes on doing the same, on ARM](/explain/tinyelf-arm))
* [Process creation]($docroot$lsc-wiki/explain/proc.html)
* [Dynamic linking]($docroot$lsc-wiki/explain/rtld.html)
* [C runtime]($docroot$lsc-wiki/explain/crt.html)

* [dnload](https://github.com/faemiyah/dnload/blob/master/README.rst):
  interesting read on all things sizecoding on mostly FreeBSD, but is
  applicable to Linux as well. Mostly correct and up-to-date, minus
  our smol hacks.
* [vondehi]($docroot$lsc-wiki/explain/vondehi.html)
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

