---
title: "Auld's OGL Framework"
layout: "wiki-page"
---

## Minimal (?) Ogl and Win32 Set-up Code

**You won't see this on NeHe**

Disregarding for the moment which compiler and compressor you use, this code is close to minimal for what it achieves.

Let me know if you can improve on this.

Tricks used include

*   do {} while loop, not while{}
*   PVOID hDC is re-used as hWnd
*   hDC is assigned a value at declaration time
*   ^ used not ==
*   Peekmessage not usual windows message handling
*   Overriding WinMainCRTStartup instead or using WinMain
*   Chained assignment to force loading of constant once
*   Using Edit windows class - smallest string size for any class
*   GetSystemMetrics used for resolution of screen
*   No checking of error codes .. its a demo, I'm going to test it anyway
*   ChosePixelFormat call is embedded in SetPixelFormat
*   Assumes unused fields of pfd are set to zero (DANGEROUS BUT WORKS)
*   Don't set sizeof field in pfd (DANGEROUS BUT WORKS)
*   Don't set the version number of pfd...implicitly its 0 (DBW)

```
//
// (c) Auld 2005
// Re-use granted as long as long as you give credit in
// either your demo or accompanying files please.
//
// The following code sets up an Opengl window under win32
// It is double buffered, hides the mouse, has 32 bits of depth/Z.
// The main loop includes a clear for depth and color bits and 
// a swapbuffers call for drawing.
// It exits when escape is pressed...
// Tested under XP.
//
#include <windows.h>
#include <GL/gl.h>
```

```
int WINAPI WinMainCRTStartup()
{
   MSG m;                 
   PIXELFORMATDESCRIPTOR pfd;  
```

```
   static PVOID hDC = CreateWindow("EDIT", 0, 
                        WS_POPUP|WS_VISIBLE, 
                        0, 0,
                        GetSystemMetrics(SM_CXSCREEN), 
                        GetSystemMetrics(SM_CYSCREEN),
                        0, 0, 0, 0);
   hDC = GetDC(hDC);    
   ShowCursor(FALSE);           
   pfd.cColorBits = pfd.cDepthBits = 32; 
   pfd.dwFlags    = PFD_SUPPORT_OPENGL | PFD_DOUBLEBUFFER;			
   SetPixelFormat ( hDC, ChoosePixelFormat ( hDC, &pfd) , &pfd );
   wglMakeCurrent ( hDC, wglCreateContext(hDC) );
```

```
   do {
       glClear  (GL_DEPTH_BUFFER_BIT|GL_COLOR_BUFFER_BIT);
       // insert Breakpoint winning 4kdemo here
       SwapBuffers(hDC);
       PeekMessage(&m, 0, 0, 0, PM_REMOVE);     
   } while ( m.wParam ^ VK_ESCAPE);
}
```

Byte crunching is fun :-)

* * *

There are indeed few tunings available to this frame:

```
   static PVOID hDC = CreateWindow("EDIT", 0, 
                        WS_POPUP|WS_VISIBLE, 
                        0, 0,
                        GetSystemMetrics(SM_CXSCREEN), 
                        GetSystemMetrics(SM_CYSCREEN),
                        0, 0, 0, 0);
```

could be optimized to:

```
   static PVOID hDC = CreateWindow("EDIT", 0, 
                        WS_POPUP|WS_VISIBLE|WS_MAXIMIZE, 
                        0, 0,
                        0, 
                        0,
                        0, 0, 0, 0);
```

Doing so you have a longer sequence of zeroes that compresses better, but also saved a whole import and two calls.

The next point I would consider would be replacing your mainloop with something like

```
   while(1)
   {
     ...
     if (GetAsyncKeyState(VK_ESCAPE)) ExitProcess(0);
   }
```

resulting into much fewer immediates to be stored. But for the cost of an additional import.

Lets take a more detailed look at what your call actually costs: PeekMessage(&m, 0, 0, 0, PM_REMOVE);

```
   mainloop:
   ...
   00401544 6A 01             push        1    
   00401546 6A 00             push        0    
   00401548 6A 00             push        0    
   0040154A 6A 00             push        0    
   0040154C 8D 45 E4          lea         eax,[ebp-1Ch] 
   0040154F 50                push        eax  
   00401550 FF 15 14 20 40 00 call        dword ptr [PeekMessageA] 
   00401556 8B 45 EC          mov         eax,dword ptr [ebp-14h] 
   00401559 83 F0 1B          xor         eax,1Bh 
   0040155C 75 E6             jne         mainloop 
```

This is 26 Bytes mainly used for the immediates and the MSG accesses.

The code i suggested above in comparison to this might get translated to

```
   mainloop:
   ...
   0040153E 6A 1B             push        1Bh  
   00401540 FF 15 10 20 40 00 call        dword ptr [GetAsyncKeyState] 
   00401546 66 85 C0          test        ax,ax 
   00401549 74 F3             je          mainloop: 
   0040154B 6A 00             push        0    
   0040154D FF 15 00 20 40 00 call        dword ptr [ExitProcess] 
```

As you can see it takes 21 bytes total

If you are using import by hash the additional import only will cost you 4 Bytes. Note that it's also safe to skip the 0 from ExitProcess, hence the stack must be valid at that point (or you really messed it up!) with the top element pointing to a kernel32 function or some local variable if you had them.

So at all using the following code will be fine, resulting into 19 Byte:

```
   // even more optimized mainloop
   while(1)
   {
     ...
     if (GetAsyncKeyState(VK_ESCAPE))
       ((void (WINAPI *)(void))ExitProcess)(); 
   }
```

Going even further you could tweak up your calls by using an indexed function list, which in the best case will cost you only 3 Byte instead of the 6 byte calls above.

Thus you can get down the later mainloop version to 13Byte (+4 for import) and the original version to 23Byte

Another point would be that you could omit the stackframe closing part and the return at the end of the function. However this would need some more compiler specific workaround to do, but could save you the needed bytes.

About your pixelformat setup: It really looks pretty dangerous since you are storing the descriptor locally (meaning its memory comes from the stack, with a good chance that it might be undefined. I would at least consider setting the nSize field as recommended by the MSDN, hence it suggests that there are different versions of the struct which might crash your code easily when mixed. If you have any experience throughout different configurations.

In my current framework im totally omitting the descriptor using wglChoosePixelFormatARB however that will introduce you lots of extension loading overhead and also a little descriptor, which under normal conditions is not worth at all. But if you have a really good framework (I doubt you could get this effeticly in C), you could get FSAA this way for almost no extra costs.

-- Icehawk

* * *

Icehawk thanks for the great contribution. I've plugged your suggestions into my environment under GCC (did I forget to mention that?). The WS_MAXIMISE works great and saves 16 bytes under compression, I presume due to the long sequence of zeroes.

I couldn't get close to your results with your loop and key check suggestions. However, your suggestion led me to do the following:

```
do {
 ...
} while (!GetAsyncKeyState(VK_ESCAPE));
```

in the main loop. In all therefore, this has shaved **34** bytes after compression. 16 for the maximise trick and 18 for getting rid of PeekMessage. I don't think savings were as great as you expected because you forgot that the PeekMessage assembly will compress very well with sequences of 6A 00 in it. Just a guess...like I said, I draw the line at assembly I'm afraid.

-- Auld

* * *

Take a look at [Import by hash](import-by-hash) for even further optimisation tricks you might apply. Even the best exe packer should not be able to transform your imports to import by hash and give you an ebp indexable functionpointer list which lets you crunsh your API calls to 3 Byte each.

-- Icehawk

* * *

[Crinkler](http://www.crinkler.net "http://www.crinkler.net") actually does exactly that. It automatically transforms your imports into import by hash with 4 byte hash codes. To index into the function pointer list, you can simply use

```
   mov ebp, _ImportList+24+128
```

and then to call a function

```
   call [ebp + (__imp__CreateWindowExA@48 - (_ImportList+24+128))]
```

where the +24+128 comes from the fact that the first function address is at _ImportList+24 (for header-technical reasons) and you can refer up to 128 bytes backwards using a byte offset, so this (almost) maximises the number of functions you can call in 3 bytes.

However, I doubt you will get any benefit from this. It turns out that the normal 6-byte API calls compress quite well. I did some measurements on a couple of OpenGL intros, and from what I saw, these calls typically compress to between 11 and 16 bits on average. The 3-byte calls will be compressed as well of course, but probably not noticably better than the normal ones, since they contain basically the same information, just encoded in a different way. Add to that the nuisance of having one register less and being forced to write your whole intro in assembler.

A bonus feature of Crinkler with regards to imports is its **ordinal range import** which can be used with static DLLs like opengl32\. The idea is to just store the hash code of the first function and then import a range of ordinals starting with that. It works if the function ordinals are fixed relative to each other, which is the case with opengl32 for all currently available Windows versions (including Vista). It reduces the import overhead of OpenGL functions from 4 bytes per function to typically 12 bytes in total. I might add some more details on this on the [Import by hash](/index.php?title=Import_by_hash "Import by hash") page some day...

This technique spreads the function pointers a bit more in memory, as it also imports many unused functions. This means that the ebp offset trick will not be able to use byte offsets for all of them, which will really screw up compression. The normal API calls get around 1 bit bigger apiece. Still a big gain.

Hmm, this has moved a bit away from the OGL Framework. Maybe we should have a page for general size optimization techniques...

-- Blueberry

* * *

Additional byte killing:

We have this:

```
   mainloop:
   ...
   0040153E 6A 1B             push        1Bh  
   00401540 FF 15 10 20 40 00 call        dword ptr [GetAsyncKeyState] 
   00401546 66 85 C0          test        ax,ax 
   00401549 74 F3             je          mainloop: 
   0040154B 6A 00             push        0    
   0040154D FF 15 00 20 40 00 call        dword ptr [ExitProcess] 
```

You could replace this (5 bytes):

```
   test ax,ax
   je mainloop
```

With this (4 bytes):

```
   test eax,eax
   je mainloop
```

Or just do the following (3 bytes):

```
   dec eax
   js main
```

Now we have 3 (there are more) possibilites to do the "same thing", we can chose the one that compresses best (in our given context). Entropy can really make you crazy... I once just changed "EDIT" to "edit" and it compressed better.

Additionally I push the parameter for the ExitProcess at the beginning of the programm (i give CreateWindowExA one 0 parameter more, I don't know if you could reach that in high level by changing the declarations).

Talking about the use of ebp-x for function pointers, imho it's not bad to do that, but I prefer to use ebp-x as a data storage (in high level that would be something like "global locals", I use that for e.g. smaller FPU calls). And who needs stackframes in a 4k intro? ;)

-- las

* * *

After the comments here, the code is now updated, for completeness. This is the latest known minimal version (in C).

Tricks Added:

*   "edit" in lowercase which might compress better depending on what you add to this code (las)
*   Peekmessage and msg declaration removed and replaced with key state check (Icehawk)
*   WS_MAXIMIZE added rather than two calls to GetSystemMetrics (Icehawk)
*   The call to getDC is integrated with createwindow so no temporary lvalue is needed (auld)
*   Grouped pfd and hDC variable access better (saves about 4 bytes ... your mileage may vary) (auld)

```
//
// Re-use granted as long as long as you give credit in
// either your demo or accompanying files please...(auld)
//
// Thanks go to Icehawk for WS_MAXIMIZE optimisation and the idea
// how to get rid of Peekmessage
// The following code sets up an Opengl window under win32
// It is double buffered, hides the mouse, has 32 bits of depth/Z.
// The main loop includes a clear for depth and color bits and 
// a swapbuffers call for drawing.
// It exits when escape is pressed...
// Tested under XP.
//
#include <windows.h>
#include <GL/gl.h>
```

```
int WINAPI WinMainCRTStartup()
{              
   PIXELFORMATDESCRIPTOR pfd;  
   pfd.cColorBits = pfd.cDepthBits = 32; 
   pfd.dwFlags    = PFD_SUPPORT_OPENGL | PFD_DOUBLEBUFFER;	
```

```
   PVOID hDC = GetDC ( CreateWindow("edit", 0, 
                        WS_POPUP|WS_VISIBLE|WS_MAXIMIZE, 
                        0, 0, 0 , 0, 0, 0, 0, 0) );         
   SetPixelFormat ( hDC, ChoosePixelFormat ( hDC, &pfd) , &pfd );
   wglMakeCurrent ( hDC, wglCreateContext(hDC) );
```

```
   ShowCursor(FALSE);  
```

```
   do {
       glClear ( GL_DEPTH_BUFFER_BIT | GL_COLOR_BUFFER_BIT );
       // insert Breakpoint winning 4kdemo here
       SwapBuffers ( hDC );   
   } while ( !GetAsyncKeyState(VK_ESCAPE) );
}
```

-- auld

* * *

Another thing that just came to my mind, talking about "CreateWindow"... Seems it's not inside user32.dll (nor any other dll) afaik, the Delphi-Header calls CreateWindowExA (with first param=0) instead of using an DLL function for CreateWindow... You should use CreateWindowExA instead (and I recommend to use it with first param=WS_EX_TOPMOST else it can happen that your window is not 100% "on top")

-- las

* * *

Yes you are right las: CreateWindow just calls the corresponding Ex version. And of course it's a good idea to use WS_EX_TOPMOST as extended argument, otherwise you will have windows before your graphics window when for example switching to another window via taskmanager etc. (However skip that flag if you want to run your debugger over the window...)

auld, you should think about changing your WS_MAXIMISE to WS_MAXIMI**Z**E... that might do better since I doubt that everyone has such strange hacked headers like you seem to have. 

-- Icehawk

* * *

Done .. American spelling , Not British, should have cut and paste from real code.

Although I get the comments about ExA and WS_EX_TOPMOST, so far its not significant. Running the code from say DevC++ will result in the effect you describe. Running the code from a shell or a windows panel, it covers the screen fine. Sure, skype and Azureus and so forth can pop up inconvenient windows but such programs are unlikely to be running on a demo machine and as it adds a small number of bytes I'll wait until its a problem.

No, I stick by my choice of CreateWindow for now ...its smaller and in practice gives very few problems. As 4k is all about cheating...I think its the right choice, but reader beware.

-- auld

* * *

That's really smaller? Because it - as said before and confirmed by Icehawk - USES CreateWindowExA (in your code). Could you upload an executable somewhere? I would like to take a look at the disassembly with OllyDBG. 

-- las

* * *

Yes really, really. The version posted is 482 bytes using my compiler and dropper. If I add the code

```
PVOID hDC = GetDC ( CreateWindowExA(WS_EX_TOPMOST,"edit", 0, 
                       WS_POPUP|WS_VISIBLE|WS_MAXIMIZE, 
                       0, 0, 0 , 0, 0, 0, 0, 0) );
```

instead then the result is 485 bytes...

I'll upload some executable if you are curious but that will have to wait until another day.

-- auld

* * *

Use instead of WS_EX_TOPMOST just 0, the result should have exactly the same size as with CreateWindow. Take a look at your windows header and check the defines for CreateWindow.

-- las

* * *

Yes, winuser.h defines CreateWindow (..) to be CreateWindowExA (0,...) and yes that is the same size as the code posted when tested (because its the same code). But you are losing me...if we don't use WS_EX_TOPMOST and the resulting code is the same size...whats the advantage?

-- auld

* * *

Most people use VC++, not GCC like me. In order for the code to work under VC++, make these changes:

change PVOID hDC= to HDC hDC=

Also, and I should do this anyway,

change int WINAPI WinMainCRTStartup() to void WINAPI...

-- auld

* * *

**Visual C++ 2005 Express port:**

Now you can download aulds framework ported to Visual C++ 2005 Express:

* [http://www.rbraz.com/source/1K_FrameWork_VC2005.zip](http://www.rbraz.com/source/1K_FrameWork_VC2005.zip "http://www.rbraz.com/source/1K FrameWork VC2005.zip")
* [in4k github repo](https://github.com/in4k/1K_FrameWork_VC2005)

The best about this is that it compress to 427 bytes only :) To compress, just use the same aulds settings for dropper and apack.

And don't forget to give me some credits if you use it, please.

-- rbraz
