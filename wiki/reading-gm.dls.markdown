---
title: "Reading dm.dls"
layout: "wiki-page"
---

So, reading sounds from gm.dls is easy, right? You take the data offset, you SetFilePointer(), and you ReadFile() the samples. But first you have to open the file. The normal approach would be:

```
HANDLE h = CreateFile( "c:/windows/system32/drivers/gm.dls", GENERIC_READ, FILE_SHARE_READ, 0, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, 0);
```

Bad, I have windows installed in my D: drive. So, second attempt is:

```
char path[256];
int pos = GetSystemDirectory( path, 256 );
for( i=0; i<16; i++ ) path[pos++] = "/drivers/gm.dls"[i];
HANDLE h = CreateFile( path, GENERIC_READ, FILE_SHARE_READ, 0, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, 0);
```

This is better, but we added A LOT of bytes to our intro, what a pity. And the worse is yet to come. It will not work in Windows XP 64 bit (and Vista I think), because in that OS the gm.dls file is located at "drivers/etc/" folder. So, the most compatible code so far seems to be:

```
static char paths[] = "\\drivers\x0"
                      "\\drivers\\etc";
static char fileName[] = "\\gm.dls";
```

```
char *pathsPtr = paths;
do {
    int pos = GetSystemDirectory( path, 256 );
    while( *pathsPtr ) path[pos++] = *pathsPtr++; pathsPtr++;
    for( i=0; i<sizeof(fileName); i++ ) path[pos++] = fileName[i];
    h = CreateFile( path, GENERIC_READ, FILE_SHARE_READ, NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL );
}while( h==INVALID_HANDLE_VALUE );
```

But now this is really expensive if you compare it to the first version. Now, I found a trick that it might be well know to others, I don't know, but I still want to share it just in case. It all based on the obsolete Win32 API function OpenFile(), that according to MSDN should not be used other than in Windows 3.1 (16 bit). However it works perfectly fine in all 32 bit versions of Windows, as well as on Windows XP 64 and Vista 64 bit as long as you compile your exe as 32 bit image.

OpenFile() seems to return a HFILE instead of a HANDLE. No problem, you can cast it :) The advantage of this function is that it searches system folders, and this avoid using the GetSystemDirectory() function (and the import) as well as the first manual string concatenation. Because I usually work on XP 64 machines, I want to keep compatibility with it too, so the final code looks like:

```
static char *paths[2] = { "drivers/gm.dls", "drivers/etc/gm.dls" };
```

```
HANDLE h;
for( h=INVALID_HANDLE_VALUE, int i=0; h==INVALID_HANDLE_VALUE; i++ )
    h = (HANDLE)OpenFile( paths[i], (OFSTRUCT*)sampleBuffer, OF_READ );
```

This is much smaller than the previous compatible version, something like 40 bytes if I remember well, and still as compatible (2K, XP32, XP64 and Vista tested). Also note I used right slashes instead of backslashes for the paths, it seems to compress better in some situations. Second note, I usually close my files, release the rendering context and destroy my windows, but if you're lacking the space you might skip the close the file handle step of course.
