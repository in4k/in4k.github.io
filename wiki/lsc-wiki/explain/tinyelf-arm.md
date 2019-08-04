# Tiny ELF binaries on ARM

Looks like breadbox didn't want to go down *this* rabbithole. So we'll have to
do it instead.

* Target platform: OABI is dead, so EABI it is. Anything running Linux seems to
  support `thumb`, `half` and `fastmult`, and usually `edsp` as well.
  * So what will the minimum target CPU be? ARMv6T (RPI1)? ARMv5TE?
* All ARM instructions are 4 bytes wide
* There's a "Thumb" mode, with reduced instructions (eg. no free shifts, no
  3-operand instructions) where all instructions are 2 bytes wide
* Switch to Thumb like this: `add lr, pc, #1; bx lr`
  * On older ARMs, it was possible to directly write to `pc` and switch mode,
    but this doesn't seem to be possible on ARMv6 anymore.
* The `mov` opcode doesn't accept arbitrary immediate values, so you sometimes
  have to spill values to a "constant pool", or be creative with assignments
  and register write-backs in load/store ops
* Null bytes decode to `andeq r0, r0` in ARM mode, or to `movs r0, r0` in Thumb
  mode, both are no-ops, unlike in x86 where null bytes cause a segfault.
  * Instruction encoding is relatively sane, so you can predict what low-value
    32-bit ints will decode to so you can treat them as (almost-)no-ops.
* It's a RISC, so you don't have one-byte-instructions, flexible addressing
  modes or stringops, but there are a few useful parts in ARM:
  * `pc`, `sp` etc. behave as regular registers
  * You can shift one operand of an instruction by a constant value for free,
    it doesn't cost any bytes. (ARM mode only) This can also be used to do
    fixed-point multiplications etc. (eg. `add r0, r0, lsr #1` for `r0*1.5`)
  * `ldmia`/`stmia` are great for copying stuff around
* [`e_machine` (and `e_type`) seem to be the only checked header fields
  ](https://code.woboq.org/linux/linux/arch/arm/kernel/elf.c.html) (`e_entry`
  alignment checks are normal, because if it wouldn't be aligned, the code
  would segfault on entry.)
  * Of course, `e_entry`, `e_phoff`, `e_phnum` need to contain the right
    values, and `e_phentsize` and `e_ehsize` need to be correct as well.
* `phdr` parsing etc. is done architecture-independently, so the same tricks
  should be usable here as well.
  * Turns out it's even more relaxed than x86 when messing with `p_paddr`,
    `p_padding` and `p_flags`. It seems to be the case that the kernel and CPU
    will happily let you execute code in read-write pages.
* Apparently the kernel doesn't look at the immediate field of `swi` and `bkpt`
  instructions __if it's configured as EABI-only__ (which we assume).
* [Dynamic linking stuff](https://linux.weeaboo.software/explain/rtld#dynamic-linking_arm)

### Minimal ELF Poc

Not *that* minimal :) (But it should be able to show you which fields can be bogus quite clearly.)

```
gcc -c -o tiny.o tiny.S
ld -nostdlib -nostartfiles -T tiny.ld -o tiny.elf tiny.o
objcopy -O binary tiny.elf tiny.bin # somehow ld --oformat=binary no worky?
```

(Of course, the toolchain should be `arm-linux-gnueabi` or sth.)

```
@ tiny.S
.arch armv5te ; @.cpu arm946e-s

.section .ehdr,"awx",%progbits
.align 4
.arm

#define ORG 0x08048000

ehdr:
        e_ident:
                .byte 0x7F; .ascii "ELF"
                .byte 'p' @ ELFCLASS32
                .byte 'c' @ ELFDATA2LSB
                .byte 'y' @ EV_CURRENT
                .byte '/' @ EI_OSABI_SYSV
                          @ EI_ABIVERSION
                .ascii "K2^TiTAN" @.byte 0,0,0,0,0,0,0 @ EI_PAD
        e_type: .2byte 2 @ ET_EXEC
        e_machine: .2byte 40 @ EM_ARM
        e_version: .ascii "gree"
        e_entry: .4byte _start
        e_phoff: .4byte phdr - ehdr
        e_shoff: .ascii "tsto"
        e_flags:                                  
                @.4byte 0x2|0x4|0x40|0x80|0x00400000|0x05000000
                        @ 2: hasentry
                        @ 4: thumb interwork
                        @ 40: 8-bit struct alignment
                        @ 80: EABI
                        @ 00400000: little-endian AAPCS
                        @ 05000000: EABI v5
                .ascii "brea" @ 0x05000000 @ EABIv5, no float stuff
        e_ehsize: .2byte e__end - ehdr
        e_phentsize: .2byte p__end - phdr
        e_phnum: .2byte 1
        e_shentsize: .ascii "dbox&&"
        @e_shnum:
        @e_shstrndx:
        e__end:
phdr:
        p_type: .4byte 1 @ PT_LOAD
        p_offset: .4byte 0
        p_vaddr: .4byte ORG
        p_paddr: .ascii "all@"
        p_filesz: .4byte _start__end - _start + e__end - ehdr + p__end - phdr
        p_memsz: .4byte _start__end - _start + e__end - ehdr + p__end - phdr
        p_flags: .byte 5 ; ascii "IRC" @ R=4 W=2 X=1
        p_align: .ascii "#lsc" @ 0x1000
        p__end:

.global _start
_start:
        mov r7, #1 @ SYS_exit
        mov r0, #42
        swi #31337 @ literal can be nonsense
        @bkpt #1337 @ like x86 int3 @ literal can be nonsense
_start__end:
```

```
/* tiny.ld */
OUTPUT_FORMAT("elf32-littlearm","elf32-bigarm","elf32-littlearm")
OUTPUT_ARCH(arm)
ENTRY(_start)

SECTIONS {
        . = 0x08048000;

        .ehdr : {
                *(.ehdr*)
        }
}
```

```
00000000  7f 45 4c 46 70 63 79 2f  4b 32 5e 54 69 54 41 4e  |.ELFpcy/K2^TiTAN|
00000010  02 00 28 00 67 72 65 65  54 80 04 08 34 00 00 00  |..(.greeT...4...|
00000020  74 73 74 6f 62 72 65 61  34 00 20 00 01 00 64 62  |tstobrea4. ...db|
00000030  6f 78 26 26 01 00 00 00  00 00 00 00 00 80 04 08  |ox&&............|
00000040  61 6c 6c 40 60 00 00 00  60 00 00 00 05 49 52 43  |all@`...`....IRC|
00000050  23 6c 73 63 01 70 a0 e3  2a 00 a0 e3 69 7a 00 ef  |#lsc.p..*...iz..|
00000060
```
