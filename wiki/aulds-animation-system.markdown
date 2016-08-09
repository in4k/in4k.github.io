---
title: "Auld's Animation System"
layout: "wiki-page"
---

# Whats an animation system?

Its the part of your demo nobody notices. The cogs. It controls which graphics effect are drawn and when. It may help to sycnhronise sound effects or changes in musical tempo. Because nobody notices it, it doesn't add to your demo rating so really, if you could, you'd like to use zero bytes: its just overhead thats all. You would also like it to be easy to edit and maintain. Anyone who has tried knows that a long sequence of if, then, else is just a headache.

In 4k constant overhead like this is a killer. So lets analyse the problem, pack some data and write the tightest of tight controllers.

Definitions:

* Demo Scene: something drawn on the screen for a specific period of time
* Demo Timeline: the time ordered sequence of scenes for the demo

Assumptions:

+ Maximum runtime of 4k demos is about 3 minutes or 180 seconds.
+ Graphical effects remain on the screen for at least 1 second. OK arguably we might want finer granularity, but for now lets assume it.
+ Scenes do not overlap. We'll come back to this later.
+ Scenes draw something and return quickly and are re-entrant safe we can call a scene many times

Therefore, with these assumptions, our timeline is a simple array of times e.g.

```
const unsigned int scenes[] ={0,3,15,17,65,104,255};
```

This defines 6 "scenes" running from (0s->3s), (3s->15s), (15s->17s), (17s->65s), (65s->104s), and an end program routine (104s->255s).

As the maximum value that is expected in the array is 180 seconds (3 minute barrier) we could use unsigned bytes. However we chose to use ints instead as shorts produce longer code. 255 can be used to signal exiting the demo. How do we call the right scene?

Define each scene to be a function: static void sceneName () {}

Then the timeline could also include an array of function pointers thus:

```
const static void (*scenesfp[])() = {
sceneName1, sceneName2, sceneName3, sceneName4, sceneName5, exitName
};
```

Lastly code to control our timeline, we do this

```
int i=-1;
while (timeinseconds>scene[  i] {};
scenesfp[i](); 
```

Thats a fantastically tight routine: No switches, no case, no ugly if,then,else. Note that is technically a cummulative probablity distribution function. As such it could be wrapped up in a function and used to determine, say, which matrix to apply in an IFS, or whether to accept a random ray in a stochastically sampled BRDF or...ok I digress. The point is it can be used for more than just animation choices.

Getting time in seconds can be a slight cheat. Suppose we have the number of milliseconds since the demo began , we can do this:

``` 
timeinseconds = time >> 10;
```

Its not exactly the time in seconds but its only 0.024% out. Who cares? This is a 4k demo.

Note: Using this method, our demo ends after 104 seconds as defined in the array. Its weird that we define "ending" to run from 104 to 255 seconds but so long as we exit the program (or cause it to be exited) in exitName(), it will only be called once at around 104 seconds.

In exitName I put

```
m.wParam = VK_ESCAPE;
```

which forces the program to exit wherever I check for the escape key.

# Extending to cover multiple scenes at the same time

The scene array above would become an array of triples (start time, end time, index to function pointer array). Our control software would become:

```
// Hmm be careful fo these 2d arrays...they are inefficent...
for (i=0; i<=NUMSCENES; i  ) {
  if ( BETWEEN (scenes[i][0], scenes[i][1], timeinseconds) ) scenesfp [ scenes[i][2] ]();
}
```

Be careful not to overlap the exitScene with anything.
I leave the BETWEEN macro to your imagination :-)

# Transitions

A word on transitions. A transition is typically thought of as something that happens between scenes. Examples include fades, wipes, spins etc. In the methodology described above, a transition is just another scene.

What does a scene look like?

We stated that scenes draw something and then return quickly. Therefore scenes here may be called many times. Take sceneName2 which runs at time (3->15). It may be called many time in the period 3-15 seconds. Scenes should therefore be written to respond to time and draw their content dependent on time. There are dozens of ways to do this, leading to many neat tricks. However, the simplest is to capture the current time when the scene is first called and use this as a subtractor from the current time each time the scene is called.

However, scene design is left to you.

# Extensions

Question: can you see how to write a demo that modifies its own sequence each time it runs (remember function pointers are just variables)?

Question: Can you see how to extend the code to run the timeline backwards as well as forwards? Then, from one arbitrary time to another?

Question: How to slow everything down or speed it up?

If you've understood the article and can answer those questions, you know you have a an animation system with the flexibility to do things never seen in 4k demos. Oh yes and its about 100 bytes (depending how many scenes you have).

-- auld
