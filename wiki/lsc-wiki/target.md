
## Some notes about the target platform

We'll mostly target the Revision compo rules, as it's the biggest party, and
they seem to be the most strict.

* ***The distro is always the latest Ubuntu***
* ***CODE WILL NOT BE RAN AS ROOT***
* These packages are installed by default: [18.10
  ](http://releases.ubuntu.com/18.10/ubuntu-18.10-desktop-amd64.manifest),
  [19.04](http://releases.ubuntu.com/19.04/ubuntu-19.04-desktop-amd64.manifest).
* Note that only the 64-bit versions of these libraries are installed. *If
  you're running a 32-bit binary, it **MUST** be static!*
* No SDL. Or maybe [not anymore?]($docroot$lsc-wiki/party/revision/sdl.html)

