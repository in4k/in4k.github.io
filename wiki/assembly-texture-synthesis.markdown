---
title: "Assembly Texture Synthesis"
layout: "wiki-page"
---

## General Framework

First of all before beginning with any generating techniques, I'll shortly introduce some concepts that are used throughout the whole article to minimize the size overhead for the generators. However you can apply most of the stuff to arbitrary texture formats.

I'll assume textures to be 256^2 in size and 32bits per pixel if not stated other. Under this assumption it is pretty easy to handle the access, since pixels are 32bit aligned and using 256^2 as texture size gives the bonus being able to use only one counter for looping over all pixels, beeing able to directly access x/y via low/high byte register access.

I'll also assume that esi is bound to a source image and edi is bound to the destination image memory. Any other assumptions will be states explicitly.

A simple fill texture to a constant color would for example look like this:

```
; assume edi = destination texture memory
; assume eax = fill color

xor ecx, ecx
dec cx
fillloop:
  ; you can use cl and ch here to access x/y counter values
  stosd
loop fillloop ; warning last pixel wont be processed.
              ; you need to use dec ecx, jns instead
```

## Checkerboard texture

![checkerboard6qi.png](http://in4k.untergrund.net/images/checkerboard6qi.png)

```
; assume ecx = gridsize [0..7]
xor ebx, ebx    ; using ebx this time
dec bx          ; as counter
checkerloop:
  mov eax, ebx  ; move x/y to eax
  shr eax, cl   ; scale by ecx
                ; note bits shiftet from ah to al
                ; wont affect checkerboard, because
                ; we use only the lowest bits of ah/al
  xor ah, al    ; invert ah using al as mask
                ; if the lowest bits were were equal, it will
                ; become 0 else if will become 1
  sahf          ; load ah to the flags (now carry holds lowest bit)
  sbb eax, eax  ; substract eax, from eax (resulting into 0)
                ; and also substract 1 more if carry was set
                ; so we get black/white
  stosd         ; store the pixel
  dec ebx
jns checkerloop
```

## Noise texture

![noise5nn.jpg](http://in4k.untergrund.net/images/noise5nn.jpg)

```
; assume ebx = seed (must not be 0)
xor ecx, ecx
dec cx
@fillloop:
  add eax, ebx
  rol eax, 3
  add ebx, eax
  stosd
loop @fillloop
```

Note: this noise does not have good "random" properties, but should work well for most purposes in a 4k

## Saturated color blending

![bluenoise2ue.png](http://in4k.untergrund.net/images/bluenoise2ue.png) ![greenchecker0zh.png](http://in4k.untergrund.net/images/greenchecker0zh.png)

```
; assume color in eax
xor ecx, ecx
dec cx
shr ecx,1 ; processing 2 pixel per iteration
push eax
push eax
blendloop:
  movq mm0, QWORD [esi+ecx*8]
  paddusb mm0, QWORD [esp]        ; using psubusb would substract the color
  movq QWORD ptr[edi+ecx*8], mm0
  dec ecx
jns blendloop
pop eax
pop eax
```

You could also modify the code to blend (additive or subtractive) two images:

```
; assume second source image in edx
xor ecx, ecx
dec cx
shr ecx,1 ; processing 2 pixel per iteration
blendloop:
  movq mm0, QWORD [esi+ecx*8]
  paddusb mm0, QWORD [edx+ecx*8]  ; using psubusb would substract the second image
  movq QWORD ptr[edi+ecx*8], mm0
  dec ecx
jns blendloop
```
