---
title: "Win32"
layout: "wiki-page"
---

# Frameworks

* [iq's 1k/4k framework](https://github.com/in4k/isystem1k4k)
* [hitchhikr's 1k framework](https://github.com/in4k/1K-D3D-SW-OGL-FrameWorks)
* [Prismbeing's intro system](https://github.com/armak/pbr-introsystem) - 4klang + pixelshader framework
* [Psycho's DirectX11 1k/4k framework](https://github.com/psycholns/TinyDX11)
* [Auld's OGL Framework](aulds-ogl-framework) - For doing cool stuff with OpenGL on Win32.
* [Auld's 1k Framework](aulds-1k-framework) - If 4 KB isn't challenging enough for you, do it in 1 KB!
* [Drawing Pixels](drawing-pixels) - Small win32 framework

# Pixelshader / Raytracing references

* [http://shadertoy.com](ShaderToy)
* [http://mercury.sexy/hg_sdf/](A glsl library for building signed distance functions by Mercury)

# Introduction to Win32 as 4 KB Platform

Win32 is the most common platform currently for making 4 KB intros. This doesn't mean that it would be the best or the easiest one. But it is the most typical OS installed on computers, so it is what most people are currently writing for.

On Win32 you basically create your intro using either DirectX or OpenGL, which you can pretty much trust both being available. Currently most used compiler is probably Microsoft Visual C++ (version 6 or newer). If you are lazy and don't want to deal with throwing off C runtime library, you can link dynamically against the C runtime library MSVCRT.DLL, which is available basically on all Windows computers. However, it is of course recommended that you don't link at all against the C runtime library because you can save space that way.

_THIS TEXT IS SOMEWHAT DEPRECATED, MOST PEOPLE USE PIXELSHADER RAYMARCHING ON THEIR INTROS NOWDAYS, SOMEONE FIX THIS PLEASE_

## Note About Compatibility Issues

As you will end up doing multitude of tricks to reduce the size of the executable, it is very likely that you will select some size optimization strategies which may affect compatibility of the final executable on different versions of Win32 platform. Because of this it is very recommended that from start of the project you construct the code for building two different revisions. First one is the "compo" version, named by the fact that it's usually meant as a contribution to an intro competition at some event. Size of the compo version is tuned to the lowest possible amount to fit in the given limit. The another version is the "safe" version.

The safe build could be just unpacked version of the compo one, because sometimes the executable compression itself makes the worst incompatibilities. But your code may as well have some incompatible hacks - which you should know when you're making one. Easy way to do this is to use #define macros and #ifdefs to differentiate upon compile time which build you are doing. You will probably want to make additional debug builds of both the compo and safe build as well.

Note that it is strongly recommended, that you refrain yourself from making the intro to work only on competition computer (and providing safe build for all others). The safe build should be only there for making the intro work also on older versions of Windows OS, e.g. if your tricks are compatible only with Windows XP (and maybe Windows 2000). Do not use "safe" build as an excuse to completely sacrifice compatibility between different computers!

* * *

**See also:**

*   [Sample Sources](sample-sources) for examples on how to get started.
*   [Import by hash](import-by-hash) for a commonly used method to import dll functions.
