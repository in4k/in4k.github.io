---
title: "Auld's IFS for 4kb"
layout: "wiki-page"
---

## What is an IFS ?

An iterated function system is a number of transformations, repeatedly applied to a point. Each time the point is transformed a new, random, transform is chosen from the set of possible transforms and the process repeated. After each transform the point is drawn. The result, with correct transformations and enough iterations can be amazing.

Here are some example images, created using an IFS.

_Add images_

## How does an IFS Work?

The C code at the heart of an IFS is:

```
 newx = a*x + b*y + e;
 newy = b*x + c*y + f;
```

The co-efficients come from the equation:

```
V' = V.M + T
or
|x'|   |x|   |a b|   |e|
|y'| = |y| x |c d| + |f|
```

Which is essentially a 2d transformation of an arbitrary point. However, repeating a single transform many times will only result in simple movement of a point. The first key point about an IFS is that each IFS has a number of 2D transforms. Each iteration, one of the transforms is chosen and applied to the point. The fern example uses 4 sets of co-efficients.

The next key thing to understand is that if a transform is selected completely randomly from the set of transforms, it will take many iterations to produce an image. How many? Perhaps 1,000,000! This is because a point is often tranformed to the same position it was in previously, meaning the same point is plotted and there is no visible improvement in the image. How can this be improved? A bias is applied so that transforms are chosen more often that will move the point to new locations.

The last key element to understand is the idea that when the IFS starts, it takes a few iterations for the IFS to settle down and begin drawing the correct points. Normally, therefore, IFS code is complicated by a few extra iterations which are not plotted at the start of the execution.

## Squeezing an IFS Down for 4k Demos

In this section, a number of techniques for reducing the size of IFS code are discussed. The result is code and data for the famous fern IFS which fits into around 250 bytes of compressed binary. One of the resulting functions is generically useful for things like implementing an animation system. All code is in C.

### Fixing the Number of Sets of Co-efficients

Key point one above implies that any given image produced by an IFS can use any number of transforms. However, there are an amazing variety of images that can be produced using just two transforms. More so 3 and an astonishing number with 4\. In the 4k code below therefore we fix the number of co-efficient sets (transforms) to be 4\. This is sufficient to draw all manner of plants, leaves, abstracts, fractal shapes, spirals, dragons and so on. In particular it draws two of the most famous IFS images: the fern and the maple leaf.

Its possible to define _less_ than 4 sets of co-efficients of course, if your image doesn't require them. I'll explain how later.

### Avoiding the Initial Iterations

The actual image we produce using an IFS is known as the attractor. It is the set of interesting points. Other points are not intersting. The peculiar thing about IFS is that once you find one point on the attractor, the rest of the iterations will always produce another point on the attractor.

Normally, we iterate a few times and do not plot the points which are produced until we reach the attractor. This complicates the code.

The best way to avoid this is simply to *know* ahead of time, one point in the attractor. This is possible to work out mathematically but its a lot of maths. Better is simply to draw the image, starting with any point and then redefine your start point closer and closer to the attractor yourself by observation. This means we pass the initial point into the IFS code. For the Barnsley fern for example, (0,0) can be used as the initial point.

### Co-efficient Ordering

It is tempting to define 6 arrays for a,b,c,d,e,f..something like

```
float a[4] = { 0.5, 0.0, ...
float b[4] = { ....
```

This will make copying sets of co-efficients from web pages very easy. However, the compiler will produce bad code for this. Much better is to define a single array with all your co-efficients. Further, it is better for the compiler, to define in the order they are used in the equations: a,b,e,c,d,f.

Thus, an IFS with 4 transfoms would have an array with co-efficients:

```
a1,b1,e1,c1,d1,f1,a2,b2,e2,c2,d2,f2,......,d4,f4
```

### Iterations and Loops v Recursion

I have found that simple tail recursion is more byte efficient than a loop counting through the iterations. This was tested with GCC.

### Avoiding IFs and SWITCH

Much of the code on the internet to draw an IFS has something like:

```
if (between(0.0,0.25)) use transform1;
else if (between(0.25,0.5)) use transform2;
else if ...
```

This produces a lot of bytes in the compiled code. Instead, better is to write a function to chose the correct set of co-efficients to use and then use this to index into the array described previously. The function for chosing the correct set of co-efficients to use is described below...

### How to chose the Random Transform

We are trying to implement a probability distribution function. The probabilties for the 4 sets of co-efficients we use can be found on the internet. So we generate a random number and test against the ACCUMULATED probabilities.

For example if our 4 transforms have the following percentage chances

```
10% 50% 25% 15%
```

of being chosen, then the cummulative values would be

```
10, 60, 85, 100
```

We then chose a random number between 0,99 and check which pair of numbers we are between. In the (inefficient) code below we use rand()%100\. This can be better written as rand()&99\. However it can be less code.

The rand() function returns a value between 0..32767 normally (you need to check on the given platform but this is usually safe). Therefore scaling our cummulative probabilities to be between 0..32767 results in not needing to use &100.

Theoretically this is much better as rand(), as normally implemented, is less random in the lower order bits and doesn't give good randomness when used in conjunction with mod. I digress.

NB this is not implemented in the code sample below.

### Scaling the resulting Points

Normally code on the internet to do IFS will have a small section to scale the resulting points, transformed and ready to plot, to an area of the screen that is useful. To get rid of this code, you can pre-scale the IFS co-efficients. To do so, you need to know which ones to alter.

_add some theory later...when the wifes in bed_

Basically, a,b,c and d contain scaling factors. Pre-multiplying these gives you a smaller and smaller (or bigger and bigger) resulting image. Of course e and f are the translation factors which can be altered to put your scaled fern anywhere you wish.

## A C implementation for 4k Demos

```
 const static float fernifs[] = {0.0 ,0.0 ,0.0 ,0.0 ,0.24,0.0,
                                 0.0,-0.24,0.0,0.24,0.24,0.12,
                                -0.12,0.24,0,0.24,0.24,0.12,
                                 0.85,0.0,0.0,0.0,0.85,0.12};
 const static uint prob[] = { 1, 8, 16, 100 };
```

```
 static uint chose (const uint choice, const uint *probability) 
 {
 int i=-1;
   while (choice > probability[++i]);      
   return i;
 }
```

```
 static void ifsPlot (float x, float y, const uint iter)
 {  
 const float xc=x;
 const float yc=y;
   const float *ifs = &fernifs[ 6 * chose(rand()%100, prob) ];
   x = xc*ifs[0] + yc*ifs[1] + ifs[2];
   y = xc*ifs[3] + yc*ifs[4] + ifs[5];
   plot(x,y);
   if (iter) ifsPlot (x,y,iter-1);
 }
```

**Please credit me if you use this code or a derivative**

## How to use this code and define Less Sets of Co-Efficients

To keep memory use down, it may be necessary to define an IFS with less than 4 sets of co-efficients.

Obviously the first step is to remove the direct reference to ifsfern and replace with a parameter passed into the function. Secondly the probabilities should be passed as a parameter too.

Lastly define probabilties that force the choice function to terminate after the desired number of transforms.

So, if we had a spiral IFS with only 2 transforms, the first 20% likely , the second 80%, we could use:

```
const static uint prob[] = { 20, 100, 100, 100 };
```

Actually nothing in the code prevents any number of transforms being used. However, both prob and ifsfern could be re-used if a maximum of 4 transforms is assumed.

## Exploiting OpenGL for Smaller IFS Code

_Add this later..._

## Compressing The IFS Co-efficients for 4k Demos

Firstly you will notice that co-efficients given for the fern example above are nothing like those given on the internet. Of course they are re-arranged but more than that there is a coherency about them.

Firstly, I have straightened the fern out, removing bends in it. This results in more co-efficients being zero, leading to better compression. I don't need a bent fern.

Secondly, the distance between fronds of the fern and their angles can be approximated without ruining the overall ferniness (?) of the resulting image. **This means numbers can be grouped into similar quantities (like 0.24). This is the key to compression of co-efficients.** The co-efficients given in the code sample, though compressing much better than the original Barnsley fern co-efficients, nontheless result in a good looking fern.

Lastly, I have altered the scale of the fern directly in the numbers (as I have no scaling when I plot the point) so that it fits into a well defined area of the screen which I then capture using OpenGL, into a texture.
