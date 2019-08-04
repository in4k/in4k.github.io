---
title: "Syscalls"
layout: "wiki-page"
---

Main source: "Anatomy of a System Call", on lwn
[1](https://lwn.net/Articles/604287/), [2](https://lwn.net/Articles/604515/).
**READ THIS FIRST**

Calling a syscall is done by firing a specific interrupt, and the parameters
have to be placed in specific registers first. The kernel then handles the
interrupt as explained in the above articles. (I'm not going to copy those
texts here.)

Each syscall is identified by its number, which should be placed in a specific
regsiter before invoking the syscall. A table can be found in your system's
`include/asm*/unistd*.h` files. Note that syscall numbers are
architecture-dependent and some syscalls aren't implemented on certain hardware
platforms, and some are only available in later versions of the kernel.

### i386

On `i386`, syscalls are invoked using the `int 0x80` instruction. The syscall
number is placed in `eax`, arguments are placed in `ebx`, `ecx`, `edx`, `esi`,
`edi` registers. The return value is placed in the `eax` register.

### x86_64

On `x86_64`, syscalls are invoked using the `syscall` instruction. The syscall
number is placed in `rax`, arguments are placed in `rdi`, `rsi`, `rdx`, `r10`,
`r8` and `r9`. *`r11` and `rcx` are destroyed when invoking a syscall.* The
return value is placed in the `rax` register.

### ARMv6

This is probably true for ARMv5 and ARMv7 as well. No guarantees for ARMv8
(aarch64).

Syscalls are invoked using the `swi #0` instruction. The syscall number is
placed in `r7`, arguments are placed in `r0` through `r6`. The return value is
placed in the `r0` register.

