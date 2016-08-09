---
title: "Crinkler"
layout: "wiki-page"
---

[Crinkler](http://crinkler.net) is the current best-in-class tool for generically squeezing bytes from 1kb and 4kb win32 intros. The tool is without source code, which inhibits future research and development.

Sadly, the only option for investigation upon improving the techniques is to re-implement the excellent work that the authors have done on Crinkler. Once a freely licensed equivalent _with source_ exists, future research on improvement can be done there. Sensible licenses to avoid the situation reoccurring, would be:

*   GPL for the tool
*   BSD for the embedded decompression code

Everyone can use and improve the tool; everyone can still produce binary demos without revealing their personal source code. Crinkler has been out for a couple of years ago; the present authors could be approached with the re-licensing suggestion above—an "Open Crinkler" will gain even more techniques and optimisations.

## Techniques

Some information can be gleaned by reading:

*   [http://ftp.jp.openbsd.org/pub/scene/parties/2005/assembly05/seminars/crinkler-compression.ppt](http://ftp.jp.openbsd.org/pub/scene/parties/2005/assembly05/seminars/crinkler-compression.ppt "http://ftp.jp.openbsd.org/pub/scene/parties/2005/assembly05/seminars/crinkler-compression.ppt")

The following are a list of the techniques I understood to be used by the code. The order top to bottom during creation (by the tool) and bottom to top during the decoding stage of the final executable.

### Block ordering

Compression works best when it able to associate pieces similar pieces of data already close to each other. You can either do this ordering yourself, trying to put all your code in one place, all your images in another, floating point values in a third place...

Or you can leave it in tiny pieces and let a computer try every combination. Each unlinked object file normally contains two pieces (.text and .data), the more unlinked separate object files you can provide, the more ordering combinations can be tried. By performing the re-ordering before the linking stage, the information about the reordering remains is transfered to the order of the code and data itself.

Given N object files, the number of combinations is (2N)!⇒3.6million combinations for just 5 object files. In reality if all the code is going to end up in one place and all the data in another, the ordering possibilities are actually a much more management 2(N!)⇒240 combinations.

Block reordering is a case of the Asymmetric Travelling Salesman Problem. Each block (section from a object file) must be inserted exactly once, with the challenge to maximise the similarity between consecutive (and previous) blocks.

One of the suggestions made by the Crinkler authors is to set sections so that they request to be 256-byte aligned; this should lead to the first 24-bits being the same for all data lookups within a single function or block.

### Jump/call transformation

In x86 code, call and jump are relative to the current instruction pointer. To reach the same destination from different locations, the ±offset will be different _from_ each of those locations.

<table border="1">

<tbody>

<tr>

<td>1 byte</td>

<td>1 byte</td>

<td>1 byte</td>

<td>1 byte</td>

<td>1 byte</td>

</tr>

<tr align="right">

<td>opcode</td>

<td colspan="4">±relative offset to</td>

</tr>

<tr align="right">

<td>0xe8</td>

<td colspan="4">0x00000042</td>

</tr>

</tbody>

</table>

Jump transformation

<table border="1">

<tbody>

<tr>

<td>1 byte</td>

<td>1 byte</td>

<td>1 byte</td>

<td>1 byte</td>

<td>1 byte</td>

</tr>

<tr align="right">

<td>opcode</td>

<td colspan="4">absolute offset</td>

</tr>

<tr align="right">

<td>0xe8</td>

<td colspan="4">0x00013c40</td>

</tr>

</tbody>

</table>

Although this new destination looks harder to compression, if it appears more than once, the duplicates can be matched up.

The 0xe8 remains in the code, allowing a fully reversible transformation. A dumb approach can be taken to values of 0xe8 that appear elsewhere in the code segment; just replace all of them—it is unlikely to make the code significantly worse.

### PPM(8) decoder

*   [http://en.wikipedia.org/wiki/Prediction_by_partial_matching](http://en.wikipedia.org/wiki/Prediction_by_partial_matching "http://en.wikipedia.org/wiki/Prediction by partial matching")

Prediction by Partial Matching estimates the most likely value of the next byte by producing a predicted based on likelihood of previously observed sequences. In English text, the letter 'Q' is followed by 'U' with high probability. After a couple of instances, this probability is learnt.

Crinkler uses a prediction based on up to the last 8 bytes; a set of parameters are used to vary the prediction model for the current data type. The parameters is a bit mask of which bytes to take into account when making the prediction. Predictions are ranked in order and used in a similar where to a Move To Front (MTF) transformation.

PPM needs a model that best suits the current data. A global rule that we tend to (implicitly) teach to model is that data items start on a 8-bit boundary. A truely generic model would not even have this rule.

*   Input data tends to be stored on 8-bit boundaries.

For specific data, the most likely modelling would be 24-bit, 32-bit or 64-bit boundaries.

### Dynamic loader/linker

Static binaries contain all the code necessary in one large execution file; linking OpenGL into your program would that it is no longer a ≤4kB intro (or ≤4MB...).

When you would like to use a function from another library, you need to do one of the following.

#### Leave a request

Requesting a function in advance means storing the name of library (.so/.dll) and the name of each function (printf, glVertex3f). The list of requested functions will go in the _symbol table_ at the end of your program, the dynamic loader will take care of finding the library, patching everything up and reporting if your executable cannot be run as there are dependencies on libraries that cannot be located.

#### Ask on demand

The next open is to skip the symbol table, but to keep a list of the functions you are going to need. Each function can be requested on demand using the dynamic loader library. In the Unix world this is dlopen() and dlsym(). For this to work, you must store each function name as a full string.

On Win32 symbols can be loaded by number (by ordinal) instead of by name. In this case you ask for the "Nth" function in a library. If the design of the library is changed, the Nth function will change. Loading by ordinal is not sensible if you would like your executable to run on more than one machine.

On a Linux machine, the entries for dlopen() and dlsym() will be included in the normal symbol table to avoid a chicken and egg situation.

#### Do it yourself

Doing a partial do-it-yourself (DIY) job can save you, just like the realworld. The symbol table can be dropped completely and you are completely free to choose your own method for matching up symbols.

##### By ordinal

When a computer is finding and loading a library by hand, locating the Nth symbol in the library is the easiest. That doesn't mean it is sensible and will find your executable can only work on some machines (the one it was created on).

Loading by ordinal should be avoided, there are better ways (see below), assuming you have the bytes to spare.

##### By string

The easiest for a human is finding by name.

On Unix, the built-in system libraries all include hash of the names of the functions inside them in order to speed up linking.

##### By substring

By name, but only an approximate (and hopefully unique) port of the name.

##### By hash

Comparing strings is a slow operation, the technique to make it faster is to compute hashes (numerical approximations) for each string, comparing all the result and then doing a string match to make doubly sure.

We could always precompute the hash approximations, and skip the creation part. Faster!

When making a 4kB demo, the concentration is on size and not speed. One option is to skip the final string match stage and assume we will always get lucky—and ignore the final string comparison check.

Because libraries may already contain a hash for functions, the code needed to do custom hash calculation can be skipped over and just a comparison made against hashes already listed in the library.

Another option for find-by-hash is to match against the hashes already in the library, get the string name of the function and pass the name to `dlsym()` as before.

### PE header

*   Negative starting offset
*   Borrow unused fields
*   Make use of blank space

## Ideas for improvement

### Remove limitations

Crinkler is designed for 4 kB executables; but there is no reason (other than the time constraints of block ordering) why the code cannot be fixed to remove the limitations. Any program is potentially compressible using the techniques presented.

### Lossy compression

Lossy compression is a dangerous area, but one which we know reaps rewards (JPEG, MP3, DIVX). Data is actively throw away in an effort to boost self similarity and resultant compression. (Lossly compression doesn't tend to work for code, except by manually removing a whole effect!).

When applied to data, lossy storage would be a way to _always_ ensure your program fitted, even if it didn't look quite as your expected.

*   Truncation of floating point values (drop least significant bits)
*   Truncation of integer data (drop least significant bits)

If data is genuinely data and does not have any affect on program flow control (the program handles correctly handles divide-by-zero), then suitable sections of data _could_ be lossily compressed. The user-interface would probably be a slider much JPEG compression.

Lossy compression probably has more benefit for 64 kB demos; 4 kB tends to already store most information pragmatically as data storage is not normally a space efficient option.
