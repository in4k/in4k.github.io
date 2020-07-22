---
title: "Sound"
layout: "wiki-page"
---

No demo is complete without sound! Here are some specific resources and tools to help you add sound to your 1k, 4k or 8k demo:

# Tools

* [4klang](http://4klang.untergrund.net/) by Alcatraz
    * [pouet release](http://www.pouet.net/prod.php?which=53398)
    * [pouet forum thread](http://www.pouet.net/topic.php?which=10480)
    * [4klang-assets repo](https://github.com/in4k/4klang-assets)
* [64klang2](https://github.com/hzdgopher/64klang) by Alcatraz
* [AmigaKlang](https://www.pouet.net/prod.php?which=85351)
* [Axiom](https://github.com/monadgroup/axiom) by Monad
* [Buzzic](http://www.pouet.net/prod.php?which=54407) by stan_1901
* [Cinter](https://bitbucket.org/askeksa/cinter) by Blueberry (for Amiga 4ks)
* [Clinkster](http://www.pouet.net/prod.php?which=61592) by Blueberry
* [Fuxplux (4k Synth)](http://www.pouet.net/prod.php?which=13016) by Mostly Harmless
* [Ghostsyn](https://github.com/Juippi/ghostsyn) by faemiyah
* [GmDlsTool for Sound](http://www.pouet.net/prod.php?which=30541) by xplsv
* [Komposter](http://komposter.haxor.fi/) by TDA
* [Oidos](http://www.pouet.net/prod.php?which=69524) by Blueberry
* [PuavoHard Intro Music Composer](http://www.puavohard.net/php/prod/phpimc) by PuavoHard ([pouet link](http://www.pouet.net/prod.php?which=53671)) ([pouet thread link](http://www.pouet.net/topic.php?which=10793))
* [Sonant](http://www.pouet.net/prod.php?which=53615) by Youth Uprising ([javascript port](http://sonantlive.bitsnbites.eu/))
* [http://wurstcaptures.untergrund.net/music/](http://wurstcaptures.untergrund.net/music/)
* [SoundToy](http://www.iquilezles.org/apps/soundtoy/), sandbox by IQ to render audio using shaders.
* [Tunefish](https://github.com/paynebc/tunefish) by Brain Control
* [WaveSabre](https://github.com/logicomacorp/WaveSabre) by Logicoma

## Running Windows VST plugins in Linux

Most 1k/4k synths are distributed as Windows VST .dll's which you import into specific trackers.

If you're not running Windows notice that you can still use these Windows VSTs under Linux, using [wine](https://www.winehq.org/) and [airwave](https://github.com/phantom-code/airwave), simply install it using the method appropriate for your distribution then run 'airwave-manager', click "create link" (left most icon) select the vst dll under 'VST plugin' and the directory your DAW looks for VSTs under "Link location". Tested and working with 4klang and clinkster on ubuntu 14.04 LTS using wine 1.6.2.

There are other VST wrappers for Linux as well, e.g. this: [https://github.com/abique/vst-bridge](https://github.com/abique/vst-bridge) So if one of them doesn't like a certain plugin, you can always try the other. :)

## Using 4k synths like 4klang in Linux

See [the page on Linux](linux.markdown). Basically, see
[this](https://gitlab.com/PoroCYon/4klang-linux) and remember that the synths'
runtimes can only be used in 32-bit executables.

# Articles

Coding sound can be fun too! Here are some articles to help you make your own synth:

* [Auld's 4k synth](aulds-4k-synth)
* [Saida's 1024b Midi Sound System (masm)](saidas-1024b-sound-system)
* [iq's Reading gm.dls](reading-gm.dls)
* [Viznut's Algorithmic symphonies from one line of code](http://countercomplex.blogspot.pt/2011/10/algorithmic-symphonies-from-one-line-of.html)
* [pouet thread on timings](http://www.pouet.net/topic.php?which=10820)
* [Some 4klang notes on Alcatrazes' Equilibrium soundtrack by cce](http://www.pouet.net/topic.php?which=11327)

## FR08 Synth article series by KB

* The Workings of fr-08's Sound System, Part 1: The Concept
    * URL: http://www.kebby.org/articles/fr08snd1.html
    * Locally: [fr08snd1.html](http://in4k.untergrund.net/various%20web%20articles/fr08snd1.htm)
* The Workings of fr-08's Sound System, Part 2: Why are SMF Files Smaller?
    * URL: http://www.kebby.org/articles/fr08snd2.html
    * Locally: [fr08snd2.html](http://in4k.untergrund.net/various%20web%20articles/fr08snd2.htm)
* The Workings of fr-08's Sound System, Part 3: The Basic System
    * URL: http://www.kebby.org/articles/fr08snd3.html
    * Locally: [fr08snd3.html](http://in4k.untergrund.net/various%20web%20articles/fr08snd3.htm)
* The Workings of fr-08's Sound System, Part 4: Let's Talk About Synthesizers
    * URL: http://www.kebby.org/articles/fr08snd4.html
    * Locally: [fr08snd4.html](http://in4k.untergrund.net/various%20web%20articles/fr08snd4.htm)

# APIs

## DirectSound (part of DirectX)
* DirectSound 8.1 source code resources for MASM from Calebs' ASM-Site
    * URL: http://people.freenet.de/CalebsSite/
    * Files:
        * the x-includes - Locally: [Direct X 81 Includes.rar](http://in4k.untergrund.net/sound/Direct_X_81_Includes.rar)
        * explore the dsound - Locally: [Direct X 8 Dsound Example.rar](http://in4k.untergrund.net/sound/Direct_X_8_Dsound_Example.rar)
        * endless starway - Locally: [Direct X8 Endless Stairway.rar](http://in4k.untergrund.net/sound/Direct_X8_Endless_Stairway.rar)

# MIDI Resources
* (MASM) Intervall trainer Programmed by [-alloces-]:
    * URL: http://masm.by.ru/asm.html
    * Locally: [inttrain.zip](http://in4k.untergrund.net/sound/inttrain.zip)
* Moore's Guide to Programming MIDI Version 1.0
    * URL: http://www.programmersheaven.com/zone10/cat136/1147.htm
    * Locally: [MIDIPROG.zip](http://in4k.untergrund.net/sound/MIDIPROG.zip)
* The standard MIDI spec and file format
    * [Midi Spec](http://www.borg.com/~jglatt/tech/midispec.htm)
    * [Midi File format](http://www.borg.com/~jglatt/tech/midifile.htm)
* [random pouet thread about midi not working on x86 assembler](http://www.pouet.net/topic.php?which=10720)

# Sound Processing Resources
* Yehar's sound DSP tutorial for the braindead! version 1999.04.23
    * Hugi 19
        * URL: http://www.hugi.scene.org/main.php?page=hugi19
        * Locally: [hugi 19 - codsp.htm](http://in4k.untergrund.net/html_articles/hugi%2019%20-%20codsp.htm)
* [Music-DSP Source Code Archive](http://www.musicdsp.org/)
