---
title: "MacOS X"
layout: "wiki-page"
---

W O R K  I N   P R O G R E S S ! ! !

* * *

Making size limited 1k/4k intros for OS X can be said it is like hybrid from Windows and unix systems. On the other hand tricks like import-by-hash dynamic loading is rather starightforward to do like in Windows, on the other hand the unix backend allows some nifty things like shell dropping.

In high-level there are certain pros and cons working with OS X in size-limited world.

Pros:
* OS X frameworks make it possible to load huge amounts of functionality with a single dependency (Cocoa)
* Loading libraries and resolving symbols from libraries using hash function is really straightforward and easy
* Static overhead is rather minimal. Decompression + Dynloading amounts to ~250 bytes.

Cons:
* Minimum size of OS X MACH-O binary is 4096. (Since 10.10.5) This is no issue for 4k intros, but basically mandates shell dropping on 1k
* OpenGL is either legacy or core, you can't mix. GLSL compiler is strict about syntax
* Apple is killing of 32-bit libraries and frameworks one by one. Moving to 64-bit loses some benefits there are in OS X
* Apple likes toss around functions between libraries in every major version making hash-imported libraries break.

* * *

Fundamentals (32 bit executables)

The OS X native executable format, [Mach-O](http://developer.apple.com/documentation/DeveloperTools/Conceptual/MachORuntime/index.html), contains bare minimum header and load commands followed by content for the segments defined. A practical executable needs segment, main, dylinker and at least one dylib command.

Due to recent jailbreak exploits the kernel, Apple has made the kernel side format validation really strict. Mach-O header and load commands needs to be of correct size and offsets need to point into the correct command issued. Only commands loaded by dyld can be still fudged around. Practically this means that only library list and sdk-header can be modified, which does not optimize much but allows making header more regular for compression purposes.

Example of minimal executable with 180-bytes of header

```Assembly
	org 0
	bits 32

%include "symbols.asm"

FileStart:
	[section .text]
MACHHeader:
	dd 0xfeedface				; magic
	dd SOLVE(CPU_TYPE_X86)			; cpu type
	dd 0					; cpu subtype (wrong, but works)
	dd SOLVE(MH_EXECUTE)			; filetype
	dd 4					; number of commands
	dd MACHCommandsEnd-MACHCommandsStart	; size of commands
	dd 0					; flags
MACHHeaderEnd:

MACHCommandsStart:

Command1Start:
	dd SOLVE(LC_MAIN)			; this is main
	dd Command1End-Command1Start		; size
	dd CodeStart-MACHHeader			; eip
	dd 0
	dd 0					; stack
	dd 0
Command1End:

Command2Start:
	dd SOLVE(LC_SEGMENT)			; this is segment
	dd Command2End-Command2Start		; size
SegNameStart:
	times 16-$+SegNameStart db 0		; section name

	dd 0					; vmaddr
	dd 0x100000				; vmsize
	dd 0					; fileoff
	dd FileEnd				; filesize
	dd 0					; maxprot
	dd SOLVE(VM_PROT_READ)|SOLVE(VM_PROT_WRITE)|SOLVE(VM_PROT_EXECUTE) ; initprot
	dd 0					; nsects
	dd 0
Command2End:

Command3Start:
	dd SOLVE(LC_LOAD_DYLINKER)		; dyld
	dd Command3End-Command3Start		; size
	dd DyldName-Command3Start		; offset
DyldName:
	db '/usr/lib/dyld',0			; 14 bytes
;	db 0,0					; breaking the spec here, no-one minds.
Command3End:

Command4Start:
	dd SOLVE(LC_LOAD_DYLIB)
	dd Command4End-Command4Start		; size
	dd DyName-Command4Start			; offset
	dd 0					; timestamp
	dd 0					; min ver
	dd 0 					; max ver
DyName:
	db 'Cocoa.framework/Cocoa',0
Command4End:

MACHCommandsEnd:

CodeStart:
	ret
CodeEnd:

	times 4096-(CodeEnd-MACHHeader) db 0
FileEnd:
```

* * *

OpenGL

* * *

Audio

* * *

Tools

* * *

