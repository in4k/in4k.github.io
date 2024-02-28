---
title: "Javascript"
layout: "wiki-page"
---

# Tools

* [Brotli bootstrapping based on WOFF2 font](https://gist.github.com/lifthrasiir/1c7f9c5a421ad39c1af19a9c4f060743) packer that bypasses need for local server
* [compeko](https://gist.github.com/0b5vr/09ee96ca2efbe5bf9d64dad7220e923b) pack JavaScript into a self-extracting html+deflate
* [NGPNG](https://github.com/subzey/ngpng)
* [fetchcrunch](https://github.com/subzey/fetchcrunch) Like ZPNG and JsExe but using CompressionStream
* [ZPNG](https://xem.github.io/projects/zpng.html) Pure javascript version of JsExe [online](http://codegolf.github.io/zpng/)
* [RegPack](http://siorki.github.io/regPack.html) RegPack, a solid LZss based JavaScript packer by Siorki.
* [JSmin](https://fmarcia.info/jsmin/test.html) the good'ole JS minifier by Douglas Crokford. JSmin only removes whitespace. | [wayback machine](https://web.archive.org/web/20170520173624/http://www.fmarcia.info:80/jsmin/test.html)
* [UglifyJS](http://lisperator.net/uglifyjs/#demo), an excellent JS minifier doing AST transformations.
* [Closure Compiler](https://developers.google.com/closure/compiler/), a Java-based tool that can validate and minify JS code at the AST level
* [jsexe](http://www.pouet.net/prod.php?which=59298), the most advanced PNG bootstraper written for Windows by cb/Adinpz.
* [PNGinator](https://gist.github.com/gasman/2560551), a simpler PNG bootstraper written by Gasman in ruby.
* [SoundToy](http://www.iquilezles.org/apps/soundtoy/), sandbox by IQ to render audio using shaders.
* [wurstcaptures bytebeat music page](http://wurstcaptures.untergrund.net/music/), sandbox by rarefluid to render bytecode audio using javascript audio element.
* [sonant-x](https://github.com/nicolas-van/sonant-x)

# GLSL Minifiers
* [http://www.ctrl-alt-test.fr/glsl-minifier/](http://www.ctrl-alt-test.fr/glsl-minifier/)
* [https://github.com/flowtsohg/glsl-minifier](https://github.com/flowtsohg/glsl-minifier)
* [https://github.com/TimvanScherpenzeel/glsl-minifier](https://github.com/TimvanScherpenzeel/glsl-minifier)

# Other References
* [JavaScript and Demoscene Sizecoding, by p01](https://www.youtube.com/watch?v=waFzG0IVXCw&t=2s&ab_channel=Lovebytedemoparty) 
* [http://nanard.free.fr/demojs/](http://nanard.free.fr/demojs/) | [local copy](/html_articles/demojs stuff.html)
* [http://js1k.com/](http://js1k.com/)
* [https://js1024.fun/](https://js1024.fun/) succcessor to js1k
* [Several releases & post mortems by p01/ribbon](http://www.p01.org/)
* [Making of Cyboman 5](http://www.tpolm.org/herring/?p=410) | [local copy](/html_articles/Making of Cyboman 5 « TPOLM.html)
* [Experimental music from very short C programs](http://www.pouet.net/topic.php?which=8357)
* [JS performance and global eval() (attn: p01 and cb/adinpsz)](http://www.pouet.net/topic.php?which=8770)

# Compression

RegPack and other LZss based packers work well in Javascript however PNG bootstraping gives much better compression ratios by giving access to gz/deflate compression and tools to JavaScript intros. PNG bootstraping is a polyglot file which is both a PNG image representing the code compressed, an HTML page which bootstrap the code compressed by loading it into an image, drawing and reading the pixels onto a canvas to rebuild the code uncopmressed before evaluating it.

# Visuals

The web platform offers many ways to render and animate things, from HTML+CSS including CSS transforms, transitions and filters, to SVG, 2D Canvas and eventually webGL ( a binding of OpenGL ES ). Each with their own strengths and weaknesses. The most common approaches remain 2D Canvas and webGL.

# Audio

There are basically two ways to produce and play sounds in JavaScript. Using the [Audio element](https://html.spec.whatwg.org/multipage/embedded-content.html#the-audio-element) or the [Web Audio API](https://webaudio.github.io/web-audio-api/).

The basic setup for either techniques clock around 170 bytes.

## Audio element

The Audio element is a simple HTML element whose core attributes and methods are `src`, `loop` and `currentTime`, `volume`, `play()` and `stop()`. To use this element in an intro, generate a WAVE PCM file and load it as a `data:` URL. 

    // HEADER of an 8bits mono WAVE PCM at 16384 Hz 
    wavePCM = 'RIFF_oO_WAVEfmt ' + atob('EAAAAAEAAEABAAAAAEAAAABACAA') + 'data';
    for(t = 0; t < duration; t += 1/16384)
      wavePCM += String.fromCharCode(/* insert awesome synth here */);
    A = new Audio('data:audio/wav;base64,' + btoa(wavePCM));
    A.play();
    // You can now access A.currentTime

## Web Audio API

The Web Audio API is a power house of audio nodes used to produce, filter, mix and play sounds in JavaScript. The ScriptProcessor node give programatic access to the next chunk of sound data needed to do realtime sound synthesis.

    A = new AudioContext;
    S = myAudio.createScriptProcessor(L = 4096, T = 0, /*channels=*/1);
    S.connect(A.destination);
    S.onaudioprocess= e => {
      for (i = 0; i < L; i++, T += 1 / A.sampleRate)
        e.outputBuffer.getChannelData(0)[i] = /* insert awesome synth here */
    }

