# HOWTO Build DevIL 1.8.0 with JPG and PNG support

## Prerequisites:
[CMake](https://cmake.org/) installed - to generate the VS solutions.  
[DevIL](https://sourceforge.net/projects/openil/files/DevIL/1.8.0/DevIL-1.8.0.zip/download?use_mirror=altushost-swe) - to build.  
[libpng](http://www.libpng.org/pub/png/libpng.html) - to build.  
[zlib](https://sourceforge.net/projects/libpng/files/zlib/1.2.11/) - to build(click on latest, mine is zlib1211.zip).  
[jpeglib](https://www.ijg.org/) - for a couple includes.  


## DevIL(Generating the solution)

1. Where it says `Where is the source code` add the path to where you unzipped the `archive/DevIL`  
ex: `D:\metin2\dev_branch\current_src\howto_extern\DevIL_1.8.0\DevIL`

2. Create a folder in there called `build`

3. Back to CMake: Where it says `Where to build the binaries` add `the path to this build folder`  
ex: `D:\metin2\dev_branch\current_src\howto_extern\DevIL_1.8.0\DevIL\build`  

4. Click `Configure`  

5. Change `Optional platform for generator` to `Win32` and click `Finish`

6. After it's done with the config, click `Generate`  

7. After it's done generating, click `Open Project`, or just go to your build folder and open `ImageLib.sln`(do it after step 2, if you don't want to keep VS opened for no reason)  


## LibPng and zlib

1. Extract `lpngxxxx.zip`(extract here)  
2. Extract `zlibxxxx.zip`(extract here), and rename it to `zlib`  

3. Go to `lpng/projects/vstudio` and open `vstudio.sln`  
If it prompts you to `retarget the solution`, click `Ok`  

4. Change `configuration` to `Release Library`  
5. Right click on `zlib` -> `Project Only` -> `Build only zlib`  
6. Right click on `pnglibconf` -> `Project Only` -> `Build only pnglibconf`  
7. Right click on `libpng` -> `Project Only` -> `Build only libpng`  

<ins>__*Change to `debug` and do the same if you need it.*__</ins>

8. From lpng copy `png.h`, `pngconf.h` and `pnglibconf.h` to `DevIL/include`

9. Extract `jpegsr9e.zip`(that's my latest version - yours might be different, but it doesn't matter)
Note: You can also get them from your client's `extern/include/jpeg` folder.

10. Copy `jconfig.vc` to `DevIL/include` and rename it to `jconfig.h`  
11. Copy `jpeglib.h` and `jmorecfg.h`  to `DevIL/include` as well  


## Back to DevIL

1. Go to `DevIL/src-IL/include/config.h`  

2. Find `#define IL_NO_JPG 1` and change it to `#undef IL_NO_JPG`  
3. Find `#define IL_NO_PNG 1` and change it to `#undef IL_NO_PNG`  

4. Create the following folder trees in DevIL:
	```
	DevIL/libs/Win32/Release
	DevIL/libs/Win32/Debug
	```

5. Paste the libs from `lpng/projects/vstudio/Release Library(.lib && .pdb)` to `DevIL/libs/Win32/Release`  

6. Paste the libs from `lpng/projects/vstudio/Debug Library(.lib && .pdb)` to `DevIL/libs/Win32/Debug`  

#### Building LibPng and zlib inside DevIL:
<ins>__Do the same for debug, if you need it(or change to all configurations)__</ins>

1. Right click on `IL` project  

2. `Librarian -> General -> Additional Dependencies ->` add `libpng16.lib; zlib.lib`  

3. `Librarian -> General -> Additional Library Directories ->` add `$(SolutionDir)../libs/$(Platform)/$(Configuration)/`  


#### <ins>Static Lib Configuration(optional)</ins>:
`Select IL, ILU and ILUT -> Right Click -> Properties`  

#### <ins>Configuration: same for both(Release and Debug)</ins>
1. `General -> Configuration Type -> Static Library -> Apply Changes`  
2. `Advanced -> Target File Extension -> .lib -> Apply Changes`  

#### <ins>Release</ins>:
`C/C++ -> Code Generation -> Runtime Library -> Multi-threaded(/MT) -> Apply Changes`  

#### <ins>Debug</ins>:
`C/C++ -> Code Generation -> Runtime Library -> Multi-threaded Debug(/MTd) -> Apply Changes`  



## Client Extern
1. Build solution

2. copy the libs from `DevIL/build/lib/platform/config` to `your client's extern(libs)` folder  

3. Change __your__ `DevIL(might be called IL) include` folder with the one from `DevIL/include`  


## Client Source
#### <ins>UserInterface/UserInterface.cpp</ins>
find  
`#pragma comment( lib, "DevIL-1.7.8.lib" )       // images`  

and change it to  
`#pragma comment( lib, "DevIL.lib" )       // images`  

add this as well if you want more descriptive errors(read further)  
`#pragma comment( lib, "ILU.lib" )       // images`  

find   
`ilInit();`  

add  
`iluInit();`  


#### <ins>UserInterface/PythonApplicationModule.cpp</ins>
You don't need ILU.lib if you don't want to do this.

find   
`#include <il/il.h>` or just look for `il.h`  

and add  
`#include <il/ilu.h>`  

find  
```cpp
	if (ilLoad(IL_TYPE_UNKNOWN, szFileName))
	{
		// [...]
	}
```
And add this  
```cpp
	else
	{
		ILenum Error;
		while ((Error = ilGetError()) != IL_NO_ERROR)
		{
			TraceError("Image %s failed to load. ErrorID: %d. ErrorMsg: %s\n", szFileName, Error, iluErrorString(Error));
		}
	}
```

You might get errors about the includes, make sure the include folder matches(Might be il-1.7.8/whatever, change it to il/whatever).


## Server source
I only use `windows`, sorry, I don't have a tut for `freeBSD`, but it should follow the same principles.

#### <ins>Extern</ins>:
Change your include folder and libs, like you've done with the client.

#### <ins>game/src/GuildMarkUploader.cpp</ins>
If you get an error from this one, find `ILvoid` and change it to `void`  

From this:  
`	ilCopyPixels(0, 0, 0, SGuildMark::WIDTH, SGuildMark::HEIGHT, 1, IL_BGRA, IL_BYTE, (ILvoid*)m_kMark.m_apxBuf);`  

to  
`	ilCopyPixels(0, 0, 0, SGuildMark::WIDTH, SGuildMark::HEIGHT, 1, IL_BGRA, IL_BYTE, (void*)m_kMark.m_apxBuf);`  


