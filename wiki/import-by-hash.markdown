---
title: "Import by hash"
layout: "wiki-page"
---

Importing by hashes has become quite popular those times, since importing by ordinals is too unstable accross different configurations and the standard importing technique PE files use is pretty heavy loaded.

I started writing this article for a possible size optimisation in respect to [Aulds OGL Framework](aulds-ogl-framework). For this reason the given implementation will not be asm for now.

## How imports work

The default way imports work, is that your executable has some extra data containing a section with strings of which librarys it uses and the names of the used function plus some additional overhead for describing the section.

The PE loader will, when loading your executable, load and map all dll's specified in this section and then assigns the destination to external calls. If you want more details on how this work, do some search for "import address table" and "relocation"

However there is another way of mapping dll's into your process memory and retrieve function entries. The functions needes for this are:

```
 LoadLibraryA
 GetProcAddress
```

We will only need the mapping function later on. But for now lets imagine you only use those 2 calls to retrieve all your imports. What will your executable look alike after?

Well its pretty much the same as before, just that you shiftet out the import and library names out of the import section to the data section. So no gain at all, since you could not get rid of any strings or the import section at all!

## Mapped dll's

Ok so far we only took a look at our imports. So how about looking at the others exports instead? For this we need to understand, how a mapped dll is structured and how to get its virtual address(es).

The last of those two problems is solved pretty fast: LoadLibraryA returns what is calles HMODULE or a module handle in Windows. However it is actually a Pointer (virtual address) to a place in your processes memory, where the the dll got mapped (this is also called the librarys base address)

From here on things go pretty straightforward, because the mapped dll is pretty similar to the dll on your disk. The only main difference is, that the sections (code data exports etc) got loaded to different addresses, than they resided in the exe.

## RVA's PE-Headers and co.

Before we can continue, we first need a short definition: Addresses inside the PE structure are stored as so called RVA values. This stands for "Relative Virtual Adress" and simply means that the address is relative to the dll base (see above).

If you want to calculate the VA (Virtual Address) from a given RVA, just add your HMODULE to it.

*   VA = RVA + HMODULE

Now we are prepared to explore the PE. The header is defined as followed (this and all upcoming definitions are taken out of winnt.h)

```
typedef struct _IMAGE_DOS_HEADER {      // DOS .EXE header
    WORD   e_magic;                     // Magic number                       0-2
    WORD   e_cblp;                      // Bytes on last page of file         2-4
    WORD   e_cp;                        // Pages in file                      4-6
    WORD   e_crlc;                      // Relocations                        6-8
    WORD   e_cparhdr;                   // Size of header in paragraphs       8-10
    WORD   e_minalloc;                  // Minimum extra paragraphs needed    10-12
    WORD   e_maxalloc;                  // Maximum extra paragraphs needed    12-14
    WORD   e_ss;                        // Initial (relative) SS value        14-16
    WORD   e_sp;                        // Initial SP value                   16-18
    WORD   e_csum;                      // Checksum                           18-20
    WORD   e_ip;                        // Initial IP value                   20-22
    WORD   e_cs;                        // Initial (relative) CS value        22-24
    WORD   e_lfarlc;                    // File address of relocation table   24-26
    WORD   e_ovno;                      // Overlay number                     26-28
    WORD   e_res[4];                    // Reserved words                     28-36
    WORD   e_oemid;                     // OEM identifier (for e_oeminfo)     36-38
    WORD   e_oeminfo;                   // OEM information; e_oemid specific  38-40
    WORD   e_res2[10];                  // Reserved words                     40-60
    LONG   e_lfanew;                    // Address of new exe header          60-64
} IMAGE_DOS_HEADER, *PIMAGE_DOS_HEADER;
```

Every 32bit PE (exe or dll) under Windows has a header like this. It is actually only a DOS header which is not used at all by Windows. The last field e_lfanew is a RVA pointing to the header, we are interested in:

*   NT_HEADER_RVA = *(long*)(HMODULE + 0x3C)
*   NT_HEADER_VA = HMODULE + NT_HEADER_RVA

```
typedef struct _IMAGE_NT_HEADERS {
    DWORD Signature;                                 // 0-4
    IMAGE_FILE_HEADER FileHeader;
    IMAGE_OPTIONAL_HEADER32 OptionalHeader;
} IMAGE_NT_HEADERS32, *PIMAGE_NT_HEADERS32;
```

with

```
typedef struct _IMAGE_FILE_HEADER {
    WORD    Machine;                                 // 4-6
    WORD    NumberOfSections;                        // 6-8
    DWORD   TimeDateStamp;                           // 8-12
    DWORD   PointerToSymbolTable;                    // 12-16
    DWORD   NumberOfSymbols;                         // 16-20
    WORD    SizeOfOptionalHeader;                    // 20-22
    WORD    Characteristics;                         // 22-24
} IMAGE_FILE_HEADER, *PIMAGE_FILE_HEADER;
```

```
typedef struct _IMAGE_OPTIONAL_HEADER {
    //
    // Standard fields.
    //

    WORD    Magic;                                   // 24-26
    BYTE    MajorLinkerVersion;                      // 26-27
    BYTE    MinorLinkerVersion;                      // 27-28
    DWORD   SizeOfCode;                              // 28-32
    DWORD   SizeOfInitializedData;                   // 32-36
    DWORD   SizeOfUninitializedData;                 // 36-40
    DWORD   AddressOfEntryPoint;                     // 40-44
    DWORD   BaseOfCode;                              // 44-48
    DWORD   BaseOfData;                              // 48-52

    //
    // NT additional fields.
    //

    DWORD   ImageBase;                               // 52-56
    DWORD   SectionAlignment;                        // 56-60
    DWORD   FileAlignment;                           // 60-64
    WORD    MajorOperatingSystemVersion;             // 64-66
    WORD    MinorOperatingSystemVersion;             // 66-68
    WORD    MajorImageVersion;                       // 68-70
    WORD    MinorImageVersion;                       // 70-72
    WORD    MajorSubsystemVersion;                   // 72-74
    WORD    MinorSubsystemVersion;                   // 74-76
    DWORD   Win32VersionValue;                       // 76-80
    DWORD   SizeOfImage;                             // 80-84
    DWORD   SizeOfHeaders;                           // 84-88
    DWORD   CheckSum;                                // 88-92
    WORD    Subsystem;                               // 92-94
    WORD    DllCharacteristics;                      // 94-96
    DWORD   SizeOfStackReserve;                      // 96-100
    DWORD   SizeOfStackCommit;                       // 100-104
    DWORD   SizeOfHeapReserve;                       // 104-108
    DWORD   SizeOfHeapCommit;                        // 108-112
    DWORD   LoaderFlags;                             // 112-116
    DWORD   NumberOfRvaAndSizes;                     // 116-120
    IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES];
} IMAGE_OPTIONAL_HEADER32, *PIMAGE_OPTIONAL_HEADER32;
```

So far nothing, we could use, but a look at the DataDirectory reveals:

```
typedef struct _IMAGE_DATA_DIRECTORY {
    DWORD   VirtualAddress;
    DWORD   Size;
} IMAGE_DATA_DIRECTORY, *PIMAGE_DATA_DIRECTORY;
```

Those entries give us the RVA to the different sections the dll has. And the first one of those will be the export table we are looking for.

*   EXPORT_HEADER_RVA = *(long*)(NT_HEADER_VA + 0x78)
*   EXPORT_HEADER_VA = HMODULE + EXPORT_HEADER_RVA

Now we are pointing to the export section header, which has the following structure:

```
typedef struct _IMAGE_EXPORT_DIRECTORY {
    DWORD   Characteristics;        // 0-4
    DWORD   TimeDateStamp;          // 4-8
    WORD    MajorVersion;           // 8-10
    WORD    MinorVersion;           // 10-12
    DWORD   Name;                   // 12-16
    DWORD   Base;                   // 16-20
    DWORD   NumberOfFunctions;      // 20-24
    DWORD   NumberOfNames;          // 24-28
    DWORD   AddressOfFunctions;     // 28-32  RVA from base of image
    DWORD   AddressOfNames;         // 32-36  RVA from base of image
    DWORD   AddressOfNameOrdinals;  // 36-40  RVA from base of image
} IMAGE_EXPORT_DIRECTORY, *PIMAGE_EXPORT_DIRECTORY;
```

Well lucky us! This is exactly what we would need. RVA's to a list of strings and the import addresses! To sum all our addresses up:

```
NT_HEADER_RVA     =  *(long*)(HMODULE + 0x3C)
NT_HEADER_VA      =  HMODULE + NT_HEADER_RVA

EXPORT_HEADER_RVA =  *(long*)(NT_HEADER_VA + 0x78)
EXPORT_HEADER_VA  =  HMODULE + EXPORT_HEADER_RVA
```

```
NUM_FUNCS        =  *(long*)(EXPORT_HEADER_VA + 0x18)
```

```
FUNCTIONS_RVA     =  *(long*)(EXPORT_HEADER_VA + 0x1C)
FUNCTIONS_VA      =  HMODULE + FUNCTIONS_RVA
```

```
FUNCNAMES_RVA     =  *(long*)(EXPORT_HEADER_VA + 0x20)
FUNCNAMES_VA      =  HMODULE + FUNCNAMES_RVA
```

```
FUNCORDINALS_RVA  =  *(long*)(EXPORT_HEADER_VA + 0x24)
FUNCORDINALS_VA   =  HMODULE + FUNCTIONS_RVA
```

## Hashing over the function names and resolving exports

We actually already gathered all addresses we need. FUNCNAMES_VA points to list of RVA's to null terminated strings of the export function names.

```
for (int i = 0; i < NUM_FUNCS; i++)
{
   char *importName = (char*)(((long*)FUNCNAMES_VA)[i] + HMODULE);
   for (int j= ...)
   if (DoHash(importName) == MyHashes[j])
     MyImport[j] = ResolveImport(i);
}
```

Wow now we found the index of the string of the function we want, but how to come from this index to function address? Unfortunally it is not an index for the function list, but for the function ordinals list, which contains the function list index. Also note that the ordinal list is a 16bit integer list and the function pointers are RVA values!

```
void* ResolveImport(int StringIndex)
{
   unsigned short ordinal = ((unsigned short*)(FUNCORDINALS_VA))[StringIndex];
   return (void*)(((long*)FUNCTIONS_VA)[ordinal] + HMODULE);
}
```

## intermediate result

With only LoadLibraryA and the routines described above we can implement an import by hash loader, which could resolve all functions by hashing the export list of the loaded library we know the HMODULE of. In practise 4 byte for the hases should be passably for avoiding collisions with other import names. Thus we only need 4 bytes per imported function instead of saving the whole name!.

However we still have the problem of the remaining LoadLibraryA which we need to map dll's to our processes memory and get the HMODULE from them. Well we could resolve LoadLibraryA from Kernel32.dll with the same method but oops where to get the module handle from then? We cant call LoadLibraryA("Kernel32") to get it for resolving LoadLibraryA afterwards with... damn!

If we just could, the whole import section could be dropped. Well but as always there is a solution. Actually there are even more ways to retrieve the kernel32.dll HMODULE from within the main without any API calls.

If you are interested in those methods look over to the hacker scene with their exploit codes (they basically start at some entry without knowing anything like we do) The most reliable and small method so far however is the PEB method. I wont describe it here. Let me know when you want to know more about those stuff and I might write something about it.

## A C++ implementation

(ok if you put all local variables to function scope its actually C, but I I'm too lazy for and like using temporarily loop variables.)

Notes:

*   PEB is used for resolving Kernel32 HMODULE (sorry but 1 line of inline asm is needed for that, but its worth it since it lets drop you the whole import section!)
*   RVA's are resolved to VA's in a very ugly way because the compiler seems to make much better code this way than using temporary variables.
*   The functionpointertable where imports get loaded to is a local variable and thus ebp indexed resulting into only 3 bytes for calls as long it is not too big. For this its really important to make sure the compiler does _NOT_ ommit the stackframe!
*   The code is really ugly sorry :)
*   You can and should only use API calls from within your main(). Passing the functionpointer table to another function will annul the advantage you gained by ebp indicing in the main()

```
// no need for any includes and linkings against libs at all, 
// since we will do it the hardcore way

#define STDCALL __stdcall
#define VK_ESCAPE 27
typedef unsigned long DWORD;

// Define some function pointers for 1,2, ...5 args
typedef DWORD (STDCALL *funcPtr0)(void);
typedef DWORD (STDCALL *funcPtr1)(DWORD);
typedef DWORD (STDCALL *funcPtr2)(DWORD, DWORD);
typedef DWORD (STDCALL *funcPtr3)(DWORD, DWORD, DWORD);
typedef DWORD (STDCALL *funcPtr4)(DWORD, DWORD, DWORD, DWORD);
typedef DWORD (STDCALL *funcPtr5)(DWORD, DWORD, DWORD, DWORD, DWORD);

// Here the function name hashes go
unsigned int hashTable[] = {
	0xE9826FC6,   // LoadLibraryA
	0x38A66AE8,   // ExitProcess
	0xDE59F860,   // GetAsyncKeyState
	              // add your hashes here
	0 // zero terminate (cheaper than storing size)
};

// add your functions here in the order the hashtable has here
#define LoadLibraryA     ((funcPtr1)functionTable[0])
#define ExitProcess                (functionTable[1])
#define GetAsyncKeyState ((funcPtr1)functionTable[2])

```

```
void ResolveHashes(unsigned long module, funcPtr0* functionTable)
{
	int i = (unsigned char)((i^i)-1);
	long tmp = *(long*)(*(long*)(module + 0x3C) + module + 0x78) + module;
	int numImp = *(long*)(tmp + 0x18); // number of imports
	while (--numImp >= 0)
	{
		// resolve import #numImp
		unsigned char *importName = (unsigned char*) (*(long*)(*(long*)(tmp + 0x20) + module + numImp * 4) + module);

		// now hash the import name and check it against a stored hash value

		// code taken from [http://www.20to4.net/dat/importbyhash_v3.cpp](http://www.20to4.net/dat/importbyhash_v3.cpp "http://www.20to4.net/dat/importbyhash v3.cpp")
		// so dont blame me for this asm passage, use your own hashing if you want to :)
		// I however, was too lazy to use another hashfunc and recalc all my hashes...
		unsigned int hash;
		__asm
		{
			pushad
			mov esi, importName
			xor ebx, ebx
			lodsb
		calcloop:
			xor bl, al
			rol ebx, 6
			lodsb
			test al, al
			jnz calcloop
			mov hash, ebx
			popad
		}

		// now lookup hash
		int j = (unsigned char)((j^j));
		while (hashTable[j])
		{
			if (hash == hashTable[j])
			{
				// found the hash now resolve function entry
#define IMPORT_INDEX *(unsigned short*) (*(long*)((tmp + 0x24)) + module + numImp * 2)
				functionTable[j] = (funcPtr0)(*(long*)(*(long*)(tmp + 0x1C) + module + 4 * IMPORT_INDEX) + module);
			}
			j++;
		}
	}
}
```

```
// this is important! dont let the compiler optimize stackframes
// for the main because we _WANT_ ebp addressing (smaller!!!)
#pragma optimize("y",off)

// need stdcall here to generate appropriate stackframe
void STDCALL mainCRTStartup()
{
	// use locally (ebp indexed) function table
	funcPtr0 functionTable[30]; // here your actual functions are stored
	                            // note that there is a maximum of about 30
	                            // if you use more only around 30 get translated
	                            // to small calls, while you pay alot to the rest!
	unsigned int module;		// temporarily needed for calculating kernel32

	// please forgive me this, but its really absolutly necessary
	// if we want to detect kernel32 via PEB
	// other methods like topstack could might hacked via plain c code
	// however those methods arent as small and not guaranted to run!
	// topstack especially does not run on XP64!

	__asm
	{
		mov eax, DWORD ptr fs:[30h]
		mov module, eax
	}
	// to check out why this works, search for "using PEB for retrieving kernel32"
	module = *(long*)(*(long*)(*(long*)(*(long*)(module + 0x0C) + 0x1C)) + 0x8);

	// resolve everything from kernel32
	ResolveHashes(module, functionTable);
	// and now since we have LoadLibraryA everything from other dlls...
	ResolveHashes(LoadLibraryA((DWORD)"user32"), functionTable);
	// ... load opengl32 etc ...

	// your initcode goes here

	do
	{
		// your rendercode goes here
	} while (!GetAsyncKeyState(VK_ESCAPE));
}
```

## Some final Words

Using the shown frame, you can import dll functions by only needing to save 4 bytes per imported function and the dll names. The frame is far from perfect, since it's implemented in C and not in assembler, which gives you much better control at many parts. For those who are familiar to assembler it should be no problem to write down an implementation from scratch. It's even easier than this hacked C++ frame.

If for some reason you also want to see my asm implementation, let me know and I'll add it.

In this article might be lots of minor errors (probably mostly typos and bad grammar sorry for that)... oh well and the layout is not really that great. Feel free to modify.
