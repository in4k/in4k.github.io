---
title: "Graphics"
layout: "wiki-page"
---

# Shadertoy / Raymarching

Single quad pixelshader effects have become common in 1kb and 4kb intros. You raymarch your entire scene inside the pixelshader.

The most common tool to do single quad pixelshader effects is [Shadertoy](http://shadertoy.com).

Here are some resources for learning raymarching and graphics coding in shadertoy:

* [iq's formulanimations tutorial series](https://www.youtube.com/playlist?list=PL0EpikNmjs2CYUMePMGh3IjjP4tQlYqji)
* [Mercury's Signed Distance Functions Library](http://mercury.sexy/hg_sdf/)
* [Shadertoy Hackaton @ NVScene 2015](https://www.youtube.com/watch?v=jjU3rO36zCs)

# 4K Procedural Graphics

A relatively recent trend of democompos has been the 4kb procedural graphics executable compo. Where the executable under 4 kilobytes creates a single _still_ image procedurally.

If you can find a list of those entries on pouet:

 * [link](http://www.pouet.net/prodlist.php?type%5B0%5D=procedural+graphics&page=1&order=thumbup)

Some great information on creating procedural graphics can be found in iq's web site:

* [http://iquilezles.org/www/](http://iquilezles.org/www/)

And here's a great tool helping creation using GLSL:

* [4K Procedural GFX Monitor](http://pouet.net/prod.php?which=52974) by SystemK

# Auld's OpenGL Tricks Corner

* [Auld's Mega Small (Almost Free) Geometry](aulds-mega-small-almost-free-geometry) using OpenGL Utility library (glu32).
* [Auld's OGL Framework](aulds-ogl-framework)
* [About Flow2](about-flow2) - Using OpenGL with GLSL shaders in EXE to make an 1K intro.

# Tiny Geometry Tools

* [Qoob demoscene modeler](http://qoob.weebly.com/)

# Using Graphics APIs with Assembly

A hard part of tight graphics coding is getting the API to work with an assembler. Here are some resources.

## DirectX with Assembly

* Scronty Soft - DirectX 8
    * Includes: http://www.scrontsoft.com/Scrontsoft.asp?pageID=1
    * Tutorials: http://www.scrontsoft.com/Scrontsoft.asp?pageID=2
    * Examples: http://www.scrontsoft.com/Scrontsoft.asp?pageID=3
    * Entire Package, Locally: [d3d 8.0 from Scrontys site.rar](http://in4k.untergrund.net/direct%203d/d3d_8.0_from_Scrontys_site.rar)
* Scronty Soft - DirectX 8.1
    * Includes: http://www.scrontsoft.com/DX81.asp?pageID=1
    * Tutorials: http://www.scrontsoft.com/DX81.asp?pageID=2
    * Examples: http://www.scrontsoft.com/DX81.asp?pageID=3
    * Entire Package, Locally: [d3d 8.1 from Scrontys site.rar](http://in4k.untergrund.net/direct%203d/d3d_8.1_from_Scrontys_site.rar)

## OpenGL with Assembly

* Article from Hugi Issue 26
    * URL: http://www.hugi.scene.org/main.php?page=hugi26
    * Locally: [hugi 26 - coding corner graphics polaris opengl with assembly in 4kb.htm](http://in4k.untergrund.net/html_articles/hugi%2026%20-%20coding%20corner%20graphics%20polaris%20opengl%20with%20assembly%20in%204kb.htm)

* [hitchhikr SoftWorks](http://franck.charlet.pagesperso-orange.fr/sources.html)
    * [hitchhikr’s 1k framework](https://github.com/in4k/1K-D3D-SW-OGL-FrameWorks)
    * [OpenGL in Assembly language](http://franck.charlet.pagesperso-orange.fr/Ogl_Asm.zip) - A set of 40 examples i did for the masm32 forum OpenGL section.

# Software Samples
* Sample ports of NeHe Content
    * URL: http://nehe.gamedev.net/
    * Locally: NEHE.rar
    * Includes:
        * Setting Up An OpenGL Window
        * Your First Polygon
        * Adding Color
        * Rotation
        * 3D Shapes
        * Texture Mapping
        * Texture Filters, Lighting & Keyboard Control
        * Blending
        * Moving Bitmaps In 3D Space
        * Loading And Moving Through A 3D World
        * Flag Effect (Waving Texture)
        * Display List
* MASM Wildfuse Open GL / and Glut Tutorials.
    * Masm version: Rudolph Pretorius aka nVRwere
    * URL: No longer exists
    * Locally: [WildFuse.rar](http://in4k.untergrund.net/open%20gl/WildFuse.rar)
    * Includes:
        * glmouse.zip
        * glsample.zip
        * glutmasmex1.zip
        * glutmasmex2.zip
        * glutmasmex3.zip
        * glutmasmex4.zip
        * glutmasmex5.zip
* From asm.yeah.net (no longer exists)
    * Locally: [Open GL Samples from asm.yeah.net.rar](http://in4k.untergrund.net/open%20gl/Open_GL_Samples_from_asm.yeah.net.rar)
    * Lots of stuff - such as:
        * bezier.zip
        * circle.zip
        * clock.zip
        * clover.zip
        * digital-clock.zip
        * directdraw.zip
        * directflip.zip
        * grays.zip
        * kaleido2.zip
        * linedemo.zip
        * lover.zip
        * mirrors.zip
        * opengl-cube.zip
        * opengl-fullscreen.zip
        * opengl-texture.zip
        * opengl-torus.zip
        * opengl_nasm.zip
        * orthog-circle.zip
        * randrect.zipr
        * banding.zip
        * scramble.zip
        * shapewindow.zip
        * sinewave.zip
        * sprite.zip
