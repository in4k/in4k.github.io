---
title: "Auld's Noise Algorithms"
layout: "wiki-page"
---

This page is here to promote discussion of Noise in 4k demos. Please use the discussion page to add your own links/comments/algorithms.

This page is only a beginning - the subject is huge. At the same time I want to immediately offer one full implementation.

### What is Noise?

Useful noise is _not_ random numbers. There nothing wrong with textures of random numbers but lets be clear, its not noise. Useful noise has several properties:

*   Its repeatable. For any input value the same random value is always returned.
*   It has coherency. The change in random value from point to point is not random but smoothed in some way.

_add some references_

### What is Noise Useful For?

If you don't know yet, any good noise function can be used for clouds, stucco, water, landscapes, wood, marble and a variety of natural phenomena. In addition noise can be used to alter animations, line drawings, music and any calculation that requires small variations from the deterministic value that require a more natural look/sound. For example, a character standing still can be animated slightly using noise to make them look like they are fidgeting.

_add some references_

### Perlin Noise

Perlin noise is the best known noise function. Some people mistake Perlin noise for noise itself. It is not but it is one of the most widely used in computer graphics and perhaps there is a very good reason for this: it works very, very, very well for natural phenomena.

### Typical Noise Algorithm

*   Define a function that returns random values within a given range and take as input a seed. For any given seed the random function always returns the same value.
*   For each octave of noise
    *   Reduce the amplitude of the noise function
    *   Increase the frequency of the noise function
    *   Add new octave values into interpolated average of prevous surrounding octave values.
*   Smooth the resulting noise

What is an octave? An octave is a set of random numbers with the same amplitude and frequency. Summing octaves of noise is the key to obtaining natural looking effects. This is very hard to explain without images and as they can't be uploaded here, I'll simply reference another web page which explains octaves rather well...

_add reference_

## Possible Ideas for 4k Noise

Firstly I will be using OpenGL. At the time of writing the Noise function in OGL shaders does not work. It is still therefore recommended to create Noise textures.

The basic idea is as follows. Render successive octaves of texture, formed from random numbers, on top of one another, blending until we have a single noise texture on the screen. Then we will capture this into a texture and use it to render noise. The results are visually very good.

### Repeatability

We will store our resulting noise in a texture. This means for any given Noise(x,y) the result is repeatable.

### Interpolation

In some theory, linear interpolation of random values for noise textures is shunned. It is claimed to give bad results because the results are rotationally variant. For most purposes this is frankly rubbish. **Contrary to what the books say, linear interpolation is good enough.** In any case, other forms of interpolation are rotationally variant when done on a square grid of points anyway, its just less visible. Most people writing this rubbish have no idea what they are talking about.

In order to interpolate each octave (the most expensive part of a noise generation function), we will use texture hardware. Put simply, we will draw each octave of noise as a textured polygon instead of using cosine/cubic interpolation, array indexing, averaging and addition. Texture hardware uses linear interpolation of course.

### Increasing the frequency

As we are rendering textures, we will simply double the resolution of textures each octave to double the frequency. This works because if we render a texture with more and more detail in extactly the same screen space, we get a higher frequency of random numbers across the screen.

In practice the code below actually halves the frequency each loop but the effect is the same.

### Adding Octaves

Now we have decided to use texture hardware to render octaves it should be obvious that using blending to add octaves is easy. We will create an octave of noise, create a texture from it and then draw a polygon using that texture. The polygon will be 0.5 alpha and we will belnd it with the previous polygons rendered until we blend all octaves. This way we avoid calculations for adding octaves.

### Reducing Amplitude

Alpha blending means we do not need to worry about amplitude of the octave any more. We can use the full range of random values at each octave. But why? Each blend will effectively reduce the contribution by a previously rendered octave. To be specific, using:

```
  glEnable (GL_BLEND);
  glBlendFunc (GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
```

together with an alpha of 0.5 means that the first octave rendered on a polygon will be reduced in intensity (amplitude) 2*number of octaves. This is why we render the higher frequency octave first: it should have the lowest amplitude. Normally noise octaves reduce the amplitude and double the frequency. Our blending and texture resolution will substitute.

```
Blending IS summing octaves AND reduction of amplitude
Texture resolution IS frequency
```

### Smoothing

Many noise generation algorithms recommend smoothing of the noise function. For example Perlin uses interpolation to smooth between noise values. In order to smooth the noise we will render our octaves of noise at resolutions lower than the screen area covered by the polygon. This means that OGL will interpolate values *before* we capture the final noise texture. This inherently smooths the noise. In other words, we do not have an explicit smoothing step.

## C code for An OpenGL Noise Texture

**As usual, if you use the code in part or whole below please gives credits or greets. Thanks.** Its a lot of work to come up with these small pieces of code.

```
GLubyte rarray[256*256];
```

```
static void captureNoiseTexture (const uint texid, const uint per, 
                                uint noct, GLubyte **pix)
{   
uint s=512;
uint i;
  // enable blending
  glEnable (GL_BLEND);
  // set standard blending function
  glBlendFunc (GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
  // persistance in alpha
  glColor4ub (255,255,255,per);
  do {
     // we could use s*s but its more bytes...
     i = 256*256;
     // an array of random values...
     while (i) {rarray[--i]=rand();}
     // halve the resolution of texture each octave
     s >>= 1;
     newTexture (1, 1, s, GL_LUMINANCE, &(rarray[0]));
     drawCircle();
  } while (--noct);
  // now capture what we drew and put it into a texture
  captureArea (GL_RED, 256, 256, pix);
  newTexture  (texid, 1, 512, GL_LUMINANCE, *pix);
  glDisable (GL_BLEND);
}
```

## An Explanation of Key Features

### Additional Functions

Its probably fair to assume most 4k demo coders have their own create texture and capture texture functions but if you want to see mine, here they are:

```
static void captureArea (const uint format, const uint x, const uint y, GLubyte **pix)
{
   *pix = (GLubyte*) calloc (512*512, sizeof(GLubyte));
   glReadPixels (x,y,512,512,format,GL_UNSIGNED_BYTE,*pix);
}
```

```
static void newTexture (const uint tn, const GLint ncomp, const GLint s, const GLenum format, 
                        const GLubyte *pix)
{
     glBindTexture (GL_TEXTURE_2D, tn);
     gluBuild2DMipmaps (GL_TEXTURE_2D, ncomp, s, s, format, GL_UNSIGNED_BYTE, pix);       
}
```

These are useful in other contexts so I separate the code.

### Whats the Per input ?

Per is used as the alpha value. Its a cheat. It sort of qualifies as persistance. The idea of persistance is complicated and not well defined. Think of it as how rough the resulting surface is. A value between 1 and 255 can be used. I recommend 120-130 for most cases. <100 gives rougher and more random surfaces.

For fog/clouds:

```
captureNoiseTexture(1,120,7,&tex1);
```

For gravel:

```
captureNoiseTexture(1,120,2,&tex1); 
```

For rough stone:

```
captureNoiseTexture(1,88,4,&tex1);
```

### Why Draw a Circle?

There are no cubes or squares in OpenGL. In GLU we have a disk which can draw a circle. I use this to draw textures because the surface is flat of course and calling a GLU function is much cheaper than defining your own quad. Here is the exact call if you are interested:

```
gluDisk (sph, 0, 1, 16, 1);
```

Its the cheapest possible flat surface in OGL...unless you know otherwise.

### Why odd parameters for capturing the texture?

I am working at a certain screen resolution. This glReadPixels fits nicely within the circle. You may need to use other parameters.

### Why Capture Red?

Capturing Luminance does not work as we are rendering R,G,B with the same value but alpha is different. Luminance is the average of the above values so unexpected things happen.

Trick: try capturing luminance, not red channel for a ready to use cloud texture!

## Why Write Noise as OpenGL?

OK so that proves you can write a correct noise function in OGL exploiting blending and texture hardware and readpixels. But why bother? Several reasons:

*   There are a lot of bad noise functions in demos. Good enough for some detail or for swirling and alpha blending but not good enough to do clouds/landscape and so forth. The one above is.
*   I suspect, but don't know for sure, that optimising a C version of Perlin would still result in more bytes. The awful array accesses and averaging would result in large byte count when compiled.
*   Crinkler. Crinkler is already and will continue to be the best optimising tool for demos. It includes a special import mode (ranged hash) for OGL which reduces OGL function importing to just over a byte per function. 

Therefore forcing more and more of the logic into OGL function calls can only be a good thing. In the above code 90% of the code is actually calls to OGL and thus will benefit hugely from Crinklers compression.

## Limitations

Extending this to be a 3d texture is difficult. 3d is useful for time based animation of clouds for example. Fundamentally we only capture 2D textures. So for now the limitation is that we can only use this for static noise textures.

The texture does not wrap.
