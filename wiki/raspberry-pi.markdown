---
title: "Raspberry Pi"
layout: "wiki-page"
---

# Introduction to Raspberry Pi as 4 KB Platform

Taken from Trilkk's [Technicalities for very small (1k/4k) Raspberry Pi intros](http://www.pouet.net/topic.php?which=10915) pouet.net forum thread:

After Assembly, in the comment section of Fuzzdealer, there was discussion on how small Raspberry Pi intros are possible currently. I've investigated this lately, and could as well share the findings.

It turns out most of the tricks can indeed be done, but both the limitations on 32-bit ARM and the current state of Raspbian or other Linux distributions limits the minimum possible size.

--

Let's first assume that we're going to use C/C++ since being lazy is the way to go. If we try to do the normal procedure:

* Make ELF32 header stub, fill in armv6 specifics.
* Collect required symbols from our source file, make the classic r_debug loader.
* Compile our intro to asm with -S.
* Rip out header pushes and exit code after debug break from our entry point (_start).
* Make fake .bss, have it exist after end of file 
* Concatenate the header stub and intro together.
* Link to an ELF object file.
* Copy result out with objcopy to a standalone executable.
* Add xz shell script unpacker header, concatenate it with executable.

...and it fails.

The reason is, unlike ia32/amd64, arm32 is missing some essentials most intros would want:
```
intro.cpp:(.text+0x4cc): undefined reference to `memset'
intro.cpp:(.text+0x4f0): undefined reference to `memset'
intro.cpp:(.text+0x73c): undefined reference to `__aeabi_uidivmod'
intro.cpp:(.text+0x758): undefined reference to `__aeabi_idivmod'
```

(ignore memset for now, we'll get to that later)

__aeabi_uidivmod and __aeabi_idivmod are of course actual functions that implement a signed or unsigned division algorithm that doesn't seem to exist in any 32-bit ARM cores. Normally gcc would just copy them over at the link phase, but since we are explicitly linking with our own code only and nothing else, they're not available.

One could of course try to avoid division, but in this case, the stub would use it to play Viznut bytebeat, so we can't do without.

I spent quite some time trying to figure out a way to dig out those functions from libgcc.a, but gave up in the end. There's two main reasons why it doesn't make any sense. First, there's a mess of different division function blocks called from each other. Second, these are 'safe' functions. They check their input and call raise() if it's bad. If we're making a small intro, we don't need any of that.

It's better to just reimplement shitty versions of them. Digging the internet gives the specs for these functions:

* Numerator in r0.
* Denominator in r1.
* Return quotient in r0 (this is standard aeabi return register).
* Upon return, r1 must be filled with remainder.

Looking at those rules, we can make the shittiest division in the history of programming to get to try this thing:
```
unsigned __aeabi_uidivmod(unsigned num, unsigned den)
{
  // r0 and r1 already correct here due to aeabi calling convention
  int quotient = 0;

  while(num > den)
  {
    num -= den;
    ++quotient;
  }

  volatile register int r1 asm("r1") = num;
  asm("" : /**/ : "r"(r1) : /**/); // output: remainder
  return quotient; // r0
}
```

So we replace the step Concatenate the header stub and intro together. with:

* Make an ELF object of the ELF headers (yes, this is possible).
* Gather all missing symbol code into one source file, compile it.
* Make an ELF object file of the intro content and imaginary .bss.


Linking these three together gives us an object file from which we can step through the rest of the steps, and produces a real, working tiny Raspberry Pi intro executable:
```
Wrote 'src/intro': 1782 bytes
```

The size is kind of depressing though. Identical stub displaying the same raycast sphere and playing the same Viznut bytebeat is 1100 bytes on FreeBSD-amd64 and 1040 bytes on FreeBSD-ia32.

It's obvious having to do division manually as an algorithm is increasing that size a bit, but part of the problem lies in the current Raspberry Pi distributions. On normal desktop FreeBSD/Linux, when you want a window with a GL context and mouse cursor hidden, that's 3 calls. On Raspberry Pi, SDL won't do that for you, and to have the same result, you need 12 calls. Additionally it's not only parameters to functions, you need to have structures initialized to set values passed to the egl/VideoCore calls... very much as if one tried to manually make a GLX display context. Slow, tedious, and quite a lot of bloat.

(that memset actually comes from right here, probably something stupid in my code, but ld decides it's better off inserting a memset than using the assembly verbatim)

--

All in all, this means 4k is very much possible, but as things stand, 1k seems way out of reach. Even if Yellow Rose established SDL as the one de-facto library accepted for small intros on *nix, the one included in the most popular RasPi distro (Raspbian) does not automate anything.

When we made My Mistress the Leviathan for a platform that is very similar in performance to Raspberry Pi 2B, we included SDL2 .deb for Odroid C1+ in the release. This is because at the time of development, dependency hell in Hardkernel repository would remove Mali OpenGL driver if you installed their SDL2 (package built from their exact sources worked just fine tho). Even given the sorry state of any 'official' SoC distributions, of which Raspberry Pi community is definitely the best one, pushing this kind of custom distribution stuff goes counter to the "it-has-to-run-with-what-comes-in-the-distribution" spirit.

If anyone has any info on sidestepping the massive egl/videocore bloat on RasPi I'm listening.
