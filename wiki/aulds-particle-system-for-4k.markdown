---
title: "Auld's Particle System for 4kb"
layout: "wiki-page"
---

There are many, many ways to write particle systems. My style is to write something that is a recognisable particle system, with some flexibility but occupying as few bytes as possible. Here are the features.

So far the system can achieve explosions, fountains and sprays, waterfalls, twinkling star fields, moving star fields, and even animating an object across the screen (single particles can be useful too!).

There are dozens of possibilities such as growing grass that I haven't tried yet.

### The Features

*   2d or 3d
*   Gravity in any direction and strength
*   Controllable direction vector
*   Controllable age range
*   OpenGL/D3D agnostic (use either)
*   Flexible particle drawing
*   Particles are recreated after dying
*   Control of time a particle stays "dead"
*   Creation of particle in 1,2 or 3 space within a given random setting
*   ~350 bytes
*   C code
*   Multiple particle systems at once on screen

### What Limitations are There?

*   No rotation of the particles: this is a serious omission and I'll have to fix it in the future
*   No moving of the particle source
*   All particles in a single system are drawn the same way but different systems can draw particles in different ways at the same time

I think the code is quite long already and I wonder if someone can do much better? It would be great to hear from you if your system is much smaller.

## The C code for 4k Demos

```
#define MXPARTS  2048
```

```
//random number 0.0..1.0
static float pfrand(void) {return rand()/(float)RAND_MAX;}
//random number -1.0..1.0
static float frand(void) { return 1.0-(pfrand()*2); }
```

```
// a function to find a random number within a floating point vector
// 0..m[i]
static void randV3 (float *p, const float *m) {
uint i=3;
    do { *p++ = frand()**m++; } while (--i);
}
```

```
// add vector a to v and then v to p
// point += velocity += acceleration
static void addV3 (float *p, float *v, const float *a) {
uint i=3;
    do { *p+++= *v+++= *a++; } while (--i);
}
```

```
typedef struct particle_struct {
       uint age;   
       float p[3],v[3];     // p=pos, v=vel 
}particle;
```

```
static particle p1[MXPARTS];
static particle tmpp;
```

```
typedef struct psystem_struct {
     const float g[3], p[3], v[3];
     const uint np, mxage, mnage;
     void (*drawfp)(const particle*); //func pointer used for drawing
     const particle *parts;
} psystem;
```

```
// pre-declaration ... maybe this can be removed with better code
static void drawPart (const particle*);
```

```
psystem staticstars ={
       0.0, 0.0, 0.0,   //gravity
       1.0, 1.0, 0.0,   //2d random initial position
       0.0, 0.0, 0.0,   //velocity
       200, 200, 10,    //number of stars; max age, min age
       drawPart,        //how to draw this star
       &p1[0]           //array of particles ot use
};
```

```
static void drawPart (const particle *part)
{
// could be anything here, using age of particle, D3D, anything...
      glBegin(GL_POINTS);  glVertex3fv(part->p);  glEnd(); 
}
```

```
static void setParticle ( const psystem *sys, particle *part )
{
    // velocity and position
    randV3 (part->p, sys->p);
    randV3 (part->v, sys->v);
    part->age = sys->mnage + rand()%sys->mxage;
}
```

```
static void ageAndDrawParticles(const psystem *sys)
{
// allows fewer particles in system... 
uint i=sys->np;
particle *lp;
  do {
    lp=&(sys->parts[i]);
    // tried rewriting this many times...no shorter
    addV3 ( lp->p, lp->v, sys->g );
    // if particle exists...age and draw  or, if not, create it...
     (lp->age--) ?  sys->drawfp(lp) :  setParticle(sys, lp) ;
  } while (--i);
}
```
