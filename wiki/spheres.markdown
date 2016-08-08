---
title: "Spheres"
layout: "wiki-page"
---

Demos are full of spheres. I'm going to worship the sphere for a while here. Its useless meanderings but you might find something useful.

The [equation of a sphere](http://mathcentral.uregina.ca/QQ/database/QQ.09.03/jaidev1.html "http://mathcentral.uregina.ca/QQ/database/QQ.09.03/jaidev1.html") is very simple but is not a lot of use to us in our intros.

Opengl built in spheres are created using

```
gluSphere(q,r,s,t);
```

Forget q, its just a quadric. r is the radius. s an t are the number fo slices in the sphere (longitudenal and latitudenal). By increasining the numbers the sphere is more highly tesselated. At low numbers (3,4,5,6...) crystal like structures can be formed.

Sometimes though you want your create your own sphere, maybe so you can distort the vertices to do one of the so-called spherical harmonics shapes. The most obvious way to tesselate a sphere is to break it into lattitudenal and longitudenal cross sections, sort of making a sphere from a series of circular cross-sections. Here is a great article on this which results in optimised tri-strips. [Standard Sphere Tesselation](http://in4k.untergrund.net/html_articles/hugi_27_-_coding_corner_polaris_sphere_tessellation_101.htm "http://in4k.untergrund.net/html articles/hugi 27 - coding corner polaris sphere tessellation 101.htm")

One of the problems with spheres made in this way is that at the top and bottom of the sphere there are more points (each circle has the same number of points and at the top and bottom the circles are very small compared to at the equator). One solution ot get equally spaced vertices is to divide an octahedron recursively.

[Tesselate An Octahedron to Make a Sphere](http://www.gamedev.net/reference/articles/article427.asp "http://www.gamedev.net/reference/articles/article427.asp")

This gives a much more pleasing sphere effect.

It's also a good idea to create a high tesselated cube (or even just six planes) and normalize the vertices to create a sphere. Not only it gives you a more or less good distribution of points, but also it provides a natural parametrization without degeneracies for texture mapping.

Sometimes though, you dont need a solid sphere and simply some set of points on the sphere will do. This is an [enormous topic of research](http://ogre.nu/sphere.htm "http://ogre.nu/sphere.htm") - efficient ways to generate a set of points on a sphere with even distribution. However here is one solution:

[Random and evenly distributed points on a sphere](http://root.cern.ch/root/html/examples/spheres.C.html "http://root.cern.ch/root/html/examples/spheres.C.html")

As we are size coders here, here is an almost minimal example of input to POVRay to display a sphere on a plane with nice lighting.

```
union{sphere{z*9-1,2}plane{y,-3}finish{reflection{,1}}}background{1+z/9}
```

which is originally from Tekno Frannansa and the image can be found [here](http://local.wasp.uwa.edu.au/~pbourke/rendering/scc3/final/cjjqmo.html "http://local.wasp.uwa.edu.au/~pbourke/rendering/scc3/final/cjjqmo.html"). Its just 72 bytes of "code".

Spherical harmonics on a sphere can be done on the CPU or on the GPU. Accessing the co-ordinates of a gluSphere and distorting them using sins and cosines can be done in even 1k GLSL shaders.

[1k sphereical harmonics with ambient occlusion style lighting](http://dbfinteractive.com/index.php?topic=592.0 "http://dbfinteractive.com/index.php?topic=592.0") 1ks using this technique can be seen

[here](http://www.intro-inferno.com/production.php?id=1340 "http://www.intro-inferno.com/production.php?id=1340"),[here](http://www.intro-inferno.com/production.php?id=1321 "http://www.intro-inferno.com/production.php?id=1321") and [here](http://www.intro-inferno.com/production.php?id=1325 "http://www.intro-inferno.com/production.php?id=1325").

Ray tracing a sphere is trivial and forms the basis of many intros. There are two basic technqiues. One is less code and one is faster. Here is a comparison. [Raytracing a sphere by algebraic and geometric methods](http://www.devmaster.net/wiki/Ray-sphere_intersection "http://www.devmaster.net/wiki/Ray-sphere intersection")

Lastly, probably the best ever rendering of spheres in a demo can be found [here](http://www.pouet.net/prod.php?which=18766 "http://www.pouet.net/prod.php?which=18766") and beautiful non real-time renderings of spheres are [here](http://www.morphographic.com/Sphere.htm "http://www.morphographic.com/Sphere.htm").
