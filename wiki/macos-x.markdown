---
title: "MacOS X"
layout: "wiki-page"
---

WORK IN PROGRESS!!!

* * *

Making size limited 1k/4k intros for OS X can be said it is like hybrid from Windows and unix systems. On the other hand tricks like import-by-hash dynamic loading is rather starightforward to do like in Windows, on the other hand the unix backend allows some nifty things like shell dropping.

In high-level there are certain pros and cons working with OS X in size-limited world.

Pros:

* OS X frameworks make it possible to load huge amounts of functionality with a single dependency (Cocoa)
* Loading libraries and resolving symbols from libraries using hash function is really straightforward and easy
* Static overhead is rather minimal. Decompression + Dynloading amounts to ~250 bytes.

Cons:

* Minimum size of OS X MACH-O binary is 4096. (Since 10.10.5) This is no issue for 4k intros, but basically mandates shell dropping on 1k
* OpenGL is either legacy or core, you can't mix. GLSL compiler is strict about syntax
* Apple is killing of 32-bit libraries and frameworks one by one. Moving to 64-bit loses some benefits there are in OS X
* Apple likes toss around functions between libraries in every major version making hash-imported libraries break.

* * *

Fundamentals (32 bit executables)

The OS X native executable format, [Mach-O](http://developer.apple.com/documentation/DeveloperTools/Conceptual/MachORuntime/index.html), contains bare minimum header and load commands followed by content for the segments defined. A practical executable needs segment, main, dylinker and at least one dylib command.

Due to recent jailbreak exploits the kernel, Apple has made the kernel side format validation really strict. Mach-O header and load commands needs to be of correct size and offsets need to point into the correct command issued. Only commands loaded by dyld can be still fudged around. Practically this means that only library list and sdk-header can be modified, which does not optimize much but allows making header more regular for compression purposes.

Example of minimal executable with 180-bytes of header

```Assembly
	org 0
	bits 32

%include "symbols.asm"

FileStart:
	[section .text]
MACHHeader:
	dd 0xfeedface				; magic
	dd SOLVE(CPU_TYPE_X86)			; cpu type
	dd 0					; cpu subtype (wrong, but works)
	dd SOLVE(MH_EXECUTE)			; filetype
	dd 4					; number of commands
	dd MACHCommandsEnd-MACHCommandsStart	; size of commands
	dd 0					; flags
MACHHeaderEnd:

MACHCommandsStart:

Command1Start:
	dd SOLVE(LC_MAIN)			; this is main
	dd Command1End-Command1Start		; size
	dd CodeStart-MACHHeader			; eip
	dd 0
	dd 0					; stack
	dd 0
Command1End:

Command2Start:
	dd SOLVE(LC_SEGMENT)			; this is segment
	dd Command2End-Command2Start		; size
SegNameStart:
	times 16-$+SegNameStart db 0		; section name

	dd 0					; vmaddr
	dd 0x100000				; vmsize
	dd 0					; fileoff
	dd FileEnd				; filesize
	dd 0					; maxprot
	dd SOLVE(VM_PROT_READ)|SOLVE(VM_PROT_WRITE)|SOLVE(VM_PROT_EXECUTE) ; initprot
	dd 0					; nsects
	dd 0
Command2End:

Command3Start:
	dd SOLVE(LC_LOAD_DYLINKER)		; dyld
	dd Command3End-Command3Start		; size
	dd DyldName-Command3Start		; offset
DyldName:
	db '/usr/lib/dyld',0			; 14 bytes
;	db 0,0					; breaking the spec here, no-one minds.
Command3End:

Command4Start:
	dd SOLVE(LC_LOAD_DYLIB)
	dd Command4End-Command4Start		; size
	dd DyName-Command4Start			; offset
	dd 0					; timestamp
	dd 0					; min ver
	dd 0 					; max ver
DyName:
	db 'Cocoa.framework/Cocoa',0
Command4End:

MACHCommandsEnd:

CodeStart:
	ret
CodeEnd:

	times 4096-(CodeEnd-MACHHeader) db 0
FileEnd:
```

Please see the definition of SOLVE() macro in the tools/examples section

Obviously the next question is how to get hand to the libraries loaded. There are two known ways to do it, assuming the hash-loading is the desired way to resolved:

1. Use combination of __dyld_get_image_header, __dyld_get_image_name, __dyld_get_image_vmaddr_slide to get the information of the each image loaded. These functions take integer as a parameter from 0 onwards (0 is current image)
2. Use __dyld_get_all_image_infos to get list of the all images loaded. This is easier, but does not allow getting the vmslide, which means that libraries that need to be slided (i.e. non-optimized libraries) can't be loaded or the slide has to be calculated by some other means. For most common libraries this is not really an issue.

After the address of the library is known (as well as the name) resolving the symbols is rather trivial. The library address points to the mach-o header of the library and seeking the dysymtab where function addresses and names can be found.

The only remaining problem is that where the __dyld-functions can be found. Fortunately for us dyld is loaded into a static address 0x8fe00000 + ASLR fudge, which is fortunately available at the start of the execution (and only at the start of the execution, it will be gone after that)

* * *

Compression

There has been many variants of PPM-like compressions for the size coded entries, which is still algorithm used by many intro tools (Crinkler, elfling). For OS X we can use [onekpaq](http://www.pouet.net/prod.php?which=66926) that has Crinkler-like performance. (Tool is open source and can be used for other platforms as well)

Due to verbose headers and 4k minimum size limitation it is almost always mandatory to use shell-dropping, even if the actual executable is already compressed by some better compressor. Recede/TDA was the first Mac 4k to feature double compression. onekpaq compressed the content and shell dropper was used to get the header minimized.

Most of the same tricks that are valid for other unix are also valid for OS X. However OS X does not have xz, the favorite compressor of BSD and Linux sizecoders, we have to get by using gzip/bzip2 if we want to do shell dropping.

By interleaving the gzip header (6 free bytes) with the shell dropper we can have the shortest possible shelldropper, 34 bytes:

First line:
```
cp $0 /tmp/z;(sed 1d $0|zcat
```

and in gzip header (offset +4, flags needs to be 0x10)
```
)>$_;$_
)
```

Firehawk originally found out the "sed 1d"-trick, now it is widely used. This header could be minimized more if someday Apple starts to install tac or changes default shell zsh, both of which are very unlikely to happen.

* * *

OpenGL

TODO

Legacy profile minimum shader-intro (743 bytes with laturi):

```C
#include <OpenGL/OpenGL.h>
#include <ApplicationServices/ApplicationServices.h>
#include <Carbon/Carbon.h>
#include <OpenGL/glu.h>
#include <OpenGL/CGLTypes.h>
#include <OpenGL/CGLContext.h>

extern void CGSGetKeys(KeyMap k);

static const GLchar *fragment_shader=
	"void main()"
	"{"
		"float time=gl_TexCoord[0].x;"
		"vec2 pos=gl_FragCoord.xy;"
		"vec4 c=vec4(.2);"
		"if (length(mod(pos*.001,.25)-.125)<mod(time*.01,.25)) c+=.5;"
		"gl_FragColor=c;"
	"}";

void main(void)
{
	CGDisplayHideCursor(0);							// parameter not used by Quartz
	static const CGLPixelFormatAttribute attribs[]={0};			// anything goes.
	CGLPixelFormatObj formats;
	GLint num_pix;
	CGLChoosePixelFormat(attribs,&formats,&num_pix);
	CGLContextObj ctx;
	CGLCreateContext(formats,0,&ctx);					// first hit is good enough for us.
	CGLSetCurrentContext(ctx);
	CGLSetFullScreenOnDisplay(ctx,CGDisplayIDToOpenGLDisplayMask(0));
	
	GLuint shader=glCreateShader(GL_FRAGMENT_SHADER);			// now shader
	glShaderSource(shader,1,&fragment_shader,0);
	glCompileShader(shader);

	GLuint program=glCreateProgram();
	glAttachShader(program,shader);
	glLinkProgram(program);
	glUseProgram(program);

	float time=0;
	for (;;)
	{
		glTexCoord1f(time);						// set shader timing, should come from music
		time+=.16;

		glRecti(-1,-1,1,1);						// actual drawing
		glSwapAPPLE();
	
		KeyMap keys;
		CGSGetKeys(keys);
		if (((unsigned char*)keys)[6]&0x20) break;
	}
}
```

Core profile minimum shader-intro (771 bytes with laturi):

```C
#include <OpenGL/OpenGL.h>
#include <OpenGL/GL3.h>
#include <ApplicationServices/ApplicationServices.h>
#include <Carbon/Carbon.h>
#include <OpenGL/glu.h>
#include <OpenGL/CGLTypes.h>
#include <OpenGL/CGLContext.h>

extern void CGSGetKeys(KeyMap k);

const GLchar* vertex_shader =
	"#version 330\n"
	"out gl_PerVertex{vec4 gl_Position;};"
	"void main(){"
		"gl_Position=vec4(gl_VertexID%3>>1,gl_VertexID%3&1,gl_VertexID/3*.001,.5)-.25;"
	"}";

static const GLchar *fragment_shader=
	"#version 330\n"
	"out vec4 g;"
	"void main()"
	"{"
		"float time=gl_FragCoord.z*100;"
		"vec2 pos=gl_FragCoord.xy;"
		"vec4 c=vec4(.2);"
		"if (length(mod(pos*.001,.25)-.125)<mod(time*.01,.25)) c+=.5;"
		"g=c;"
	"}";

void main(void)
{
	CGDisplayHideCursor(0);							// parameter not used by Quartz
	static const CGLPixelFormatAttribute attribs[]={kCGLPFAOpenGLProfile,kCGLOGLPVersion_3_2_Core,0};
	CGLPixelFormatObj formats;
	GLint num_pix;
	CGLChoosePixelFormat(attribs,&formats,&num_pix);
	CGLContextObj ctx;
	CGLCreateContext(formats,0,&ctx);					// first hit is good enough for us.
	CGLSetCurrentContext(ctx);
	CGLSetFullScreenOnDisplay(ctx,CGDisplayIDToOpenGLDisplayMask(0));

	GLuint fprogram=glCreateShaderProgramv(GL_FRAGMENT_SHADER,1,&fragment_shader);
	GLuint vprogram=glCreateShaderProgramv(GL_VERTEX_SHADER,1,&vertex_shader);

	GLuint pipeline;
	glGenProgramPipelines(1,&pipeline);
	glUseProgramStages(pipeline,GL_FRAGMENT_SHADER_BIT,fprogram);
	glUseProgramStages(pipeline,GL_VERTEX_SHADER_BIT,vprogram);
	glBindProgramPipeline(pipeline);

	GLuint vao;
	glGenVertexArrays(1,&vao);
	glBindVertexArray(vao);

	int frame=0;
	for (;;)
	{
		frame++;

		glDrawArrays(GL_TRIANGLES,frame*3,3);
		glSwapAPPLE();
	
		KeyMap keys;
		CGSGetKeys(keys);
		if (((unsigned char*)keys)[6]&0x20) break;
	}
}
```


* * *

Audio

* * *

Tools

* * *

Examples

* * *