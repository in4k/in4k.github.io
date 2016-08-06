---
title: "Combining IFS and Particle Systems"
layout: "wiki-page"
---

**An Iterated Particle System?**

The idea here is simple. I aim to show that a particle system (PS) and an Iterated Function System (IFS) are basically two special cases of a more abstract entity: an iterated particle system. It might be possible to code the IPS so that it uses less memory than an IFS and a PS would together. Further I'm aiming this at 4K development so we won't add in lots of bells and whistles to the system however, it would certainly be possible in 64K demos to extend this.

Algorithm for a particle system

```
Create a volume in 3 space
Create a set of particles randomly in volume
Assign age to each particle
Create Transform for each particle
While particles not too old
    Translate the particles in space
    Rotate the particles in space
    Add gravity to particle
    Age the particles
    Draw the particles
```

Psuedo code for the IFS

```
Create random 2d space
Create a single random point in that space
Create S sets of (a,b,c,d,e,f) co-efficients with probabilities
For an arbitrary time
     Chose a,b,c,d,e,f from set S using probability  
     newx = oldx * a + oldy * b + e
     newy = oldx * c + oldy * d + f
     Draw the point
Next
```

Observations:

*   2d is a special case of 3d where all z co-ordinates are equal.
*   The equation for IFS is exactly a 2D matrix multiply which can be written using 3D matrices
*   1, the number of points tranformed in the IFS is a subset of the N particles transformed in each step of the particle system
*   IFS drawing is usually done just by plotting a pixel in 2D. This is a special case of plotting a point in 3 space...a typical way of drawing a particle system
*   The age of particles can be set arbitrarily as is the number of iterations of the IFS
*   A particle is rotated and translated by a fixed transform that is the same on each iteration. This is a special case of the IFS which the co-efficients for transform come from a set and are biased by probability. The probability is obviously 1.0 and the possible number of co-efficients is simply 1 in the particle system case.

Using these observations we could write an iterated particle system thus:

```
Set up volume for particle creation
Create 1..N particles in volume
Set up 1d array of S probabilities
Set up 2d array of 3d matrices (NxS)
Assign each particle a maximum age
Repeat
   For each particle p
      Chose Matrix (p,1..S) to apply from probabilities 
      Translate and Rotate Particle
      Apply gravity to particle
      Draw Particle
      Age Particle
   Next Particle
Until all particles are "dead"
```

Reading the above carefully it should be possible to see that an IFS is now:

*   (1xS) 3d Matrices set up with a,b,c,d,e,f co-efficients
*   A corresponding set of probabilities
*   The volume is the square area 0.0..1.0 in x and y with zeroes for z
*   A single particle created in this "volume"
*   A single age assigned to the particle equal to the arbitrary number of iterations of the IFS (100,000 or more)
*   Gravity set to a vector of (0,0,0)

Note that the 3d Matrix in the IFS case is actually of the form:

```
a b 0 0
c d 0 0
0 0 0 0
e f 0 1
```

which forces it in effect to be a 2d transform. Of course extending to a 3d IFS is now trivial. The zeroes compress well.

A typical Particle system is:

*   An array of 3d Matrices (Nx1)
*   A single probability of 1.0, forcing the choice of the matrix
*   A volume (say 0.0..0.1 in x,y,z)
*   N particles with corresponding ages created within this volume
*   Gravity set to any vector e.g. (0,-1, 0)

There you have it...an iterated particle system, a new concept which is a superset of IFS and Particle systems. We have unified the data types and the code leading to a smaller solution. Of course the resulting code is bigger than either the IFS or PS optimised code by itself but is smaller than the two together.

* * *

Comments:

Technically of course, gravity should be applied to an accelerating force, not to the position of a particle. Its relatively easy to extend the above to cope but in practice it makes only a small visible difference.

Particles may change the way they are drawn over time. To avoid cluttering the generic algorithm with this and making the code generic, its possible to add a pointer to a draw function into the particle. This function would take the particle as a parameter. In the IFS case the age would be ignored and the code would plot a point. In the case of a PS it could use the age to alter the colour of whatever particle it is we are drawing.

It may be necessary to create new particles when old ones die. This is not done in IFS and would require the use of a flag in setting up the system to indicate if this was necessary or not.

Maybe one day I'll code it and put the code here.

**If you use this idea please credit me** --auld
