---
title: "Saida's 1024b Midi Sound System (masm)"
layout: "wiki-page"
---

The only way of making decent music for a 1k in windows is by using midi.

Midi offers a small way of playing notes (one init, and then just pump notes)  

This is how its done (in masm).

Please check [full article](http://saida.lava.nu/wordpress/?p=53 "http://saida.lava.nu/wordpress/?p=53") for a fully comented code.  
[compiled exe of this source](http://www.hd.chalmers.se/~Saida/titan/mss.exe "http://www.hd.chalmers.se/~Saida/titan/mss.exe")

```
       xor eax, eax
       mov edi, offset bss_hMidi       ; use mov instead of lea. (smaller) 

       push eax                        ; PUSH 0 
       push eax                        ; PUSH 0 
       push eax                        ; PUSH 0 
       dec eax 
       push eax                        ; PUSH 0xFFFFFFFF (MIDI_MAPPER)  	    	 
       push edi                        ; ( EDI is offset to hMidi )        	         
       CALL midiOutOpen                ; call midiOutOpen 
```

after this point, we assume midiOutOpen returned ok (eax = 0)

```
   @@bss_RESET_SONG:                
       mov esi, edi                    ; set esi to point at fake data section 
       LODSD                           ; Same as "add esi,4". (destoys eax)  
                                       ; esi is now = offset "tone data"         
   @@bss_LOAD_NEXT:                 
       LODSB                           ; get BYTE from ES:ESI (the array) 
       xchg ah, al	 
       xor ebx,ebx						; set to zero before. 
       add bl, ah						; if ah is zero, that means that we  
                                       ; want to loop the song. 
       jz @@bss_RESET_SONG             ; are we done? -> loop and play again. 

       shr bl, 5                        
       and ebx, 00000011b                
       mov bx, word ptr [edi-16+(2*ebx)] 

       mov ecx, eax                    ; save toneData to ECX  
       and ax, 0001111100000000b 
       or eax, 00000000011111110010000010010001b         
       push eax                        ; push tone to play.
       push dword ptr [edi]            ; push handle to hMidi

       xchg eax, ecx                   ; Restore toneData to eax.
       sahf                            ; pushes AH to flags. (SF = Bit7 in ah)
       mov eax, [edi-8]                ; use instrument 0                             
       js @@bss_INSTRUMENT_DONE        ; Jump if bit7 is 1\.                           
       mov eax, [edi-4]		; else, use instrument 1                        
```

```   @@bss_INSTRUMENT_DONE:					
       push eax                        ; push the instrument 
       push dword ptr [edi]            ; push handle to hMidi  
       call midiOutShortMsg		
       call midiOutShortMsg		

       push ebx                        ; ebx = sleep time 
       call Sleep                      ; wait for tone to finish 

       jmp @@bss_LOAD_NEXT             ; Repeat the procedure    	 
```

code will never be executed bellow this point

```
   dw 240			; lengh index 0\.  
   dw 1			; lengh index 1 
   dw 0			; lengh index 2 
   dw 0			; lengh index 3 

   dd 12594625		; instrument index 0 
   dd 12594625		; instrument index 1 
```

```
   bss_hMidi dd ?	; handle to midi out.
```

song data starts here

```
   db 10000101b 
   db 10101001b 
   db 10001100b 
   db 10000000b 
   db 10101001b 
   db 10001100b 
   db 10000101b 
   db 10101001b 
   db 10001100b 
   db 10000000b 
   db 10101001b 
   db 10001100b 
   db 10000101b 
   db 10101001b 
   db 10001100b 
   db 10000000b 
   db 10101001b 
   db 10001100b 
   db 10000101b 
   db 10000000b	 
   db 10000010b 
   db 10000100b 
   db 00000000b 

end ProgramEntryPoint
```
