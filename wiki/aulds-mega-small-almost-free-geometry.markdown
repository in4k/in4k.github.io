---
title: "[Auld's Mega Small (Almost Free) Geometry"
layout: "wiki-page"
---

## Free Geometry in GLU

Geometry uses bytes. If only OpenGL had a lot of primitives for free. Actually GLU does. Its just they are well hidden. For example, did you know GLU has a cube? A triangle? An arrow head? A pacman?

### The GLU Primitives

GLU has four simple primitives: sphere, disk, cylinder, partial disk. Each of which has parameters that can force unusual results. The most well known is that the cylinder can also be a cone but there are far more. Here are some examples.

### Some C Code Examples

```
// setup code
static GLUquadricObj *q;
q = gluNewQuadric();
gluQuadricNormals (q,GLU_TRUE);
gluQuadricTexture (q,GLU_TRUE);
```

First of all, lets look at examples with gluDisk.

```
// the gluDisk based shapes
gluDisk (q, 0.0, 0.8, 30, 1);  // Solid Circle
gluDisk (q, 0.6, 0.8, 30, 1);  // ring
gluDisk (q, 0.6, 0.8, 4, 1);   // hollow square
gluDisk (q, 0.6, 0.8, 3, 1);   // hollow triangle
gluDisk (q, 0.0, 0.8, 3, 1);   // solid triangle
gluDisk (q, 0.0, 0.8, 4, 1);   // solid square
gluDisk (q, 0.0, 0.8, 6, 1);   // solid Hexagon
gluDisk (q, 0.6, 0.8, 6, 1);   // hollow hexagon (think beehive)
```

Now the Sphere using gluSphere.

```
// the gluSphere based ones...
gluSphere (q, 0.8, 20, 20);    // a sphere
gluSphere (q, 0.8, 20, 20);    // a smartie ...when scaled of course
gluSphere (q, 0.8, 20, 20);    // an ellipsoid...same here, fix later
gluSphere (q, 0.8, 3, 20);     // a seed like shape?
gluSphere (q, 0.8, 4, 20);     // an ikea paper lamp?
gluSphere (q, 0.8, 4, 2);      // diamond
gluSphere (q, 0.8, 3, 3);      // triangular crystal
gluSphere (q, 0.8, 6, 3);      // quartz crystal
```

Thirdly the Cylinder using gluCylinder.

```
// some interesting cylinders
gluCylinder (q, 0.4, 0.4, 0.8, 20, 20);  // cylinder
gluCylinder (q, 0.4, 0.4, 0.8, 4, 20);   // square tube
gluCylinder (q, 0.4, 0.4, 0.8, 3, 20);   // triangular tube
gluCylinder (q, 0.4, 0.4, 0.8, 6, 20);   // hexagonal tube
gluCylinder (q, 0.4, 0.0, 0.8, 20, 20);  // cone
gluCylinder (q, 0.4, 0.0, 0.8, 4, 20);   // square base pyramid
gluCylinder (q, 0.4, 0.0, 0.8, 3, 20);   // triangular based pyramid
gluCylinder (q, 0.4, 0.0, 0.8, 6, 20);   // hexagonal based pyramid
```

Lastly gluPartialDisk.

```
// some partial disk shapes.
gluPartialDisk (q, 0.0, 0.8, 30, 1, -45, 270);  // pacman
gluPartialDisk (q, 0.6, 0.8, 30, 1,0,180);      // half ring
gluPartialDisk (q, 0.6, 0.8, 3, 1,0,270);       // square ring partial
gluPartialDisk (q, 0.6, 0.8, 2, 1,0, 240);      // triangle ring partial
gluPartialDisk (q, 0.0, 0.8, 2, 1, 0, 240);     // arrow head 
gluPartialDisk (q, 0.0, 0.8, 3, 1,0,270);       // square partial
gluPartialDisk (q, 0.0, 0.8, 5, 1,0,300);       // Hexagon partial
gluPartialDisk (q, 0.6, 0.8, 5, 1,0,300);       // Hexagon ring partial
```

There may be a lot more interesting ones to explore in the partial disk area.
