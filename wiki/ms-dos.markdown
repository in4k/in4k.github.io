---
title: "MS-DOS"
layout: "wiki-page"
---

# Introduction to MS-DOS as a 4kb platform

DOS (meaning MS-DOS and compatibles) is one of the most straightforward platforms for sizecoding. Most <=256-byte intros are still made for DOS, mostly because of the minimal overhead compared to Win32 and Linux. However, present-day 4K intro developers already prefer more modern systems that provide an easier access to sound and 3D.

## Advantages of DOS

*   There are no headers in .COM executables, only plain code. The minimal executable consists of only one byte (the opcode for RET).
*   The most essential things (graphics mode initialization, pixel rendering, keyboard status check, vertical retrace sync) can be implemented in a very small amount of code, which also makes it possible to make graphical effects in only some tens of bytes.
*   It is still possible to run DOS executables in modern Win32 systems out-of-the-box.

## Disadvantages of DOS

*   It is difficult to access the sound hardware in a compatible way, especially if you want a digital PCM buffer rather than simple FM sounds. Nowadays, sound is essential to the 4K size class.
*   The notion of difficulty and incompatibility also applies to truecolor graphics and 3D acceleration features.
*   The x86 realmode poses some very archaic limitations in memory management (64K segments, 640K base memory). Using a DOS extender solves these problems but risks compatibility and adds significant overhead.
*   You can't access any fancy libraries that do things like 3D calculation or decompression for you.
