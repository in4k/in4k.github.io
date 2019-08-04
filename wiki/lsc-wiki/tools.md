### dynamic linkers

* ~~[elfling](https://github.com/google/elfling)~~ (currently broken)
* ~~[bold](http://www.alrj.org/pages/bold.html)~~ (currently broken)
* [dnload](https://github.com/faemiyah/dnload) (Linux, FreeBSD)
* [smol](https://github.com/Shizmob/smol) (glibc Linux, with non-glibc
  "compatibility mode")

### compressors

* shell dropping: `cp $$0 /tmp/M;(sed 1d $$0|lzcat)>$$_;exec $$_;`
* ~~[fishypack
  ](https://bitbucket.org/blackle_mori/cenotaph4soda/src/master/packer/?at=master): In-memory `memfd`/`execveat`-based decompressor~~ (Deprecated in favor of vondehi)
* [vondehi](https://gitlab.com/PoroCYon/vondehi): Next-generation in-memory `memfd`/`execveat`-based decompressor (See list of known bugs)
  * [autovndh](https://gitlab.com/snippets/1800243): script that bruteforces all possible gzip/lzma/xz/... options and automatically places a vondehi decompression stub at the beginning
* [oneKpaq](https://github.com/temisu/oneKpaq): Generic PAQ-based (de)compressor for 32-bit x86

### synths

* [ghostsyn](https://github.com/Juippi/ghostsyn)
* [4klang](https://www.pouet.net/prod.php?which=53398), [ForkedKlang
  ](https://www.pouet.net/topic.php?which=11312). Note that the replayer code
  is 32-bit only! You'll need some hacks to make it work on 64-bit.
* [Clinkster](https://www.pouet.net/prod.php?which=61592), replayer is 32-bit only as well.
* [Oidos](https://www.pouet.net/prod.php?which=69524), replayer is once more 32-bit only.
* [Axiom](https://github.com/monadgroup/axiom/) is "supposed" to work

See [here]($docroot$lsc-wiki/code.html#example-code_basic-synth-setup) for some synth example setup code, together with some Linux-specific patches to get some of the replayers to work.
