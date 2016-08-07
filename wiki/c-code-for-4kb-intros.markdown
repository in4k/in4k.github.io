---
title: "C code for 4kb intros"
layout: "wiki-page"
---

## Intro

You see a lot of discussion of compression, algorithmic design and PE header munging. However there is very little on how to code good tight C. Infact a search around the internet reveals very little information at all on this subject. Yet in experiments, I've found that I can save ~20% of compressed code space by writing C code specifically designed to compile to small assembler. Also in practice I have found that example source code demos do not follow such rules so there is probably a lack of knowledge about such things.

Firstly, four golden rules.

## Goldens

*   Compilers are dumber than you have been taught

Actually the truth is that something that seems obvious to a human is not obvious to a machine. Where possible do the optimisation yourself (yes, really).

*   Compilers are designed for fast code, not small code.

There may be a flag to optimise for small code but its not where the compiler writers put most of their efforts. You need to do the optimisation yourself (yes, really).

*   Simple C != small binary

Get used to life becoming more complex. No more For loops. 2D arrays are a bad thing. No more sloppy code, use every declarative construct (except register :-) that you can.

*   You can always make it smaller

Not strictly true, and the law of diminishing returns kicks in, but hold to this principle with blind faith and the code size will miraculously drop and drop and drop. You will know when its not going to go any further but believe me its a long way from the code you first wrote.

## C Rules (but doesn't Rock)

Everything down here has been learnt when using GCC. VC++ may be smarter or have different rules so be careful.

*   Declare functions as static where possible

```
static void myFunc (void) {};
```

*   Declare variables as constant where possible

```
constant float myarray[]={0.0,1.0,0.0,17.0};
```

*   Use predecrement/increment not post especially in loops

```
--n; // NOT n--;
```

*   Use do loops.

```
do {} while (--n); // NOT for loop or while loop
```

is the best sort of loop to write. One exception is that these two are the same size:

```
for (i=0; i<10; i++) {}
```

```
n = 10;
do { } while (n--); 
```

*   Flatten 2d arrays to multiple 1d arrays

Indexing a 2d array introduces a multiply. 3d are worse.

```
a[3][3]=b[3][3]*d[2][2]; // This is VERY BAD
```

*   Register is ignored by GCC and VC++

```
register int i;  // this has NO EFFECT
```

*   Do not use shorts except when passing directly to DLLs (eg OpenGL)

Anything done with shorts produces longer code than with ints. The exception would be calling glColor4ubv with stored byte variables (or even one integer :-)

*   Code to copy one array into another should be written:

```
do { *p++ = *q++ } while (--n);
```

(unless you use memcpy of course)

*   Move lines of code which use the same variables together

This reduces the load/stores as the compiler can keep a variable in a register

*   Move lines of code which statically assign variables together

This allows the compiler to assign one area of memory for all your variables in one go, not multiple leading to much shorter code

*   Move constants together in a single calculation

The compiler may not spot that two constants can be combined unless they are moved together in the code. This means making them the same order of precedence of course too.

*   Chain assignments

```
p=q=r=s=0.0;
pos+=vel+=acc;
```

This allows the compiler to load a constant value to a register just once.

*   Where possible, statically assign values to variables at declaration time.
*   Use strength reduction techniques e.g.

```
if (a^b)...; // Not if (a==b)...
a &= ~b;     // Not a%=b; useful when b is a power of two only and a is monotonically increasing
a ^= a;      // Not a=0;
```

Go ahead look up strength reduction, there is a lot of compiler work on this and lots of tricks.

*   Unroll small loops

```
doSomething(3); // this is better than: 
doSomething(2); // i=3;
doSomething(1); // do {} while (i--);
```

The effect of this is variable but generally unrolling 2-3 times will be better than writing a loop, no matter how efficient your loop is.

*   If it needs to be said, pass pointers to structures, not the structures.
*   Remove as many control structures as possible. If and switch are prime space wasters.
*   Move array declarations to be at global scope. This means they are at a fixed address: far easier for the compiler to optimise than stack based variables (declared inside your function).
*   When specifying floating point constants be careful to explicitly say they are float. GCC seems to promote to doubles even when -Os is used.

```
x*=0.0008f; // not x*=0.0008;
```

## Coding for compression

I have discovered very little here but some rules are:

*   Use as few external function calls and different DLLs as possible. Even ordinal import has a 4 byte overhead for each new function.
*   Try to quantise your static data.

In other words if you have an array: {0.1, 0.2, 0.11, 0.3, 0.29} perhaps it will work just as well with {0.1, 0.2, 0.1, 0.3, 0.3} I saved 15 bytes once by altering one number!

*   Try to align similar data blocks in memory. Put ints together, put floats together. Now put similar ints together. Similar floats.
*   When you are really stuck, move lines of code so the logic remains but the order changes. You could save 1% this way.

### Improving compressibility by consistency

Something worth a note...

When trying to minimize the size of the compressed code, it is often useful to be consistent with the code. That is, choose to do the same things always with the same code, so that there's more repetition for the compressor to take advantage from.

When coding in asm, this means things like "always use the same register for the same purpose". However, with C you tend to lose the control because C compilers usually just allocate the registers in a predefined order. Some compilers, however, let the programmer give hints on what registers to use, and you can put these hints consistently into your source code.

An example with GCC and x86:

```
#define isCtr asm("ecx")
#define isAcc asm("eax")
```

```
register int i isCtr;
i = 256;
do {
   register int a isAcc;
   a = *s++^p;
   *d++ = a;
} while (--i);
```

Note: with GCC and x86, registers other than eax, ecx and edx will be saved with separate stack pushes, so you may want to recheck all manual allocations to registers other than these.

Checking all compiler optimizations individually also helps to improve compressibility. For example, the GCC flags -O2 and -Os activate the "schedule-insns" optimizations that reorder the instructions and therefore break the consistent chunks you have carefully built. (In the example code above, the incs get separated from the related memory reads and writes when using these optimizations). This may be one of the reasons why -O1 has been noted to be better than -Os.

--viznut

## Some OpenGL Tricks

*   In OGL when specifying rotations, remember glRotatef (45,1,0,0) is equivalent to (45,45,0,0) which should compress better. More effective still is when rotating (45,1,1,1), change this to (45,45,45,45) for better compression.
*   glColor4ubv requires an array of 4 bytes to set a colour. Under most compilers you can pass the address of an *integer* to this. This leads to several tricks:
    *   Change a colour using a simple bit operation eg colour = ~colour;
    *   You can pass the address of a fucntion to a glColor4ubv avoiding storage of colour data altogether!
*   gluPerspective is just a matrix multiply. Although in texts, it is always applied to the GL_PERSPECTIVE matrix stack, it can be applied safely to the GL_MODELVIEW stack after glLoadIdentity(). This saves all those pesky glMatrixMode calls.
