## Talks

### unlord (LCA 2019)

[pdf](https://people.xiph.org/~unlord/LCA2019.pdf); [video](https://www.youtube.com/watch?v=J5WX-wN_RKY)

### Shiz & PoroCYon (Revision 2019)

[pdf](https://pcy.ulyssis.be/pres/Lin.pdf); [video](https://www.youtube.com/watch?v=a03HXo8a_Io)

#### Errata

* `-march=haswell` should be **`-march=core2`**,
  the latter *usually, but not always,* generates smaller code
  ([source](https://github.com/faemiyah/dnload#compiler-flags))
* Ferris' "Squishy" packer isn't exactly '*general-purpose*'
  as compared to kkrunchy, but the approach is *more general* at least.
* runit and s6 are perfectly fine at supervising daemons.
* ryg/fr made kkrunchy, not kb/fr.
