---
title: "Auld's Tongue in Cheek List for 4k Demos"
layout: "wiki-page"
---

h2> Aulds TIC List for 4k Demos

I started reading and working on 4k demos over a year ago now. I still don't have a 4k demo to publish. Why? Well, here is a tongue in cheek (TIC) list of all the stuff you need to publish a winning Assembly 4k demo or a Pouet 0.8 ranked 4k demo. Of course I'm not saying if you have all this you will win. I *am* saying that the entry level bar is high now.

Think of it this way... for every time someone says on Pouet "Great demo but it lacks...", I write it down.

* * *

**The List**

*   It has to work first time.

You won't get a second chance with most people. This generally means producing a compressed one for the demo machine and a safe version.

*   It has work on machine X.

You'll be marked down if my graphics card can't run your demo or my PC doesn't support your ordinal importing. This is a trade off of course in the squeeze to reach 4k.

*   It has to work soon.

You'll be marked down if you spend 30 minutes pre-calculating. You'll ve marked down if I you don't display something interesting if your demo takes a minute to load.

*   It has to exit smoothly.

Support the escape key. Everywhere. Respond instantly to it. Restore PC settings smoothly when you exit.

*   3D graphics

OGL or D3D is almost required. Hardware acceleration is the key. Perhaps a cool raytracing engine, especially accelerated through OpenGL etc.

*   Textures

You need to texture stuff so you need a texture generation system of some kind.

*   Transitions

Fades, wipes etc. You need a 2d graphics engine in other words.

*   Animation System

No, not one effect done many different times but actually "scenes" connected in some way require an animation system.

*   One or more "how did they do that" Effects

If you do 3d that Jo Schmo could do, and don't get some sort of "wow" factor going, you'll be marked down. Something somewhere has to be called an effect. Hint: you won't find an effect on NeHe. You need a unique.

*   Story

One effect isn't enough. A demo design flowing from one effect to another is good. Better is a story. People are beginning to expect even 4k demos to make a point, say something, keep your interest. Call it a theme if you like.

*   Sound effects

A scream, a boing, a whee, a thump.

*   A tune

It has to suit the demo and sound good. Pure midi might be marked down. Real time synth is best. You'll need something to generate sounds and something to play them at the right time. Synchronising with the graphics is a requirement now.

*   A credits section

You'll need a text engine. You'll need friends. Though people will understand if you can't squeeze friends into your 4k.

*   A Loading screen

Don't leave the screen blank for 2 minutes. You've already lost if you do. The bar progressing from left to right is becoming old...

*   Be memorable/different

People are becoming bored with rotating, spikey balls. You need something new.

*   ASCII art logo

You'll need something for your info file on Pouet. The cooler the better.

* * *

For fun here is an extra credit TIC list.

*   Layers

One set of graphics on another. Its done all the time in 64k and its coming to 4k. You'll need a more advanced animation engine.

*   Rendering to Texture

This enables you to capture a scene and do imaging on it or to create cool textures by rendering OGL to a texture.

*   Old Skool Effect

One or more old skool effects to show your heritage buried deep in your demo or maybe on your loading screen. Of course you don't have to be old skool but it shows some additional knowledge and respect. Its ballsy too. It says, I could do old skool if I had to but I'm better than that.

*   Shaders

Lots of people still can't do them so throw one in, even if it does use more bytes than doing the damn thing in C.

*   Alpha Blending

Throw this in, use an unusual mode and people will never work out what you are doing. Switching on the accumulation buffer is recommended for all demos at some point.

*   Physics

Bounce something. Wobble something. Drape some cloth. Navier Stokes it. Worth points because Jo Schmo can't do it.

*   Lots of geometry

Go on, its three lines of code to put as many copies of the boring geometry on the screen as possible. People are always impressed with more (how do they get that into 4k?)

*   Advanced Lighting

Bump mapping, radiosity, HDR, ambient occlusion, refraction, reflection will all get you points. Throw in shadows and wow, its a winner.

*   Mention someone 4k-famous and 4k-fabulous in your credits

Even if you never spoke to them. ;-) told you this was TIC.

* * *

Lastly here is a TIC List of **Don'ts** for 4k demos.

*   DON'T Use x&y, x^y, x|y textures...unless you can disguise them.
*   DON'T Use fire/zoom mandel/plasma/tunnel unless its for throw away show off, not the main point of your demo.
*   DON'T Copy anything from anyone - it will be spotted by someone.
*   DON'T Make a silent demo
*   DON'T Have only one effect
*   DON'T Use non-antialiased points in particle demos
*   DON'T Be too long
*   DON'T Use DLLs that aren't standard (no matter what the compo rules are, people will mark you down on Pouet)
*   DON'T Write a list at the end of the demo of numbers/size of textures etc. to impress with how much uncompression you do
*   DON'T Clip your demo music
*   DON'T end abruptly...fade
*   DON'T use Dutch colours or any other programmers colour scheme.
