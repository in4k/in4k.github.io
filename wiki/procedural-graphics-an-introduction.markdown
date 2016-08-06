---
title: "Procedural Graphics - An Introduction"
layout: "wiki-page"
---

## Procedural Graphics - An Introduction

This article is aimed at beginners or people simply interested how to squeeze three minutes of graphics into 4k. It won't help directly with code or advice on how to crunch data but will discuss some well known techniques in computer graphics for generating huge amounts of graphics data.

**A Work in progress...not complete (last updated 2006)**

## The Normal Approach: Explicit Graphics

Graphics consists of 2 main elements:

*   Textures
*   Geometry

Textures or images of any kind are normally stored as bitmaps, compressed or otherwise. Geometry is stored in a file where each point, normal, texture co-ordinate and so on are explicitly defined.

The main point here then is that the normal approach uses explicit specification of all graphical elements, stored in files.

## What are Procedural Graphics?

The basic idea is to create, on-the-fly all graphics. Data structures are created and then filled with data generated from the code itself, rather than being loaded from files as above.

In a little more detail, nearly all procedural graphics start with some very small base data. This base data should be as small as possible for 4k demos. The base data forms the seed for the code which then expands it into much larger pieces of data.

If you know how animation is done then this is a good example of precedural graphics. Imagine that an object is animated across the screen. We would not store all of the points the object goes to but only some key points. We then use code to generate intermediate points using spline based interpolation or something similar. The key points can be seen as the base data, the spline interpolator as the procedural graphics code and the resulting points the object is draw at as the procedural data generated.

So, procedural graphics has three main parts:

*   Base data, compact as possible.
*   Code that expands the base data.
*   The resulting generated procedural data.

## Special Considerations of 4k Demo Coding

Procedural graphics can be used in normal computer graphics too and animation is a good example of this. However textures and geometry might be created procedurally too. The difference with 4k is that the base data and the expansion code must also be as small as possible. Obvious really.

It is no use, for example, having a texture generation code that occupies 2.5kbytes as there will be no space left to do anything with these textures.

Similarly, its no good using 2.5k of base data for geometry if there is no space left for code to expand it.

_With 4k demo coding the base data and expansion code must be as small as possible_.

Of course we do not care how large the generated data is. In some senses, the larger the better though as many people have a _wow_ reaction when they see a lot of textures or a lot of geometry.

## Procedural Textures

Even the most highly compressed jpeg images are going to be a few kbytes and worse, the decoder is larger. Simply put, images stored as bitmaps are impossible in 4k.

We are forced to do procedural texturing.

### The General Idea

Textures are stored as texture descriptors. These descriptors describe _how to reproduce the texture, not the image itself._ They may, for example, describe a series of mathematical functions that applied in a specific order, reproduce the image.

### Simple Hard Coding

Of course, it is possible to write C code to simply "write" textures into an array. Three typical examples of such textures are checkerboards (or stripes), circles and random numbers.

This works because the code to do each one is much smaller than the storage required for the texture image. Hence it is worth procedurally generating these textures.

It is possible to build a whole library of such textures. Further, we could write routines to mix textures. For example, a checkerboard and a noise texture could be mixed simply by adding the resulting arrays from each.

Hard coded textures have the advantage that they require no base data but there are two drawbacks. Firstly they are fixed and so limited in the variety that can be achieved in a few bytes. Secondly, they are not so economic in bytes unless coded in assembler. In anycase they lose in flexibility and code space to true procedural textures.

### Shade Trees

Imagine looping over every pixel in a texture image. It would be possible to take the (x,y) values and process them mathematically to produce the colour value of the texture.

A shade tree goes much further. It is a tree structure in which each of the nodes is a mathematical routine. The leaf nodes of the tree are typically just the co-ordinates within a texture image. To evaluate the tree, we evaluate each branch in turn.

```
                 f6
                /  \
             f2      f3
            / \      / \
     f1(x,y) f2(x,y)f3  f6(x,y)
                   / \
            f5(x,y) f6(x,y)
```

The above structure tells the evaluator:

```
f6 ( f2 (f1(x,y), f2(x,y)), f3 (f3 (f5(x,y),f6(x,y)), f6(x,y) ) );
```

f1 to f6 are simple mathematical functions. Thus we loop over every pixel and evaluate the whole tree for each pixel.

This is an example of a shade tree.

Incredibly complex textures with surprising variety can occur using such a structure and most texture generators are done using some variation of a shade tree.

Remember though that for 4k we cannot store textures. Many of the texture generators available will use shade trees and then write a texture as output. This is no use in 4k and instead the texture descriptor is written. The base data, or texture descriptor would be some description of the tree sructure and the mathematical fcuntions at each node. Typically, the mathematical functions are hard coded as in the previous section and all that is stored in the descriptor are references to them.

If you are interested, I show source code for a simplified shade tree in my article:[Aulds Texture Generator](/index.php?title=Aulds_Texture_Generator "Aulds Texture Generator").

### Genetic Algorithms

Robert Cook introduced shade trees in 1984\. In his work, trees were designed by hand. This is fine if you know what (sin(exp(add(x,y)) looks like, and some people in computer graphics do, but most people do not.

To overcome this Karl Sims introduced a technique using genetic algorithms to create shade trees. Imagine for a moment that shade trees are generated randomly. That is the structure and the functions at the nodes are all assigned randomly. The result would be a random shade tree. The problem is that a very great many of the trees are not interesting: the resulting texture can be blank or single coloured. This happens very often.

Genetic algorithms is a technique for searching the huge space of possible textures and finding interesting ones. Although it is a huge subject, I shall attempt to describe it briefly.

A generation of textures is created randomly. Lets say 12 textures. These are displayed on the screen and the viewer asked to chose interesting ones interactively. Lets say that two are interesting. Now the computer creates a new generation of textures using the two chosen ones by breeding them and mutating them. To breed two textures, we take parts of their "DNA" and swap them. The "DNA" for the texture is its texture descriptor, so in essence a new texture descriptor is created by taken part of one descriptor and a complementary part from another descriptor. This is how textures are bred. To mutate a texture, we randomly change a small part of its texture descriptor. For example we might change a sin function for a tan function. Alternatively we might add or substract branches from the tree.

By using "genetic" manipulation of this kind, interesting textures "evolve" quickly. There are, of course, many ways to tune this genetic algorithm bu this is the basics.

One important point here is that the complexity of the genetic algorithm and the code for it is all outside the 4k demo. It is in a tool used to create the texture descriptor, not in the 4k demo itself. This means we can make the creation of texture descriptors as complex as we wish but evaluation, the part that generates a texture image from the descriptor is still kept simple and small. This is what we want.

### IFS

An iterated function system can be used to generate images of natural shapes such as leaves, trees and so forth or simple abstract fractal patterns. It is fairly limited in scope but can be implemented, including base data in about 200 bytes in C so it is very cheap.

Add more... What is an IFS Explain the base data, expansion code and resulting data

## Procedural Geometry

By now it should be obvious what procedural geometry is likely to be.

### The General Idea

Geometry is specified by a geometry descriptor which can be very small. A piece of code expands the geometry descriptor into points, polygons, normals and texture co-ordinates. It is not as common or obvious ho to define geometry usinf procedural methods but below are a few common methods.

### Simple Hard Coding

Compare with texture hard coding where specific, simple, textures can be produced easily. Then we could write routines to create, say, a sphere, a cube, a torus etc.

Again some of these routines can create complex shapes too, for example a p-q torus. One of the most common is to create a sphere where the vertices are modulated by a sin wave over time, giving a 3d animated flower like shape.

Routines can be combined to produce more complex objects. For example an arm can be constructed from two ellipsoids.

Nontheless, there are limits to this technique and at a basic level, simple geometric shapes are normally the result.

### "Multiplicative" Geometry

A comon technique used to combine simple geometric shapes is sometimes called multiplicative geometry. The idea is to repeatedly draw the same base shape over and over, constructing a large complex scene of geometry from some simple base geometry.

Again compare this with the shade trees used for texturing where simple basic outines are combined to form a more complex texture.

However in the case of multiplicative geometry, we will often use loops or recursion.

A simple example:

```
for z = 1 to 10
 for y = 1 to 10
  for x = 1 to 10
     translateTo (x,y,z);
     drawSphere(0.75);
  end
 end
end
```

This example draws a large cube of spheres but the only explicitly defined geometry is the small cube itself.

This technique extends very well and if routines are written using time as an input, can also be used to animate. Snakes, loops, spirals, cubes, planes and many other shapes can be created and remember each point has some base geometry. When this geometry intersects itself, it often beomes unrecognisably complex. This is just what we want.

### Genetic Algorithms

Similar to shade trees and genetic algorithms for textures, the same techniques can be used for geometry. A shade tree can have as leaf nodes, vertices from a 3d model, say a sphere. Passing each vertex from the sphere through the tree results in a new set of points. Then, normals are calculated for the new 3d model and it is displayed.

To my knowledge, I was the first person to publish this idea in "Genetic programming for easy 3D texture generation" in the Proceedings of 13th Annual Conference of Eurographics (UK Chapter). Since then more work has been done and its a fruitful area for research but not proven useful yet.

### Isosurfaces or MetaBalls

Imagine you are trying to plot a path for a space ship between some planets. The path that uses least fuel would be one where the gravity is the same strength along the path. This is easy to imagine in 2d but in 3d this path also exists. Further, in 3d its not just a path but a surface: an isosurface.

This isosurface of equal gravitational pull is usually a very interesting shape and as planets move, changes shape in a fluid and complex way, even splitting into multiple isosurfaces and joining again. If we render that surface and apply lighting or textures to it, it forms a very interesting demo.

The descriptor is obviously a set of points in three space and their potential, or gravitational pull. The expansion code would find the surface of a given potential. If we wish to have polygons result from this, we could use Marching Cubes Algorithm. The resulting procedural geometry would be a set of triangles representing a surface of equal potential (isosurface).

Perhaps sometime I'll add details on Marching Cubes but there are plenty of details on the web.

If you think about it, a single point in space will result in a spherical isosurface. A ball, if you like. Sometimes, therefore, a point with potential is called a metaball.

### L-systems

Add something about Lindenmeyer systems here...

## Why Scene Graphs are not Procedural

Scene graphs are trees of graphic state and geometry. Its beyond this article to describe them in-depth but one thing worth noting is however, that scene graphs are not procedural geometry. They define a fixed geometry, possibly with some animation rules, but fixed nontheless. Every piece of geometry in the scene graph is explicitly defined.

Of course this doesn't preclude having scene graphs in 4k demos but as they don't possess any expansion code, they don't buy us the extra that true procedural geometry does.

## Limitations of Procedural Methods

If you have watched a lot of 4k demos, you may have noticed there are some general areas that are popular. The "sameiness" of many demos is explained by the limitations of prcedural methods.

When we generate textures procedurally, it is not really possible to represent specific images (say a house) but only abstract patterns (checkerboard, noise etc.). This means that 99% of demos will have textures with patterns rather than, for want of a better term , pictures. There are some notable exceptions to this rule where for example a face is stored but this is done using vectors, not images.

The same is true of geometry. Recognisable "characters" are rare in 4k demos and if they are present, are often cartoon-like (this being the least geometry). Structures tend to be abstract and repeating. If not, it becomes obvious that the demo is very limited in scope. Hence the reason city scapes are so popular, being series of repeating buildings.

Procedural methods therefore are good for abstracts but are difficult to force into specific recognisable images or geometry.

As always its a trade off. Mixing procedural methods with explicit methods can result in, for example, a field of goblins. Each goblin would be defined explicitly in a scene graph but the position of each one, the animation and even some of the textures deorating them could be procedural. The grass would be procedural, the weather procedural and so on.

Anyone for Lord of the Rings in 4k?

-- auld
