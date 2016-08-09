---
title: "About Hive"
layout: "wiki-page"
---

Hive is a 1k, my sixth.

## First The Good

Hive has some serious respect from some size coders, which means I'm getting better. It manages to be the first 1k ever I believe to have graphics synchronised with music. In all the features crammed into the 1k are quite impressive:

*   Midi sound
*   Music
*   3d "deep room" graphics
*   Movement in 3 space
*   Graphics synched with music
*   Multithreading with priorities
*   Some safety checking on midi devices
*   Complex geometry
*   Textures and Alpha blending

Overall its a step up for me I think. I watch one of hitchs demos a lot before writing this. Please have a look at [http://www.pouet.net/prod.php?which=17350](http://www.pouet.net/prod.php?which=17350 "http://www.pouet.net/prod.php?which=17350"). Its very nice for a 1k. One of the things people like is the feeling of depth in there. This huge moving landscape in 1k is very impressive.

## Now the Bad

Its fair to say the music is moody but its very primitive. The graphics are great for the size but "ugly" for most people who know nothing about 1k. No subtle lighting or shading, just throbbing colours.

## The Tech

C, GCC, OGL. My usual. This time I gave up on a real synth and went back to midi. I think this is optimal and acceptable at 1k. The tech is all in the sound.

## The Music

I did a few evenings research on algorithmic music before writing this. In general the stuff sounds awful, but one method caught my attention.

[http://reglos.de/musinum/](http://reglos.de/musinum/ "http://reglos.de/musinum/")

I wrote what I think is a minimal implementation of the algorithm described here and included that in the code. The music isnt great but its clearly not random and has those lovely fractal properties of real music. I'm convinced with a more sophisticated use of instruments something very nice could be included in 1k...

Maybe its interesting to note (no pun intended) that I play every note. Correctly done the algorithm would identify the same note twice and just not play the note again. It sounds better like this but I was out of bytes.

## The Sound

Midi. In order to keep the sound as consistent as possible I kick off the midi in a new thread, with high priority. You can see this in the code below. Without this the music was simply too variable in speed and unacceptbale.

## The Design

Like I said I was inspired by hitchs work and my only goal here was to create a depp landscape feel like he had managed. The grid is actually composed of gluCylinders, slightly overlapping.

## Some Experimenting with ASM and C

We tried making the code smaller by recoding pieces in asm. Overall it looks like we could have made the code about 10% smaller. Thats 100 bytes. Well worth it in 1k. However, during experiments I found 30 more bytes in the C and later a few more. The jury remains out. Clearly asm is smaller, but by how much?

## The Code

```
//
// Hive
// 3d, texturing, midi sound, threads, music
// 
// Auld May 2006
//
#include <process.h>
#include <windows.h>
#include <GL/gl.h>
#include <GL/glu.h>
#include <math.h>
typedef unsigned int uint;
typedef unsigned char uchar;
static uint i,thump;
static GLUquadricObj *q;
static HMIDIOUT device; 
static uint col = 0x101080FF;

// Counts the number of bits in an unsigned integer
// Note clever trick ot terminate early by testing the integer
// to see if it is already 0, after shifting it
const static uint bitCount (uint n)
{
int count=0;
     do{
         // is there a 1-bit at least sig bit of n 
         count += n & 1 ;  
         // shift n
         n >>= 1 ;
     }  while (n);      
     return count ;
} 

static uint note;
// runs in a seperate thread
// addTune creates the music and makes midi clss to play it
static void addTune (void *p)
{
   do {
       // must sleep or this thread will just absorb all CPU!!
        Sleep(160);
        // set a useful instrument on midi channel 40...
        midiOutShortMsg(device, 0x27c1);//19 or 22 or 27
        // MAGIC! Keep incrementing note. Then count number of bits
        // This gives fractal like music with selfrepetiion! Amazing.
        note+=31;
        // thump is used here and also for synching in size of cylinders...
        thump =  bitCount(note); 
        // thump is used for the midi tone value.
        // It is shifted by 8 to get to right place in midi instruction 
        // and shifted by 3 to multiply tone up a bit
        midiOutShortMsg(device, (thump <<10)|0x00400091);
   }while(1);
   // we dont need to exit as when we exit the main process this thread is also killed.
}

// Here we capture a texture from the screen...thats all
// Unless we call glTexParameteri(, the texture is not diplayed properly
static void texture () 
{
   glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MIN_FILTER,GL_LINEAR);
   glCopyTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, 0, 0, 256, 256, 0); 
}

// Draws a cube of objects about the origin
// 2*loop +1 objects are drawn (-loop to loop) in each dimension
static void drawGrid (const int loop) 
{
int z,y;
   int x=-loop;    
   do {
     y=-loop;
     do {
     z=-loop;
        do {
             glPushMatrix();
             // the colour is a cheat, we cast an int to pretend its an array of 4 bytes
             glColor4ubv((uchar*)&col);
             glTranslatef(x,y,z);
             // SYNCH!!!!! Thump is used here from the music routine to synch the cylinder length
             gluCylinder(q, 1, 1, 0.05f*thump, 8, 8);
             glPopMatrix();
             z++;
        } while (z-loop);
        y++;
     } while (y-loop);
     x++;
   } while (x-loop);
}

void WINAPI WinMainCRTStartup()
{   
  uint g;           
  PIXELFORMATDESCRIPTOR pfd;  
  pfd.cColorBits = pfd.cDepthBits = 32; 
  pfd.dwFlags    = PFD_SUPPORT_OPENGL | PFD_DOUBLEBUFFER;	
  PVOID hDC = GetDC ( CreateWindow("edit", 0, 
                      WS_POPUP|WS_VISIBLE|WS_MAXIMIZE, 
                      0, 0, 0 , 0, 0, 0, 0, 0) );         
  SetPixelFormat ( hDC, ChoosePixelFormat ( hDC, &pfd) , &pfd );
  wglMakeCurrent ( hDC, wglCreateContext(hDC) );
  ShowCursor(FALSE);  
  q = gluNewQuadric();
  gluQuadricTexture (q,GLU_TRUE);
  midiOutOpen(&device, MIDI_MAPPER, 0, 0, 0);
  // launch the music rendering as a seperate thread
  // Then we must set the thread priority to be highest possible so that
  // the music is rendered at a consistent rate...
  SetThreadPriority(_beginthread( &addTune,0,0), THREAD_PRIORITY_TIME_CRITICAL);    
  do {
    // if we dont handle state here, our second cube of cylinders will be fucked up in some
    // intersting but not useful way...
    glEnable(GL_TEXTURE_2D);
    if (i> 200) glDisable(GL_BLEND);
    glEnable (GL_DEPTH_TEST);
    // This cube of cylinders, the solid ones close to us
    // is from -3 to 3
    g =3;
    // capture whats on the screen...
    texture();
    // Tricky to explain but we only clear the screen and swap buffers 
    // on every other loop. This enables us to call drawGrid just once
    // despite drawing two cubes of cylinders...
    if (!(i&1)) {
      SwapBuffers ( hDC );
      glClear ( GL_DEPTH_BUFFER_BIT| GL_COLOR_BUFFER_BIT ); 
      glDisable (GL_TEXTURE_2D);
      glEnable(GL_BLEND);
      glDisable(GL_DEPTH_TEST); 
      // The first cube of cylinders drawn will go from -6 to 6
      g=8;
    }
    // crucial, nothing is drawn without this
    glLoadIdentity(); 
    // viewing angle 120, aspect ratio 1, near, very near and far at 128 units
    gluPerspective(120, 1, 0.01f, 128.0f);
    glBlendFunc(GL_SRC_ALPHA,GL_ONE);
    // These two make us move through the cube of cylinders
    glTranslatef (0,1,1);
    if (i>300) glRotatef (i-300,1,1,1);
    drawGrid(g);

    // my sucky global counter   
    i++;
    if (GetAsyncKeyState(VK_ESCAPE) ) ExitProcess(0);
  } while (1);
}
```

Based around my 1k framework, (see other articles), here is the code for Hive. You need GCC to compile it. Let me know if you spot optimisations please by adding them to the discussion page.

If you use this code to go further or adapt it to something else, please give me some credits, thats all I ask. I do not like placing GPL on my code as I don't like GPL and its philosophy. So some .nfo credits or even better on screen would be great.

Have fun.

-- auld
