---
title: "Auld's Texture Generator"
layout: "wiki-page"
---

This article describes texture generation systems in broad terms. You can skip this if you know how they work already of course. The article will then discuss some cheats that can be used in 4k aimed specifically at reducing the storage size of the textures and reducing the code size of texture evaluators. Some C code is presented that evaluates textures in less than 200 bytes of compressed code and finally limitations of the code are described.

The article presents a texture evaluator that compresses to less than 200 bytes, written in C, containing 8 basic texture functions. Textures descriptions can be stored in as little as two integers and each texture descriptor may produce up to four useful textures. **The average storage for textures after compression can therefore approach 1 byte!**

### Texture Generators

Texture generation can be done in two ways: on the fly or pre-processed. Pre-processing is not appropriate for 4k as we cannot store the textures and read them later. This eliminates many very impressive texture generation tools. Even those created for 64k demos often pre-process and output an image.

A better approach for 4k is to split the process into two parts: a creator and an evaluator. The creator allows the user to explore textures and then stores them in just a few bytes by describing _how to create the texture, not by compressing the texture itself_. The evaluator reads how to create the texture and does so on the fly, writing it to a traditional texture image. The description may be just a few bytes.

The evaluator creates a texture image by evaluating each pixel in the texture image in a mathematical equation or set of maths equations. Imagine it a a big nested for loop:

```
for each y pixel
 for each x pixel
    evaluate (x,y, texturedescription)
 end
end
```

### Functions

The creator and evaluator have in common two things. The first is a set of simple functions. These functions are often very simple and not interesting by themselves. They can however be very complex (eg Perlin Noise). The texture description describes which functions and in what order they are applied to the input (x,y). The output from each function is passed to the input to the next. Some example functions might be:

```
(x*y) (x+y) max(x,y) min(x,y) (x&y) (sin(x)*cos(y))
```

### Structure

The functions in the evaluator must be connected in some way. This is typically a tree structure:

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

Obviously this gets complicated quickly and incredible textures can result. The f numbers refer to the set of functions described above.

### Evaluation v Creation

Only the evaluator is used in the final program. The creator can be as complex or as simple as possible. Its good to understand that the set of possible functions and structures combined is enormous and many of the resulting textures are uninteresting. For this reason the creator is often very very powerful and may rely on techniques like Genetic Algorithms to search the space of possible interesting textures.

This is fine and not the subject of this article. **Reducing both the texture descriptor and the evaluator to the smallest possible number of bytes, whilst representing a useful structure and a useful set of base functions is the key to 4k.**

### Colour

One complexity here is colour. The result of the texture evaluator described above is a single value. Yet colour is represented (typically) using three values. There are several possible solutions:

*   Extract the colour from the single value
*   Evaluate three such trees, one for each colour
*   Make each function take 3 values (and the root function is therefore just the colour)

Each has pros and cons.

## Fitting Texture Generation in 4k

In this section there are a number of ideas of how to reduce code size so that texture generation is suitable for 4k demos.

### Fundamental Ideas

Firstly we must make the functions as simple as possible. Secondly, we must simplify the structure they are in, making the evaluator trivial in code. Lastly, we must reduce the storage needed for textures.

### The Function Range: avoiding clamps and casts

In the description above there are functions which take two parameters, x and y, and returns one parameter. Its clear the returned parameter must be the same type as the input one.

For 4k the function range is largely decided by the functions we chose to use. **We cannot afford to clamp input values to force them into the correct domain, or worse, cast them**, as we might without memory constraints.

Two possible solutions present themselves. The first is to use functions from a maths library like sin, cos, tan, exp etc. This would be a double as input (or float) and the same as output. Unfortunately, the range and domain of say, sin, are very different. This isn't per se a problem but will mean that values in textures tend to collapse to a limited range of values quickly and far less of the created textures will be useful.

A second solution is to use integers as input and output, using intrinsic operations and write our own functions. I tried both. I tried combining them also. In the end I am happy with the integer ones.

> By limiting all functions to the same type of input and output, we avoid casting.

> By limiting all functions to have the same domain and range implicitly we are able to avoid scaling or clamping.

In practice we break rule two a little but must always bear it in mind when chosing to add a function or not. Multiply, for example, breaks the rule but is so useful, we add it.

### The Structure

Theory says that a tree should be used. An original paper on the subject by Karl Sims very quickly dismissed arrays in favour of more complex structures. However, Karl was not writing 4k demos. The code is greatly simplified by having arrays, not trees. A typical structure is therefore:

```
  f6 - f1 - f6 - f2 - f1 - f1
```

This trivialises the evaluator code and the texture descriptor. However Karl was right, this is too simple. During development I experimented with both special complex leaf nodes and with tree structures like:

```
    f2   f2   f4   f6   f3   f3
   /    /    /    /    /    /
  f1 - f6 - f4 - f4 - f1 - f2
```

However, complex functions need lots of bytes to write them in C (eg Perlin Noise) and the second tree structure wasn't effective enough in varying textures to warrant the added complexity in the evaluator code.

So in summary, the final structure is a simple array. How, then, to add some more interesting variation?

### Putting Extra Values into the Tree

A constant is added which can vary hugely from texture to texture. This constant is used within texture functions. Thus the exact same structure and functions can result is large variations in output texture just by changing the single integer constant. An examples of such functions are:

```
(v * GACONST)  (v - GACONST)  (v | GACONST) 
```

To keep the texture descriptor small, the integer is defined once per texture and used globally in every function that accesses it.

### Storing The Texture Descriptor

Storage of a texture descriptor is now simply a byte per function in the array of functions. In addition we need one value for a random seed and one value for the constant described above.

However, we can do better. If we have only 16 functions we only need 4 bits to index functions, meaning we can squeeze 8 function indices into a single integer. **The constant can be used both as input to functions and the seed for random values.**

In this way we can compress 8 function indices, a random seed and a constant into just two integer values or 8 bytes. Using Opengl tricks described below, we can represent as many as four textures using this one texture descriptor!

### The Effect on Functions

Using an array of functions where the output of one is fed into the input of the other, we are forcing the types of all functions to be the same. Further, we are forcing each function to have only one input and output.

Thus functions have the following properties:

*   One input
*   One output
*   Inputs and outputs have same type
*   Inputs and Outputs have same range and domain

### 1d arrays not 2d

There is one very important consequence here. **We no longer use 2d inputs of x,y but a single value input representing the index into the image.** Effectively we have reduced 2d to 1d! This is because all fucntions take a single input value. I briefly experimented with extracting x,y from the single valued input. It didn't pay off for me and suprisingly without this , coherent 2d textures can be formed. Thus we save a few bytes and our functions can remain simple.

### OpenGL Type Tricks

Given an array of values, each of which is 4 bytes in length, OpenGL can convert this into a texture in many different ways. The results, just by changing the type that we tell OpenGL the array contains can be vastly different.

1.  GL_RGBA, GL_UNSIGNED_BYTE will extract a colour texture with alpha from the array
2.  GL_LUMINANCE, GL_UNSIGNED_INT these are all greyscale but very different patterns
3.  GL_LUMINANCE, GL_UNSIGNED_BYTE ..
4.  GL_LUMINANCE, GL_FLOAT ..

Therefore in generating one array for our texture we can extract four or more different textures from it. Not all are useful of course but it is suprising how different each one can be and how useful they could be. **Its possible, therefore, to design textures descriptors that combined with this trick of lying about the format of the input data, produce 4 or more useful textures.**

### Extracting Colour

Colour extraction is done by treating the integer output from a function as 4 bytes: RGBA. Its not satisfactory in all cases but produces reasonable results and with the set of functions used in the code can produce subtle through to "coder colours".

## The C code

**If you use this code in Part or Whole, Please give a credit or greets**

In the code below, I use GA to prefix many items. This is because I use a genetic algorithm to generate textures. More on this later if people want to see it.

Please note this code does not compress texture descriptors so well. It uses a byte per function. One could complicate the evalGene function to extract better stored function descriptors but for clarity I do not do it here.

```
#define MAXGADEPTH 4 //set this to what you like
#define MAXGAFUNCS 8 //f1-f8 below
uint GACONST;
uint *tex1=NULL;
```

```
// The functions
static uint f1 (uint v) {return v+GACONST;}
static uint f2 (uint v) {return v+v;}
static uint f3 (uint v) {return v*v;}
static uint f4 (uint v) {return v|GACONST;}
static uint f5 (uint v) {return v&GACONST;}
static uint f6 (uint v) {return v-GACONST;}
static uint f7 (uint v) {return ~v;}
static uint f8 (uint v) {return rand();}
```

```
static uint (*GAfp[MAXGAFUNCS])(uint)={f1,f2,f3,f4,f5,f6,f7,f8};   

//evaluate a single pixel...                               
static uint evalGene(const GLubyte* gtype, uint v)
{
uint i = MAXGADEPTH-1; 
   // evaluate the array one function after another
   do {  v = GAfp[gtype[i]](v);} while (i--);
   return v;
}
```

```
//evaluate an array of pixels...                
static void evalGeneTexture (const GLubyte* gtype, uint **pix)
{   
uint i=65535; // fixed texture size of 256*256
      if (!*pix) *pix = (uint*) calloc (65536, sizeof(uint)); 
      // note no x,y only a single i value
      do { (*pix)[i]= evalGene(gtype,i); } while (i--);    
}
```

## How to Work With the Code

### How to Add Functions

1.  Add one to the MAXGAFUNCS define.
2.  Add your function in the same form as the others (taking and returning a uint)
3.  Add your new function name to the GAFuncs function pointer variable

Done!

Be careful of two things:

*   The range of a function is significant. Do not write a function that greatly reduces the range of the functions as it will make the whole system less useful.
*   Be careful of anything that responds badly to zero, such as divide. Zeroes can occur easily at any time.

### How to Change the Tree Size

Change MAXGADEPTH, thats it. Of course, your gene has to actually have that many entries when its created but thats up to you how you do that.

### How to Write a Creator

Creators can be tiny or huge. Because the search space of textures is so huge, a genetic algorithm would be useful. Mutation could be done by changing one of the functions at random in the gene, or by changing the GACONST. Breeding could be done by swapping two of the functions from two genes. Selection would be done visually.

A tiny creator would repeatedly create random genes and display them. On pressing a key one could be saved as a texture descriptor.

## Limitations of This System

*   The system with these functions produces mechanical looking textures, nothing much natural. Experimenting with noise [from one of my other articles](/index.php?title=Aulds_Noise_Algorithms "Aulds Noise Algorithms") is also possible. I calculate that it adds 160 bytes of compressed code to the texture generation.
*   Colours are extracted in a simple and not always convincing way. Further there is no way to alter the colour once extracted by shifting or re-mapping it.
*   Textures rarely wrap.
*   Textures, due to the use of quickly varying functions like &, | etc can be aliased badly. This can be useful at time though. More of a feature really.
*   Textures are a fixed size.

Not bad though considering the size of the code. In fact its possible to store 25 specific texture descriptors, extract them to 100 textures and store all that in about 100 bytes. Thats 100 designed textures in 100 bytes, not 100 random ones.

## Further Ideas

*   Add noise
*   Use sin, exp, tan etc and convert the whole structure into functions that input a float and output a float
*   Add loops by overloading certain values to represent how many time to loop through the functions from the start of evaluation. This produces fractal like textures of amazing complexity.
*   It might be possible to animate the texture by introducing a time variable into the functions or simply into the array index used for evaluation.
*   Get me a life!
