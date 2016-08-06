---
title: "MacOS X"
layout: "wiki-page"
---

_THIS PAGE IS OUTDATED, [TDA](http://www.pouet.net/groups.php?which=976&order=party) HAS BEEN RELEASING MAC 4K's SINCE THEN_

* * *

The OS X native executable format, [Mach-O](http://developer.apple.com/documentation/DeveloperTools/Conceptual/MachORuntime/index.html "http://developer.apple.com/documentation/DeveloperTools/Conceptual/MachORuntime/index.html"), is structured around fixed 4K segments, each of which may be CODE, DATA or one of several other types. For this reason, it would be difficult if not impossible to write a 4K intro directly as a Mach-O executable; more likely, a Mac 4K would be deployed as a shell script / Perl / Python / Ruby file dropper.

In order for a shell script to run as a clickable icon in Finder, it needs to be put inside an application bundle. Create the folder structure MyIntro.app/Contents/MacOS, put your script named MyIntro into the MacOS folder, and use chmod to set it as executable.

* * *

In fact, scripts do not work for .app bundles, it has to be Mach-O executable. But .app's are not the only items that can be executed from Finder, you can also rename a script to .command, and it should be executed as expected (though you lose stuff like icons, but I doubt that is important for 4k's).

Also note that unix tricks also work, such as gzexe. --scoopr

* * *

I'm not really familiar with the inner workings of OSX but I know something about Classic MacOS and Unix. Some notes:

*   In classic MacOS, you only need the CODE segment in the resource fork, and you can access all the low-level system resources you need for a 4K intro from there. However, it may not be possible to run PPC code from this environment, as PPC code needs to be located in the data fork. So you basically require a dropper/decompressor stub written in 68K code in order to make a PPC-based 4K intro for the pre-OSX macs.
*   If you can use scripts in OSX, check out the new content in [Linux](/index.php?title=Linux "Linux") article for some ideas. Perhaps a script that drops (or even generates or compiles) a Mach-O executable and starts it is the way you have to go with. If the format has a lot of stuff that doesn't compress well, you may have to manually reduce everything you can in order to improve the compressibility. (In my experience, most linkers in all operating systems produce something extra that can't be eliminated by merely altering the linker parameters).
