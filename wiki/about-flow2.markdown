---
title: "About Flow2"
layout: "wiki-page"
---

## Flow2: OGL+C+Shaders+EXE

This was the first C 1k exe to use GLSL.

You can see the intro at [Pouet](http://www.pouet.net/prod.php?which=30589 "http://www.pouet.net/prod.php?which=30589").

### Source

```
// Chris Thornborrow (auld/sek/taj/copabars)
// If you use this code please credit...blahblah
// Example OGL + shaders in 1k
// Requires crinkler - magnificent tool
// VS2005 modifications by benny!weltenkonstrukteur.de from dbf
//    Greets!
// NOTE: DX will beat this no problem at all due to OpenGL forced
// to import shader calls as variables..nontheless we dont need
// d3dxblahblah to be loaded on users machine.
```

```
#include <windows.h>
#include <GL/gl.h>
#include "glext.h"

// NOTE: in glsl it is legal to have a fragment shader without a vertex shader
//  Infact ATi/AMD  drivers allow this but unwisely forget to set up variables for
// the fragment shader - thus all GLSL programs must have a vertex shader :-(
// Thanks ATI/AMD
```

```
// This is pretty dirty...note we do not transform the rectangle but we do use
// glRotatef to pass in a value we can use to animate...avoids one more getProcAddress later
const GLchar *vsh="\
varying vec4 p;\
void main(){\
p=sin(gl_ModelViewMatrix[1]*9.0);\
gl_Position=gl_Vertex;\
}";
```

```
// an iterative function for colour
const GLchar *fsh="\
varying vec4 p;\
void main(){\
float r,t,j;\
vec4 v=gl_FragCoord/400.0-1.0;\
r=v.x*p.r;\
for(int j=0;j<7;j++){\
t=v.x+p.r*p.g;\
v.x=t*t-v.y*v.y+r;\
v.y=p.g*3.0*t*v.y+v.y;\
}\
gl_FragColor=vec4(mix(p,vec4(t),max(t,v.x)));\
}";
//p.g*3.0*t*v.y+i;\
```

```
typedef void (*GenFP)(void); // any function ptr type would do
static GenFP glFP[7];
const static char* glnames[]={
     "glCreateShader", "glCreateProgram", "glShaderSource", "glCompileShader", 
     "glAttachShader", "glLinkProgram", "glUseProgram"
};
```

```
static void setShaders() {
int i;
  // 19\. April 2007:	"(GenFP) cast" added by benny!weltenkonstrukteur.de
  for (i=0; i<7; i++) glFP[i] = (GenFP)wglGetProcAddress(glnames[i]);
  GLuint v = ((PFNGLCREATESHADERPROC)(glFP[0]))(GL_VERTEX_SHADER);
  GLuint f = ((PFNGLCREATESHADERPROC)(glFP[0]))(GL_FRAGMENT_SHADER);	
  GLuint p = ((PFNGLCREATEPROGRAMPROC)glFP[1])();
  ((PFNGLSHADERSOURCEPROC)glFP[2]) (v, 1, &vsh, NULL);
  ((PFNGLCOMPILESHADERPROC)glFP[3])(v);
  ((PFNGLSHADERSOURCEPROC)glFP[2]) (f, 1, &fsh, NULL);
  ((PFNGLCOMPILESHADERPROC)glFP[3])(f);
  ((PFNGLATTACHSHADERPROC)glFP[4])(p,v);
  ((PFNGLATTACHSHADERPROC)glFP[4])(p,f);
  ((PFNGLLINKPROGRAMPROC)glFP[5])(p);
  ((PFNGLUSEPROGRAMPROC) glFP[6])(p);
}
```

```
// force them to set everything to zero by making them static
static PIXELFORMATDESCRIPTOR pfd;
static DEVMODE dmScreenSettings;
```

```
void WINAPI WinMainCRTStartup()
{
  dmScreenSettings.dmSize=sizeof(dmScreenSettings);		
  dmScreenSettings.dmPelsWidth	= 1024;
  dmScreenSettings.dmPelsHeight= 768;
// 	  dmScreenSettings.dmBitsPerPel	= 32;
// its risky to remove the flag and bits but probably safe on compo machine :-)
  dmScreenSettings.dmFields=DM_PELSWIDTH|DM_PELSHEIGHT;
  ChangeDisplaySettings(&dmScreenSettings,CDS_FULLSCREEN);  
  // minimal windows setup code for opengl  
  pfd.cColorBits = pfd.cDepthBits = 32;
  pfd.dwFlags    = PFD_SUPPORT_OPENGL | PFD_DOUBLEBUFFER;	
  // "HDC hDC" changed 19\. April 2007 by benny!weltenkonstrukteur.de
  HDC hDC = GetDC( CreateWindow("edit", 0, WS_POPUP|WS_VISIBLE|WS_MAXIMIZE, 0, 0, 0, 0, 0, 0, 0, 0) );
  SetPixelFormat ( hDC, ChoosePixelFormat ( hDC, &pfd) , &pfd );
  wglMakeCurrent ( hDC, wglCreateContext(hDC) );
  setShaders();
  ShowCursor(FALSE); 
   //**********************
   // NOW THE MAIN LOOP...
   //**********************
   // there is no depth test or clear screen...as we draw in order and cover
   // the whole area of the screen.
   do {
        //dodgy, no oglLoadIdentity- might break...
        // change the first number to alter speed of intro...smaller is slower
        // this is the fast version
        glRotatef(0.3f,1,1,1);
        // draw a single flat rectangle on screen...
        glRecti(-1,-1,1,1);
        SwapBuffers(hDC);
   } while ( !GetAsyncKeyState(VK_ESCAPE) );   
}
```

### Why

The bugger in OGL is three fold:

*   glsl isnt exactly compact
*   You need to import shaders as extensions under windows
*   Todays ATI drivers force you to have a vertex shader to get correct operation of pixel shaders

This adds bytes and bytes compared to DX which is definitely the better choice at 1k for shaders. Nontheless OpenGL works everywhere and doesn't have that "you need dxsquiggle49_XXYY.dll" annoyance. So getting C code with glsl shaders compiled with crinkler into an EXE is still significant.

### Some of the tricks

Lets see. In the main code I use glRotatef - not to rotate anything but to pass a value which can be used for animation within the shader. This avoids having to import functions to pass variables into the shader. I dont need to set up the matrix with glLoadIdentity() as rotates are cummulative - meaning whatever the initial value in the drivers for the matrix, I'll get the desired result of a small incremental change each step.

A single rectangle is drawn that covers the screen. Normally fragment (pixel) shaders use texture co-ordinates to determine how to draw, but I used gl_FragCoord. This means I can use a single call to glRect without needing to set up texturing.

No clear screen: our rect covers every pixel on the screen.

In the vertex shader, I extract a row of the transformation matrix and use this to control the animation in the fragment shader. Notice how the gl_Position is simply the input vertex, not the more usual ftransform() to avoid the glRect spinning around.

The pixel shader has a loop and the counter in the loop must be very low. PS2.0 cards do not really implement loops. They simply unroll loops (which is why they cant cope with variable length loops). In unrolling you might hit the maximum instructions possible in a shader (defined as 256 I think in ps2.0 but varies from card to card). This is why the loop counter must be small. The correct way to implement such a function in PS2.0 would be a so-called ping-pong buffer but I dont have space.

### Crinkler v apack/dropper

Crinkler is used for compression. Crinkler needs ąbout 400 bytes of space for its depacking routine. The code here compiles smaller with apack and dropper 2.0 but the advantage of crinkler is that it gives an exe, not a com. This is the new fashion for 1ks and soon, in the same way com files went out of fashion at 4k, they soon will at 1k too.

One interesting thing is that people shout and scream that import by ordinal should not be used. Luckily authors of crinkler know better and allow you to specify when import by ordinal is used and when not (I always wanted a version of dropper that did this). From the crinkler documentation:

```
/RANGE:[DLL name]
   Import functions from the given DLL (without the .dll suffix)
   using ordinal range import. Ordinal range import imports the first
   used function by hash and the rest by ordinal relative to the
   first one. Ordinal range import is safe to use on DLLs in which
   the ordinals are fixed relative to each other, such as opengl32 or
   d3dx9_??. This option can be specified multiple times, for
   different DLLs.
```

(Mentor, blueberry, hope you dont mind this being quoted here). Nontheless, I get almost no advantage here as I have just two opengl32.dll calls that arent imported via wglGetProcAddress.

### VC++ v GCC

The code is VC++2005 Express. Compiling with this, not GCC saves around 60 bytes. After nearly 2 years of working with GCC I have now abandoned it in favour of VC++. It really does generate smaller code in most cases.

This means my old GCC 1k framework is to be replaced by Rbrazs version of this for VC++. (You can find both on this site).

### Conclusion

With some tight programming its possible to get OGL, C, shaders and an EXE in 1k. I'm looking forward to a lightweight version of crinkler for 1k.

### Credits

Its a long story but I made the 1k framework (without shaders) with the help of some people from this board. I then extended it to include shaders then Benny of DBF made it work in VC++ with the help of Rbrazs files ( I think). Meaning there are a lot of people to credit for this 1k besides me. Oh yeah and it doesnt compile to 1k without crinkler - thanks to Mentor and Blueberry.
