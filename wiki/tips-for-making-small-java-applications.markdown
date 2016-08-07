---
title: "Tips for Making Small Java Applications"
layout: "wiki-page"
---

Java application is delivered as a .jar file. The .jar file is really a zip file, containing all the class files (compiled bytecode), any resources (like images and other data) and optinally a manifest file (telling the jvm the main class).

Each .java file is compiled into one or more .class files. Each class and interface constitutes a new .class file (including inner classes). Each class file has is roughly divided into two parts. The first part contains the runtime constant pool and the other part contains the code. The constant pool takes a big chunk part from the .class file. It contains all constants used in the class, including all referenced classes, functions, strings, big integers and floating point numbers. Every method and class is referenced to by their name. Every method defined has their name included.

When writing small code, it's important to keep the number of .class files small. First of all, this reduces the amount of cross referencing between the classes, and the number of unique references to system classes goes down. This helps to reduce the size of the runtime constant pool considerably. Also every function and variable costs a little in terms of bytes. There's always a couple of constant pool entries and the function definition itself. So limiting the number of functions is also good.

The java compiler outputs debug information by default to the generated class files. A Java obfuscator is a program that reads in a bunch of .class files, mangles the classes (removing/adding information, changing method and variable names etc) and outputs a new .jar file with the same functionality. Most obfuscators can make the function and variable names as short as possible (1-2 letters long), reducing the size of the constant pool. They can also remove the debug information.

The Suns Java compiler can optimize the Java code a little. It can inline the use of "static final" constants (and the obfuscator removes these from even appearing in the class file. The compiler also removes dead code, so that "if (false) { ++i; }" does not compile into any bytes.

If you use any custom data, keep the filenames as small as possible, since they are referred to by string in from the Java code.

*   See also: [Java 4K Wiki](http://wiki.java.net/bin/view/Games/4KGamesDesign "http://wiki.java.net/bin/view/Games/4KGamesDesign") tips for squeezing games into 4K (should be perfectly good reading for making 4K demos as well!).

*   Sami Koivu's tips for minimizing bytecode size contain a few points less commonly known
    *   [http://slightlyrandombrokenthoughts.blogspot.com/2007/05/minimizing-java-bytecode-size.html](http://slightlyrandombrokenthoughts.blogspot.com/2007/05/minimizing-java-bytecode-size.html "http://slightlyrandombrokenthoughts.blogspot.com/2007/05/minimizing-java-bytecode-size.html")
        *   Action: Strip any method throws information.
        *   Justification: The VM doesn't use this information. You can't tell the compiler not to create it, but it can be stripped afterwards.
        *   Action: Rearrange local variables, putting the four that are the most used first. Heavily reuse these variables.
        *   Justification: Instructions that refer to the local variables 0-3 take up one byte and instructions that refer to the rest of the local variables take up two bytes.
        *   Action: Set the scope of the local variables so that only 4 variables are in scope at any given time.
        *   Justification: Two variables, even if they're of different types, can be stored in the same local variable "slot" if their scopes don't overlap. And why using the first four local variables is good is explained in the previous item.

# Tools

Some useful tools for making the .jar file smaller.

*   Better zip packers
    *   KZip packs zip files (which jar files are) much better than pkzip. The url contains also Pngout, a tool for packing png files as good as possible.  
        URL: [http://advsys.net/ken/utils.htm](http://advsys.net/ken/utils.htm "http://advsys.net/ken/utils.htm")

*   Java obfuscators
    *   ProGuard  
        URL: [http://proguard.sourceforge.net/](http://proguard.sourceforge.net/ "http://proguard.sourceforge.net/")
    *   RetroGuard (free for non-commercial use)  
        URL: [http://www.retrologic.com/retroguard-main.html](http://www.retrologic.com/retroguard-main.html "http://www.retrologic.com/retroguard-main.html")
