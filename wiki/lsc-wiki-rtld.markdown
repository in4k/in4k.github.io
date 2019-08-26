---
title: "Dynamic linking"
layout: "wiki-page"
---

Dynamic linking is the process of loading code and data from shared libraries.
For an overview, see [this
](https://0x00sec.org/t/linux-internals-dynamic-linking-wizardry/1082) and [
this page](https://0x00sec.org/t/linux-internals-the-art-of-symbol-resolution/1488),
or [this talk by Matt Godbolt](https://www.youtube.com/watch?v=dOfucXtyEsU).
For a series of very in-depth articles, see [this page
](https://www.airs.com/blog/archives/38) etc.

[Here](https://sourceware.org/glibc/wiki/Debugging/Loader_Debugging)'s a page
on how to debug ld.so, and [here](https://www.gnu.org/software/hurd/glibc/startup.html)'s
one on the glibc startup process (GNU Hurd, but mostly applicable to Linux as well).

### The interesting bits

Or at least, to us.

The kernel recognises a dynamic ELF executable by the fact it has a `PT_INTERP`
phdr which points to the dynamic linker. See [Process creation](lsc-wiki-proc)
for more info. However, it turns out the dynamic linker is just a regular,
statically linked ELF executable, and you can simply give it the program to run
as an argument. When using a shell dropper, this technique spares out a few
bytes.

When the kernel loads our program and the dynamic linker (`ld.so`), it makes
the process jump to the dynamic linker. On glibc, the linker's entrypoint is
`_dl_start_user` ([i386
](https://code.woboq.org/userspace/glibc/sysdeps/i386/dl-machine.h.html#153),
[x86_64
](https://code.woboq.org/userspace/glibc/sysdeps/x86_64/dl-machine.h.html#141)).
It then loads the main program's `PT_DYNAMIC` phdr, which
contains a key-value table which describes which libraries we depend on
(`DT_NEEDED`), where the symbol and string tables are (`DT_SYMTAB`, `DT_STRTAB`
), etc. The dynamic linker then loads these libraries, resolves the needed
symbols, and performs the required relocations to glue all code together.

However, there's one entry, `DT_DEBUG`, which isn't documented (the docs say
"for debugging purposes"). What it actually does, is that the dynamic linker
places a pointer to its `r_debug` struct in the value field. This behavior
is mostly portable (as in, it works on at least glibc, musl, bionic and FreeBSD).
If you look at your system's `link.h` file (eg. in `/usr/include`), you can see
the contents of this struct. The second field is a pointer to the root
`link_map`. More about this one later.

After all the dependencies are loaded etc., the runtime linker will jump to our
entrypoint (`_start`), without clearing the registers or cleaning up the stack.

### The attack plan

Normal dynamic linking needs a lot of (large) ELF header stuff. The trick is to
bypass all this, and do the necessary minimal amount of linking ourselves.

Now let's have a look at the `link_map`. (It's in the same `link.h` header.)
It's a linked list containing information of every ELF file loaded by the rtld.
(The first entry is our binary, the second is the [vDSO
](https://lwn.net/Articles/446528/). Usually, the third one is `libc`.) The
`link_map` contains the path, base address and a pointer to its `PT_DYNAMIC`
phdr. We can use the latter to traverse all the symbol tables and figure out
what the address of every symbol is. This is what **bold** and **dnload** do,
except they save a hash of the names of the required symbols, instead of the
symbol names themselves, computes the hashes of the symbol names from the
`link_map`, and compare those.

However, there are a few ways to save even more bytes:

First of all, instead of using the `DT_DEBUG` trick, we can read some of the
internal linker state that's leaked to our entry point. On `i386`, the
`link_map` ends up in `eax` because `_dl_start_user` loaded it in there, and it
is retained after a function call because of the calling convention.

On `x86_64`, it's a bit more convolved: the register the `link_map` is stored
in doesn't get preserved, but the return address to some place in
`_dl_start_user` is placed on the stack when calling some internal rtld function.
We can read this address, to which we add an offset (to the instruction that
fetches the `link_map`, which is placed in a global variable) to decode the
offset to the `link_map`, and voila.

![_dl_start_user excerpt](https://pcy.ulyssis.be/tmp/img/rtld-hax.png) (`_dl_start_user` excerpt)

The code that fetches the pointer to the `link_map` is then as follows:


    mov r12, [rsp -  8]        ; return address of _dl_init
    mov ebx, dword [r12 - 20]  ; decode part of 'mov rdi, [rel _rtld_global]' ('movq _rtld_global(%rip), %rdi')
    mov r12, [r12 + rbx - 16]  ; ???
    ; r12 is now the link_map pointer

Secondly, instead of iterating over the symbol tables (and having to compute
the hashes of every symbol), we can use the [internal `link_map` data in glibc
](https://code.woboq.org/userspace/glibc/include/link.h.html#link_map) to access
the calculated symbol hash tables, so we only have to do a hashtable lookup to
get the address of a symbol.

However, the offset to the hashtable in the `link_map` tends to vary between
glibc versions. But, note the presence of the `l_entry` field somewhere down
there. This value is known at (static) link time, we control it, so we can
simply scan the struct for the value, and use the difference between the
address of the `link_map` and the `l_entry` fields to compute the offset
between the "near" and "far" fields of the `link_map`, and use that to
read the hashtables.

Smol uses these two tricks to achieve an even smaller binary size.

### ARM

Most of the tricks presented here don't depend on the processor architecture.
Getting hold of the `link_map`, however, needs yet another hack to get it to work.

[glibc does the following](https://code.woboq.org/userspace/glibc/sysdeps/arm/dl-machine.h.html#153):

    @ call internal init stuff w/ link_map pointer
    ldr r0, .L_LOADED
    ldr r0, [sl, r0]
    bl _dl_init(PLT)

    @ load _dl_fini, jump to entrypoint
    ldr r0, .L_FINI_PROC
    add r0, sl, r0
    mov pc, r6
    
This has a few problems:
* `r0` (containing the `link_map` struct) always gets overwritten
* The return address is saved to `lr` instead of being written to the stack. This means we can't use the stack trick as in x86_64

The only useful thing that gets passed to our entrypoint is the `sl` register. The address to `.L_LOADED` would still be needed, though.

For now, only the dnload-style loading might be possible. This should work with glibc, musl and probably bionic as well.
