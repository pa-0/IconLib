# IconLib - Icons Unfolded (MultiIcon and Windows Vista supported)

By [CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897)

15 Feb 2008
[CC (ASA 2.5)](http://creativecommons.org/licenses/by-sa/2.5/ "The Creative Commons Attribution-ShareAlike 2.5 License")
41 min read 

Library to manipulate icons and icons libraries with support to create, load, save, import and export icons in ico, icl, dll, exe, cpl and src format. (Windows Vista icons supported).

-   [Download source files - 118.5 KB](https://www.codeproject.com/KB/cs/IconLib/IconLib_src.zip)
-   [Download demo project - 1.52 MB](https://www.codeproject.com/KB/cs/IconLib/IconLib_demo.zip)

![image013.jpg](https://www.codeproject.com/KB/cs/IconLib/image013.jpg)

## Introduction

At some point in the last month, I needed to create an .ICO file on the fly with a couple of images inside; preferably I was looking at code in C#. The .NET Framework 2.0 only supports **HICON** that basically is one Icon with just a single image in it. When I was searching out there, to my frustration, I did not find any Icon Editor with the source code. The only thing I found was closed commercial products charging from $19 to $39 for them and not exposing APIs at all. So the only solution was to create my own library capable of creating and parsing ICO files.

I believe in open-source code and I thought that I could help the developer community by sharing this knowledge. In addition, open-source pushes companies and commercial products to go farther.

After the work was done, I read about ICL files (Icon Libraries). These can contain many Icons inside a file and I decided to support that too. The same happened with EXE/DLLs and last but not the least I decided to support Windows Vista Icons. All this was really hard work and a lot of headache because there is not much information exposed. I ended up spending a lot of time reverse-engineering and researching over the net. I hope it will be useful for you as it is for me.

>[!Note]
>As in every new project, many things can happen. Not every case can be tested and many things cannot be seen even after they are tested. Since it is a very fresh project, if there is something that doesn't work, or you think should work differently than it does, before you give your vote, write a post and give me the chance to fix it. That way, we both get the benefit of getting more stable code and creating a more complete library, and at the same time you get my thanks if that helps.

I have also included two libraries as samples. I borrowed some icons from Windows Vista, for the 256x256 versions and I put a watermark because the icons has copyright ownership. Hopefully I won't have trouble with that.

### Objective

The objective of the library is to create an abstraction layer from the different file formats and to provide an interface to allow icon modification without the hassle of knowing internal file formats.

### Current Formats Supported

-   ICO Read and write icons with different images size and depths
-   ICL Read and write icons inside the icon library
-   DLL Read DLLs and export to a new DLL
-   EXE Import
-   OCX Import
-   CPL Import
-   SRC Import

## Multi-Icon

`Iconlib` exposes three different objects.

1.  `MultiIcon`: This is the only object that can be instantiated in the library. Once a `MultiIcon` object is created, it allows APIs to load and save in the file system or streams with standard formats as ICO/ICL/DLL.
2.  `SingleIcon`: This represents a single icon inside `MultiIcon` and it allows add/remove images in it.
3.  `IconImage`: This represents a single image inside `SingleIcon`. At this point, `IconImage` exposes the icon's lowest resources like the `XOR` (Image) and the `AND` Image (Mask). It also exposes an Icon property that will basically construct a .NET Icon created from the `XOR` and `AND` image from where you can get a `HICON` handle.

As you can see there is a hierarchical structure, basically a `MultiIcon` contains `Icons` and an `Icon` contains `Images`.

## Library Objects Diagram

![image019.jpg](https://www.codeproject.com/KB/cs/IconLib/image019.jpg)

The library contains many classes and structs but only exposes the three important classes. The developer needs to control the complete behavior of the library, the rest are all internal, many classes/structs and methods are not safe to be exposed to the developer. For that reason, I recommend keeping `IconLib` as a separated project because if the developer incorporates the source in his/her project, all the internal classes/structs/methods will become visible and probably will not be used properly.

I cannot give support for the library when it is not used the way it was designed. I'm providing the source code as a nice gesture because I believe in open-source code, and I hope you will make good use of it without ripping of the source from where it belongs.

## Icon Format

Before I started `IconLib`, I had no clue how icons work. However, I was not too long in the net before I found the excellent article [Icons in Win32](http://msdn2.microsoft.com/en-us/library/ms997538.aspx)

Although this article is outdated with the arrival of Windows Vista icons, it is very precise in explaining how Icons format files are.

Something to take care; in my first version, I followed the icon format details but the library could not load some of the icons I was testing. When I went deep into the bytes, I could notice that much of the information was missing from the directory entry.

I tested those Icons with another product and I could see that, for example one popular product had no problem opening this kind of icon, and that is because every icon directory entry points to a `ICONIMAGE` struct, this `ICONIMAGE` struct has a `BITMAPINFOHEADER` struct that contains more information than the icon directory entry itself. So basically, using the information from the `BITMAPINFOHEADER`, I could reconstruct the information in the directory entry.

The same rule cannot be applied to Windows Vista Icons, because those images don't contain a `BITMAPINFOHEADER` struct anymore. Hence, if some information is missing from the directory entry, the icon image becomes invalid.

Anyway, reconstructing the icon directory entry is a plus and discarding icon image not properly constructed is acceptable, no company should provide icons with missing information in the headers.

## NE Format (ICL)

NE Format is the popular format to store icon libraries; this format was originally used for Executables on 16-bit version of Windows.

You can get more information for NE format from the Microsoft web site at [Executable-File Header Format](http://support.microsoft.com/default.aspx?scid=kb;EN-US;q65122)

This was the most challenging part of the project. When I started researching about ICL, I had no clue that these were 16-bit DLLs. I couldn't find any data about this extension and couple of days later, I almost dropped the project. But I read in some place that ICLs are 16-bit with resources inside so my quest started on how to recover resources from a 16-bit DLL. So far my only next objective was trying to load in memory a 16-bit DLL. Of course, at first I tried to load the library with standard Win32API `LoadLibrary`, `LoadLibraryEx` but this failed with:

```
193 - ERROR_BAD_EXE_FORMAT (Is not a valid application.)
```

I'm not an expert in Kernel memory allocation but I guess this is because in Win32, the memory is protected between applications and in 16-bit is not so when trying to allocate memory for 16-bit the OS rejects the operation.

The next step was trying to load the ICL (16-bit DLL) in memory using just 16 bits APIs. If you read the MSDN WIN32 API, the only API left for 16-bit is `Loadmodule`

When I tried, it loaded the library but immediately Windows started giving strange message boxes, as "Not enough memory to run 16-bit applications" or things like that.

I wrote in Microsoft forums and other forums, but found nothing really helpful on how I could get those resources. At that time, it was very clear that I could not load 16-bit DLL in memory and that I needed to create my own NE parser/"linker".

Microsoft article about NE Format (New Executable) is an excellent source and describes in detail every field in the file.

A NE format file start with an `IMAGE_DOS_HEADER`, this header is there to keep compatibility with MS-DOS OS. This header also contains some specific fields to indicate the existence of a new segmented file format. `IMAGE_DOS_HEADER` usually contains a valid executable program to run on MS-DOS. This program is called a stub program and usually it just prints the message on the screen 'This program cannot run on MS-DOS'.

![MS DOS Header](https://www.codeproject.com/KB/cs/IconLib/image004.gif)

After we read the `IMAGE_DOS_HEADER`, the first thing to do is to know if this is a valid header. Usually every file contains what is called a magic number. It is called a magic number because the data stored in that field is not relevant to the program, but it contains a signature to describe the type of the file.

You can find Magic Number almost everywhere. The magic number for the IMAGE_DOS_HEADER is `0x5A4D`, this represents the chars 'MZ', and it stands by "Mark Zbikowski" who is a Microsoft Architect and started working with Microsoft a few years after its inception. Probably he could never have thought that his signature was going to be used thousands of times in almost every personal computer in the world.

If the magic number is 'MZ' then the only extra field we care about is the `e_lfanew`, this header is the offset to the new exe header, NE Header.

We search in the file for this offset, and then at this point we read a new header. This header is `IMAGE_OS2_HEADER` and it contains all information about the program to be loaded in memory.

![NE Header](https://www.codeproject.com/KB/cs/IconLib/image005.gif)

The first thing to do is to load the magic number again, but this time the magic number must be `0x454E` and it means 'NE'. If the signatures match, then we can continue analyzing the rest of the headers. At this point the more important field is `ne_rsrctab` as this field contains the offset of the resource table. From this offset, we get the number of bytes we have to jump from the beginning of this header to be in position to read the resource table.

If everything went well, we are ready to read the resource table.

![Resource Table](https://www.codeproject.com/KB/cs/IconLib/image006.gif)

The first field of the Resource Table is the align shift, usually you find the explanation as "The alignment shift count for resource data. When the shift count is used as an exponent of 2, the esulting value specifies the factor, in bytes, for computing the location of a resource in the executable file."

In my own words, the working of this field was tricky to understand. It was created for compatibility with MS-DOS, and it will contain the multiply factor necessary to reach the resource.

As you will see, the resource offset is a variable of type `ushort`, which means that it can only address 64Kb (65536). Actually almost every file is bigger than that, and here is where the 'alignment shift' field comes to play.

Alignment shift is a `ushort` and "usually" it is in the range of 2 to 10. This number is the number of times we have to shift the number 1 to the left. For example:

-   Alignments shift of 5 means 1 << 5 which is equal to 32.
-   Alignments shift of 10 means 1 << 10 = 1024

Now with the virtual offset address from the resource table we multiply for the result shift value and we get the real offset address in the file.

#### For Example

The resource located at the virtual address 0x2000 and the alignment shift is 5 then we get:

```
Realoffset = (1 << 5) * 0x2000
Realoffset = 32 * 0x2000
Realoffset = 0x40000
```

The real offset of this resource is at 262144 (0x40000).

Wow, this is cool right? Because we just use an `ushort` and we can locate a resource at any position. Now you will wonder where the trick lies?

The trick is for example if you use a shift alignment of 5 that means the minimum addressable space is 32 bytes (1 << 5), which means if you want to allocate 10 bytes with this method 32 bytes will be allocated and just the first 10 will be used, another 22 bytes will be wasted.

Now you might wonder, ok then let’s take the shift alignment as 0, then e won't waste space because the virtual address will match with the real address. It is not so easy, and that works only if the resource is located in the range of the first 64Kb space.

So to make it clear, this shift alignment is directly proportional to the file size.

The next table tells what the maximum file sizes are that you can get with different shift alignments:

```
(1 << 0) * (2 ^ 16) = 64KB
(1 << 1) * (2 ^ 16) = 128KB
(1 << 2) * (2 ^ 16) = 256KB
(1 << 3) * (2 ^ 16) = 512KB
(1 << 4) * (2 ^ 16) = 1MB
(1 << 5) * (2 ^ 16) = 2MB
(1 << 6) * (2 ^ 16) = 4MB
(1 << 7) * (2 ^ 16) = 8MB
(1 << 8) * (2 ^ 16) = 16MB
(1 << 9) * (2 ^ 16) = 32MB
(1 << 10) * (2 ^ 16) = 64MB
```

Calculating this value is not so easy. `IconLib` at first uses a shift factor of 9 because I thought that 32MB was more than enough for an `Icon` library. But great was my surprise when I extracted Windows Vista DLLs and `IconLib` got out of range for some files. I then incremented the shift factor to 10 which enabled me to dump the content of the Windows Vista DLL in an ICL file, but it took 63MB.

A factor of ten allows to us to create an ICL library up to 64MB but every resource will address at minimum 1024 bytes. If you think that's not bad because all resources will be bigger than 1024, it is not so easy. A factor of ten means it can address in multiples of 1024, then if the resource is 1025 then it will allocate 2048 bytes in the file system.

In conclusion, with a factor of 10, `IconLib` is wasting an average of (1024 / 2) 512 bytes by resource allocated, but at the same time it lets us create an `Icon` Library with 64MB.

My next release will predict the max file size and will adjust the shift factor dynamically; it is not an easy task if you want to predict the number without scanning memory to know the max space to be addressed, especially for PNG images where this value is dynamic too.

Hopefully shift alignment field is clear now and we come back to the resource table.

The next field is an array of `TYPEINFO`

![TYPE INFO](https://www.codeproject.com/KB/cs/IconLib/image007.gif)

`TYPEINFO` is a struct that gives us information about the resource; there are many types of resources that can be allocated, but `IconLib` is just interested in two types `RT_GROUP_ICON` and `RT_ICON`.

When `IconLib` reads the `TYPEINFO` array, it discards all structs where `rtTypeID` is not `RT_GROUP_ICON` or `RT_ICON`

`RT_GROUP_ICON` type gives us information about a icon.

`RT_ICON` type gives us information about a single image inside the icon

`rtResourceCount` is the number of resources of this type in the executable.

`rtNameInfo` is an array of `TNAMEINFO` containing the information about every resource of this type. The length of this array is equal to `rtResourceCount`.

![TNameInfo](https://www.codeproject.com/KB/cs/IconLib/image008.gif)

Here is where we have the information about the resource itself; the `rnOffset` is the virtual address where the physical resource is located. To know the real address, see how alignment shift works above.

The `rnLength` is the length of the resource on a virtual address space. This means if for example the resource has a length of 1500 bytes and the alignment shift is 10, then the value on this field will be 2.

The way to calculate the length is:

```csharp
rnLenght = Ceiling(ralresourcesize / (1 << resource_table.rscAlignShift));
rnFlags tell us if the resource is fixed, preloaded, or shareable
rnID is the ID of the resource.
rnHandle is reserved.
rnUsage is reserved.
```

Going back to `TYPEINFO`, if the `TYPEINFO` struct type is `<a name=""OLE_LINK6""></a>RT_GROUP_ICON` then we read the array of `TNAMEINFO` which gives us information about every icon in the resource.

The offset in every `TNAMEINFO` will contain a pointer to a `GRPICONDIR` struct, this struct will give information about a single icon, like how many images it contains and one array of `GRPICONDIRENTRY`, whenever `GRPICONDIRENTRY` contains information about the image, like width, height, colorcount, etc.

Now if the `TYPEINFO` struct type is `RT_ICON` then we read the array of `TNAMEINFO` which gives us information about every single image inside the resource.

Going back to the Resource Table we have another three fields, `rscEndTypes`, `rscResourcesNames` and `rscEndNames`.

`rscEndTypes` is a `ushort` value that tell us when to stop reading for `TYPEINFO` structs. The resource table struct doesn't tell how many TYPEINFO structs it contains, so the only way to know it is with a stopper flag. This flag is `rscEndTypes`. If when we read `TYPEINFO` the first two bytes are zero, then it means that we reached the end of the `TYPEINFO` array.

`rscResourceNames` is an array of bytes with the names of every resource in `TYPEINFO` struct. The names (if any) are associated with the resources in this table. Each name is stored as consecutive bytes; the first byte specifies the number of characters in the name.

For example if the array is `[5, 73, 67, 79, 78, 48, 5, 73, 67, 79, 78, 49]`

This is translated like an array of two strings "`Icon1`", "`Icon2`".

```
[5, 73, 67, 79, 78, 48, 5, 73, 67, 79, 78, 49]
[73, 67, 79, 78, 48] = "Icon1"
[73, 67, 79, 78, 49] = "Icon2"
```

If you wonder when you have to stop reading for bytes in the array, there exists another stopper flag `rscEndNames`with a value of zero. When the bytes are being read, if a null ('\x0') character is detected, then the process must stop reading the names, and they are ready to be translated as ANSI strings.

At this point we already have all the information and binary data for the `Icon`s and the images inside the `Icon`.

`IconLib`loads all the `Icon`s and `Icon` images in memory to obtain a good performance while working with them. In addition, it does not need to lock the file on the file system.

Creating an ICL file is not so complex after all. Because `IconLib` creates an ICL from scratch, it doesn't have to care about the other segments in the NE Format, so the process is relatively simple. We write an `IMAGE_DOS_HEADER`, write the `MSDOS` stub program, write an `IMAGE_OS2_HEADER`, where we choose the right alignment factor and we write the resource table at the location specified by the field `ne_rsrctab`in the `os2_header`.

When the resource table is written it has to apply the same rules when loading. This means write a partial resource table struct, the two `TYPEINFO`structs (`RT_GROUP_ICON`and `RT_ICON`and inside the `TYPEINFO``</ode>`, write the `TNAMEINFO` information)

![NE Format Sample](https://www.codeproject.com/KB/cs/IconLib/image009.gif)

The following table shows a NE format that stores 2 Icons, the first icon contains one image, the second icon contains 2 images.

That is something that I would like to mention. As I mentioned before, I redesigned the core 3 times, the first time I followed every known specification on how the Icon file has to be read and written from Icons and DLLs. When I exported icons from DLLs, I kept all information about the icon, as the Icon names, group ID and icon ID. When I saved them on the file system, I saved them in the same way I read them, so basically I could export the icons from a DLL, export it to a ICL file, then load the ICL and export to a DLL, and I would keep the same IDs for the groups and Icons.

So far I tested two popular commercial products and they could open them without problems, but for example I started to have problems when I exported some DLLs or EXEs to ICL files. For example if you open _explorer.exe_ from Windows folder in Visual Studio, the first thing you will notice is that the icons IDs are not consecutive, they start with ID 100, 101, 102, 103, 104 and jump to 107, and continue.

`IconLib`exported _explorer.exe_ to an ICL file, and import it on Visual Studio and there was no problem at all. But to my surprise, when I tried to open it with a popular Icon Editor, the icon library showed images with icons mismatched and mixed between the icons. I spent many days trying to figure why it was happening.

Basically after many tests of different applications, I could notice that those applications write the ICL files and discard the Icons and Group IDs and they expect consecutive IDs.

For ICL files there is a header to be written `TNAMEINFO`, this header contains a field which is the ID. This ID can be a `GRPICONDIRENTRY` (Icon itself) or an `ICONDIRENTRY` ID (single image ID inside the icon). When those applications write ICL files, they do it in a consecutive way, basically they discard the IDs when they imported from the DLL and write groups id as 1, 2, 3, 4, same for the icons id, they do 1,2,3,4. etc.

So basically I noticed that some applications are not prepared to handle ICL files properly for all cases. Another not so popular application passed it and basically it could read ICL files where the IDs were not consecutive, but still when it saved the ICL file it discarded the source ID and put its own.

So I had a big dilemma. Should I keep all the information and write that information as it is coming from the EXE/DLLs in the ICL files; that would make my ICL files properly constructed but incompatible with some applications out there. Or should I discard the original IDs in the importation and create consecutive IDs which means discard part of the original information and put my own (I was not keen on this solution), but small fishes can swim in a pool with big fishes unless they behave like one.

So I didn't have another choice than to redesign my core to produce those results using consecutive IDs. After I redesigned I reduced the source code because now I didn't need to keep all the information that was generated on the fly, but when icons are exported from the DLLs the original Icons IDs are lost. Anyway, a regular developer will rarely use those IDs.

I still wonder if it is a mis-implementation of those products to fully support ICLs, or if there is a rule in the NE Format files that says you can't store a resource with a "random" ID. So far, all my research concludes that you can use any ID for the `ICONIMAGES`inside NE Format.

## PE Format (DLL, EXE, OCX, CPL, SRC)

PE Format means Portable Executable; this format was created by Microsoft to supports 32-bit and 64-bit version of Windows in NE Format replacement used for 16-bit version of Windows.

Basically files format like EXE, DLL, OCX, CPL, SCR don't differ too much amongst them. For example, think of an EXE like a DLL with an entry point. When working with resources, all those files are identical. This means if the library supports PE Format then it supports all the above extensions.

Because Win32 API already supports resources handling for PE format, then it was not necessary to support this file format natively, instead IconLib makes use of Win32 APIs to gain access to the icons resources.

The only native functionality was to read the first set of headers from the PE file to detect whether the file to be loaded is a PE format or not.

If we want to access just the resources then the best way to do it is to load the library as a `DATAFILE`. This means no code at all will be executed from the library, instead Win32 API will access just the resources data.

```
hLib = Win32.LoadLibraryEx(fileName, IntPtr.Zero, LoadLibraryFlags.LOAD_LIBRARY_AS_DATAFILE);
```

`IconLib`core just supports reading and writing from and to a stream. `MultiIcon`overloads some functions as `Load`/`Save` and creates a `FileStream` from a file in the file system before calling the `Load(stream)`. Win32 API `LoadLibrary`can only load libraries from the file system; therefore the stream will be saved in a temporary file before Win32 API `LoadLibraryEX`is called.

Access to the resources is an easy task when the resources are accessed in the proper order.

The first thing that `IconLib`does is call `Win32.EnumResourceNames`sending as parameter `GROUP_ICONS`, which gives us back the ID of every icon. This ID can be a number or a pointer to a string. If the value returned is less than 65535 then it is a number, if the value if bigger than 65535 then it is a pointer to one string.

Once we have all the IDs for the icons, we call the function `Win32.FindResource`. For every ID found, this gives us a handle to the resource and then we can proceed to load and lock the resource to access the resources entries. Those entries contain the IDs of every image inside the icon just loaded/locked. Now we repeat the steps that we did before, but instead of using the constant code>**GROUP_ICONS** we use `RT_ICON`. This tells the Win32 API that we want to access the image inside the icons.

Here is the critical step that needs to be done. Under Windows XP or previous OS, after we lock the resource for the icon image we will have a pointer to an `ICONIMAGE`. This icon image will contain the `BITMAPINFOHEADER`, `Palette`, `XOR``Image`, and the `AND`Image (mask), but in Windows Vista instead, it returns a pointer to a `PNG` image, basically this is the main reason why current Icon Editors including the more popular ones will crash. They will allocate huge amount of memory or just drop the image because they will be parsing a `PNG` image like a `BMP` image.

`IconLib`resolves that issue by reading the first bytes of the Image and detecting the signature of the image creating the proper encoder instance before reading and parsing the image.

When `IconLib`has to create a `DLL`, the best way so far was to use an empty `DLL` as a template, and add the resources to it.

Win32 API offers three APIs that will do the job for us.

1.  `BeginUpdateResources`
2.  `UpdateResources`
3.  `EndUpdateResources`

MSDN tells us that you can call `BeginUpdateResources`, and then call `UpdateResources`as many times as you want, the file won't be written yet. At last you call `EndUpdateResources`and the changes are committed to the `DLL`.

That methodology worked pretty good for small `DLL` files. When `IconLib`was creating libraries with more than 80 images, everything was OK but the call to `EndUpdateResources`always failed. After a lot of unsuccessful tries, the only thing I could think of was that the API to update resources has an internal buffer. When that buffer is full, calls to `EndUpdateResoruces`fail to commit the changes into the `DLL`.

The workaround that I found was to commit every time on an average of 70 updates, this worked pretty well but enormously increased the time to update the `DLL`. For that reason, unless I can find why Win32 is doing that, I'll try to come up with my own PE Format implementation, and not use the Win32 at all. That will speed up the process a lot.

You can get more information for PE format from Microsoft web site at [Microsoft Portable Executable and Common Object File Format Specification](http://www.microsoft.com/whdc/system/platform/firmware/PECOFF.mspx)

## Windows Vista Icons Support

I wanted to create a library to work with icons without having any limitations, so support for Windows Vista was a must.

In Windows XP, they introduced icons with an alpha channel and 48x48 pixels. In Windows Vista, Microsoft introduced icons images with a size of 256x256 pixels. This image inside the icon can take 256Kbytes for the image and another 8Kbytes for the mask in uncompress format. That increases the size of icons library substantially and basically resolves this issue of storing the image in a compressed format.

The compression used was PNG (Portable Network Graphic) because it is free of patents, supports transparency (Alpha Channel) and employs lossless data compression.

The factor is on an average between 3 to 5 times smaller than uncompress bitmaps.

If you think there is not much difference, then load the file _imageres.dll_ from _Windows\System32_ in Windows Vista (11MB), do a for loop for all images and set the encoder to be BMP instead PNG, then save it to a DLL or a ICL file. You will notice that the DLL is about 45MB and the ICL about 54MB. This is where you can see that PNG really makes the difference.

To store the compress image, they could come up with a way to keep some backward compatibility and this was setting the field `biCompression`in `BITMAPINFOHEADER`to `BI_PNG` instead of `BI_RGB`. This header is already supported from Windows 3.1 and the filed `BI_PNG`from Windows 95, but instead they broke compatibility and they store the image alone. (see 'Ohh Microsoft policy about compatibility is changing?' below)

![Windows Vista Icons](https://www.codeproject.com/KB/cs/IconLib/image010.gif)

The sample only contains two images but there can be up to 65535.

Although in all my research, I saw only 256x256 images in PNG format, that doesn't mean it could not store all images as PNG. This was only a decision to have compatibility with previous version of Windows.

Personally I think Icons editors should support PNG at any size and bits depth. Icons not only are used by Windows OS, it is the same why Windows icons allow introducing non-standard images like 128x96x24 when Windows will never make use of it.

If you are creating an icon that Windows Vista will make use of, only store PNG compression for 256x256 images.

## Smart Classes/Structs

The more difficult stuff was how to come up with a clean code and APIs capable of understanding different icons formats and icons libraries and also different image compressions without creating a chaos of switch/if/else.

In my journey of creating the library, I redesigned the core from scratch 3 times, and still there is a TODO changes to avoid `IconImages`objects from knowing about different compression methods. Basically an `IconImage`should not be responsible for knowing the format of the image to be read/written. Instead it should depend on the different encoders to know this information.

Right now `IconImage`has a reference to an `ImageEncoder`object (base class), but still `IconImage`object is responsible for discovering the signature of the image to know if it has to create a `BMPEncoder`or a `PNGEncoder`instance.

Also there are a couple of changes to manage the memory allocations more efficiently, but that won't change the core design.

Coming back to what I called Smart Class/Structs: Basically an Icon is a hierarchical structure and Icon libraries are the same but contain one more level of information.

The objective of this smart classes/structs was to avoid interchange data between the different objects; instead every class/struct should be capable of reading and writing itself. If a class of struct contains more classes or structs inside, then it should ask the child to read/write that portion of information and so on.

If you open the source code, immediately you will notice that the parameter **'**`Stream stream`**'** is everywhere. This allows the object that receives this parameter to read/write itself in the stream at the current position.

For example, when a `Icon` file has to be created, the `MultiIcon`object will open a `FileStream` and will call `ImageFormat.Save(stream)`, sending as a parameter the stream just opened.

`ImageFormat` object will contain only the logic to write itself and rely on the different classes/struct to write the rest of the information.

```
ImageFormat.Save(stream)
{
 ICONDIR.write(stream)
 {
   Write iconDir header
 }
 
 Loop for each IconImage
 {
   ImageEntry.Write(stream)
   {
     Write iconEntry header
   }
 
   Image.Write(stream)
   {
     BitmapInfoHeader.Write(stream)
     {
       Write bitmap info header
     }
     Write Color Palette
 
     Write XOR image
 
     Write AND image
   }
 }
}
```

This is a simple case, but more complex cases like reading ICL (Icon Libraries) follow the same behavior.

So, following this model writing and reading different formats was really easy. It also produced a super, cleaner code.

## Image Encoder

I wanted to provide a library easy to understand and flexible enough to adapt to any kind of image format. The ideal case was to create a class with basic functionality but to leave the specific format implementation to other classes.

`ImageEncoder`object keeps all the information about one icon image and it contains information like image properties, palette, icon image, and icon mask.

This class is an abstract class that cannot be instantiated.

## BMP Encoder

`BMPEncoder` class has the logic to read and write Icon entries when `biCompression` is `BI_RGB` (BMP)

## PNG Encoder

`PNGEncoder` class has the logic to read and write Icon entries when image is PNG format.

I followed the information I could get from different sources to create Icons with PNG compression. So far the implementation doesn't have problems and Icons Images in PNG format can be opened with all Icons editors that support Windows Vista.

`Icon` libraries are a different subject. So far I did not find a single open source or commercial icon editor, including the more popular ones that allow opening or writing icon libraries like ICL or DLL with PNG compression, In some commercial products you will notice that the PNG icons are not loaded and also if you create PNG icons they are uncompressed before getting saved on a DLL or ICL. I think that is because Microsoft still didn't release any information about it and companies are waiting for the final Windows Vista to come out.

I based my work for creating compressed icon libraries (ICL, DLL) on reverse engineering in Windows Vista RC2 and following the same logic that the Microsoft boys used for icon files.

`IconLib` is capable of loading all icons from Windows Vista DLLs/EXEs and CPL files (PNG format inclusive). It also allows writing ICL/DLL icon libraries with PNG compressions.

The bad news is that only you can load them with `IconLib`for now. If you try to load an ICL or DLL with PNG images generated with `IconLib`and try to open it with third party icon editor, you will see that the PNG icons are gone, and also the icons contain images from other icons. So far, all my research concludes that there is a mis-implementation of PNG format for ICL libraries in those products and has nothing to do with `IconLib`.

Now if you wonder how I can be sure that `IconLib`generates ICL or DLL properly?.

Basically if you try to open Windows Vista icons that contains 26x256 PNG icons in any VisualSudio version (Orcas inclusive if you wonder about VS2006 so far), it will show an image with a size about 2573x1293 with XP format. Of course that image doesn't exist and you can't edit it, but that's how Visual Studio sees it.

Now if you load a DLL with 256x256 PNG files, generated with `IconLib`and save the icon that contains the PNG image to the file system and open the icon image with Visual Studio, you will notice the same behavior as the DLLs from Windows Vista.

Anyway, the entire work is based on suppositions, and I can't be really sure yet as long as any Windows Vista Libraries Icons Editors hit the market or Microsoft releases more information about it.

## Color Reduction and Palette Optimization

Usually there are programs that allow creating icons from a bitmap and they produce an Icon with alpha channel (transparency) compatible with Windows XP, those icons lacks of the support of low resolution images, Iconlib allows to add a low resolution image and also incorporate a whole namespace to produce a low resolution image from a high resolution image.

The techniques used by IconLib are:

-   Palette Optimization
-   Color Reduction
-   Dithering

### Palette Optimization

A palette is an array of RGB colors, most of the times the length of the palette is the amount of colors supported, a palette can contains any length but in most of the cases the palettes are 256 or 16 indexes.

An optimized palette is created on base to the bitmap to be processed; it will analyze the input image and will create a new palette with the most used colors from the input image, many ways might be used to create an optimized palette.

Why use a palette on a Bitmap?

Every index in the palette is a RGB color, 3 bytes are necessary to create the color (1 byte for Red, 1 byte for Green, 1 byte for Blue), this allow to create a combination of 16 million colors because each channel can produce a 256 color gradient, then 256R * 256G * 256B = 16777216 color combinations.

If on the bitmap data we store the RGB information then at least we require 3 bytes to store every pixel color.

Instead, indexed bitmaps will store just one index to an array of colors; this means that the bitmap data does not contain color information but an index to an array (palette).

This can save a lot of space but the image quality may suffer considerably because very similar colors on non-indexed image will be converted to the same color (index) on an indexed image.

There are many more data store in a Bitmap but just for example let’s compare the size of 3 bitmaps.

#### 100x100 pixels 24bpp image

1 pixel = 3 bytes

100x100x3 = 30000 bytes to store the color information.

#### 100x100 pixels 8bpp indexed image

1 pixel = 1 byte

1 palette = 256 indexes of RGB color = 256x3 = 768

100x100x1 + 768 = 10768 bytes to store the color information.

#### 100x100 pixels 4bpp indexed image

1 pixel = 1/2 byte

1 palette = 16 indexes of RGB color = 16x3 = 48

100x100x1/2 + 48 = 5048 bytes to store the color information.

The key to have a low resolution indexed image and still good looking is to choose the right color for the palette, there are different palettes that can be used.

**System palette:** this is the default Windows palette and it contains 256 colors, it has a variety of colors in a wide spectrum, IconLib make no use of this palette because if for example the icon to be color reduced has many gradients when those gradient are converted to an index pixel version many of them will have the same index and the quality of the image will be greatly degraded.

**Exact:** If the image contains less than 256 colors, those are mapped directly to the palette.

**Web:** Is the intersection between Windows and Mac OS palette, it contains 216 colors that are safe to be used on Windows or Mac OS systems.

**Adaptive:** This palette reduces the colors in the bitmap based on their frequency; for example, if your image contains mostly skin tones, the adaptive color palette will be mostly skin tones.

**Perceptual:** This palette is weighted toward reducing the colors in the bitmap to those to which we are the most sensitive.

**Selective:** The Selective palette will choose the colors from the bitmap to the web-safe colors.

**Custom:** A custom palette might be provided.

IconLib creates an optimized palette using the Adaptive algorithm with an [Octtree](http://en.wikipedia.org/wiki/Octree) structure.

### Color Reduction

The idea behind color reduction is take an 32bits (ARGB) or 24bits (RGB) image where the data of every pixel contains the RGB color information and convert this image to a indexed image, they are called indexed because every pixel data **does not** contain the RGB color information instead it contains a index to a palette (Array of colors), this palette store n numbers of colors, 32bits and 24bits images can produce 16 million colors and every pixel is stored as 3 bytes (4^th^ byte for alpha channel). Because indexed just store an index to the palette the store needed depends of the image resolution.

Non-Indexed 32 bits (16M colors plus transparency) = 4 bytes per pixel

Non-Indexed 24 bits (16M colors) = 3 bytes per pixel

Indexed 8 bits (256 colors) = 1 byte per pixel

Indexed 4 bits (16 colors) = 1/2 byte per pixel or 2 pixel per byte

Indexed 1 bit (Black&White) = 1/8 byte per pixel or 8 pixel per byte

In IconLib color reduction algorithm works pretty close with the palette optimization algorithm.

Before a pixel can be converted to an indexed pixel a palette must be available to choose the right color index.

Different palettes can be used in the process of color reduction.

See Palette Optimization above.

The algorithm I have use in the color selection was the Euclidian distance, basically it finds the nearest neighbor color in the palette, it maps the current color in the image with a color in the palette finding the shortest distance between the current color and the neighbor color in a 3D space.

### Dithering

Even when an optimized palette is used in the process of color reduction the resulting image may looks not good especially when the input bitmap contains high number of gradient, to improve the looking of image dithering is used.

[Dithering](http://en.wikipedia.org/wiki/Dither) is the process of juxtaposing pixels of two colors to create the illusion that a third color is present, basically noise is added in the process, this noise is proportional to the different color gaps between pixels.

There are many algorithm to implement dithering, and the output image vary between them, personally I like Floyd-Steinberg algorithm because the noise generated is spread uniformly creating a nice looking image.

**No dithering:** no noise is added to the output bitmap.

There are three kinds of dithering:

**Noise dither:** It is not really acceptable as a production method, but it is very simple to describe and implement. For each value in the image, simply generate a random number 1..256; if it is greater than the image value at that point, plot the point white, otherwise plot it black.

**Ordered dither:** Ordered dithering adds a noise pattern with specific amplitudes, for every pixel in the image the value of the pattern at the corresponding location is used as a threshold. Different patterns can generate completely different dithering effects.

**Error diffusion:** diffuses the quantization error to neighboring pixels.

**Floyd-Steinberg dither:** it is an error diffusion dither algorithm and is which is used in IconLib, it is based on error dispersion. For each point in the image, first find the closest color available. Calculate the difference between the value in the image and the color you have. Now divide up these error values and distribute them over the neighboring pixels which you have not visited yet. When you get to these later pixels, just add the errors distributed from the earlier ones, clip the values to the allowed range if needed, then continue as above.

In the following sample it reduces the image to 8, 4 and 1bpp from a 24bpp source image.


```csharp
IColorQuantizer colorReduction = new EuclideanQuantizer(new OctreeQuantizer(), new FloydSteinbergDithering());
Bitmap bmp = (Bitmap) Bitmap.FromFile("c:\\Pampero.png");

Bitmap newBmp = colorReduction.Convert(bmp, PixelFormat.Format8bppIndexed);
newBmp.Save("c:\\Pampero 8.png", ImageFormat.Png);

newBmp = colorReduction.Convert(bmp, PixelFormat.Format4bppIndexed);
newBmp.Save("c:\\Pampero 4.png", ImageFormat.Png);

newBmp = colorReduction.Convert(bmp, PixelFormat.Format1bppIndexed);
newBmp.Save("c:\\Pampero 1.png", ImageFormat.Png);
```

|     |     |     |     |
| --- | --- | --- | --- |
| ![TYPE INFO](https://www.codeproject.com/KB/cs/IconLib/image020.jpg) | ![TYPE INFO](https://www.codeproject.com/KB/cs/IconLib/image015.gif) | ![TYPE INFO](https://www.codeproject.com/KB/cs/IconLib/image016.gif) | ![TYPE INFO](https://www.codeproject.com/KB/cs/IconLib/image017.gif) |
| 24bits RGB 16M Colors | 8bits 256 colors<br><br>Floyd-Steinberg dither | 4bits 16 colors<br><br>Floyd-Steinberg dither | 1bit Black and White<br><br>Floyd-Steinberg dither |

## Extensible ColorProcessing Namespace

For most of the application that use IconLib the ColorProcessing namespace contains all the tools necessary to create a quality icon, but because there are so many algorithm for color reduction then it is implemented with interfaces, this means that the library can be expanded to use different algorithms if it is necessary.

For color reduction there is an interface `IColorQuantizer` and it is implemented for the default class `EuclideanQuantizer`

For palette optimization there is an interface `IPaletteQuantizer` and it is implemented for the default class `OctreeQuantizer`

For dithering there is an interface `IDithering` and it is implemented for the default class `FloydSteinbergDithering`

Any of those interfaces can be implemented and the default can be replaced.

For example, if the developer implemented the noise or random dither algorithm then the color reduction initialization could be something like:

```csharp
IColorQuantizer colorReduction = new EuclideanQuantizer(new OctreeQuantizer(), new NoiseDithering());
```

## Automatic Icon Creation

Even when with a few lines of code IconLib can create an icon with multiple images from a single one, anyway IconLib provideds a special API that will create a full Icon from a single input image.

```csharp
MultiIcon mIcon = new MultiIcon();
SingleIcon sIcon = mIcon.Add("Icon1");
sIcon.CreateFrom("c:\\Clock.png", IconOutputFormat.FromWin95);
```

`CreateFrom` is a method exposed on `SingleIcon` class, this method will take a input image that must be 256x256 pixels and it must be a 32bpp (alpha channel must be included), the perfect candidate for this method are PNG24 images created for PhotoShop or any Image editing software.

The second parameter in the API is a flag enumeration that target the OS which we want to create the icon, in the previous example it will take the input image and it will create the following `IconImage` formats.

```
256x256x32bpp (PNG compression)  
48x48x32bpp  
48x48x8bpp  
48x48x4bpp  
32x32x32bpp  
32x32x8bpp  
16x16x32bpp  
16x16x8bpp
```

There are 14 possible enumerations defined, but they can be combined to get whatever format the developer is looking for.

This method make use of the whole library to provide the best `IconImage` for each format.

## Ohh Microsoft policy about compatibility is changing?

Something I have to comment about because I think it is a breakthrough on how Microsoft usually does things from my point of view.

I have been developing on Windows platform for the last decade from the Windows 3.1 to date, and something that I saw in Microsoft APIs is the amazing compatibility between versions. Personally I think many Win32 APIs are so intrinsic and complicated because they had to keep backward compatibility, and I had so many headaches in the last years because of it.

For example the huge show stopper for Windows future generation was the GDI that imposed a set of rules that could not be broken in any way, GDI+ helped but still ran under the GDI rules, and that is the reason why there are things that Windows could never do until now.

This happened when I decided to implement Windows Vista icons support.

I read that Windows Vista icons are 256x256 and they use PNG compression for them.

At first I was 100% convinced they were going to keep backward compatibility, so I started to think how they did it. The first thing that came to my mind was that Microsoft boys were going to use the field `biCompression` in the header `BITMAPINFOHEADER`and instead set to `BI_RGB` (BMP). They were going to use `BI_PNG`(PNG) that is already supported in the header, the palette was going to be empty and the `XOR` and `AND` Image would contain the PNG data.

I was surprised when that didn't happen. Instead they completely dropped the concept of having a `BITMAPINFOHEADER`, the image (`XOR`) and the mask (`AND`). Instead, the icon directory pointed to a 100% PNG structure.

At first I thought, 'oh my God what have they done!'

This was going to break all Icons Editors out there, also Visual Studio and Resource Editors won't be able to open ICO files anymore, but when I sat and thought about it, it occurred to me that it was the right way to go.

Developers have always complained about how complicated some Win32 APIs are, and this time Microsoft heard that and did things right.

If they could have kept compatibility, it would mean that now ICO and ICON libraries could have 3 places with redundant information about each image.

Like `ICONDIRENTRY`, `BITMAPINFOHEADER`, and `PNGHEADER` usually find those bizarre things in Win32 API.

Instead now they have the Icon directory entry that points to the image itself. That way, they open the way for future implementation for different images or compressions. Still ICO files are limited by a maximum of 256x256 pixels because the Icon directory stores width and height in two byte type fields `bWidth`and `bHeight`. Probably that can be resolved using more than one Plane. But anyway still we are far from use ICONS with more than 256x256 pixels.

So this time I congratulate the boys at Microsoft for thinking "what is the best way to do it" over anything else.

If you wonder if this means VS2005 or any VS won't be able to open properly ICO or DLLs from Windows Vista, then you are right, it WON'T. I also tested ORCAS (VS2006) and it doesn't support it. But that can be easily resolved with a VS patch that hopefully will come out soon, else you will have a product like this library that will support Windows Vista Icons.

## Roadmap

`IconLib` is a powerful library to allow icons or icon libraries creation and modifications. I plan to support updates for DLLs and EXE in the next version, allowing to replace/add/delete icons inside them.

`IconLib` alone is only useful from a programming language. So, I also plan to create an advance Icon Editor application to make full use of `IconLib`. That will probably be my next article in the next few months.

If I can get file formats like .icc (Icons collection), Icns, RSC, bin (mac), I'll support them. If you know of some file format and you have the internal file structure, let me know and I'll try it to implement it.

If someone is interested in creating an open-source Icon Extractor & Editor, then he or she is welcome to use `IconLib` as the file formats engine and I can provide support for `IconLib`.

## License

[![somerights20](https://www.codeproject.com/KB/cs/IconLib/image021.gif)](http://creativecommons.org/licenses/by-sa/3.0/)

This work is licensed under a [Creative Commons Attribution-Share Alike 3.0 Unported License.](http://creativecommons.org/licenses/by-sa/3.0/)

## References

-   [Icons in Win32](http://msdn2.microsoft.com/en-us/library/ms997538.aspx)[](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/dnwui/html/msdn_icons.asp)
-   [Wikipedia Icon Image File Format](http://en.wikipedia.org/wiki/ICO_\(icon_image_file_format\))
-   [New-style EXE Format](http://www.itee.uq.edu.au/~csmweb/decompilation/new_exe.html)
-   [Executable-File Header Format](http://support.microsoft.com/default.aspx?scid=kb;EN-US;q65122)
-   [Microsoft Portable Executable](http://www.microsoft.com/whdc/system/platform/firmware/PECOFF.mspx)
-   [Portable Executable](http://en.wikipedia.org/wiki/Portable_Executable)
-   [Floyd-Steinberg dithering](http://en.wikipedia.org/wiki/Floyd-Steinberg_dithering)
-   [Floyd-Steinberg dithering source](http://en.literateprograms.org/Floyd-Steinberg_dithering_\(C\))
-   [Octree structure](http://en.wikipedia.org/wiki/Octree)
-   [Octree Quantizer implementation](http://www.microsoft.com/msj/archive/S3F1.aspx)

## License

This article, along with any associated source code and files, is licensed under [The Creative Commons Attribution-ShareAlike 2.5 License](http://creativecommons.org/licenses/by-sa/2.5/)

## Comments and Discussions

#####  Packed EXE using UPX throwing "Format not recognized by IconLib" 
[GeorgeMorgan](https://www.codeproject.com/script/Membership/View.aspx?mid=16097732) | 21-Sep-23 10:37 |

Any fix would be appreciated.  
Thanks.  

##### Current Github project of MultiIcon 
[harbor](https://www.codeproject.com/script/Membership/View.aspx?mid=4856575) | 1-Apr-23 12:31 |

The current corrected version is here: [](https://www.codeproject.com/Articles/16178/IconLib-Icons-Unfolded-MultiIcon-and-Windows-Vista?display=PrintAll)[https://github.com/harborsiem/IconLib](https://github.com/harborsiem/IconLib)  

##### My vote is 5 of 5 
[Member 14120945](https://www.codeproject.com/script/Membership/View.aspx?mid=14120945) | 23-Oct-20 7:17 |

It's a really great job! ![Thumbs Up | :thumbsup:](https://www.codeproject.com/script/Forums/Images/thumbs_up.gif)  

##### Fix for memory leak in IconImage.Transparent property 
[Magnus Madsen](https://www.codeproject.com/script/Membership/View.aspx?mid=9684807) | 10-Jul-19 9:15 |

I discovered a memory/object leak (after successfully using the library for years, no less!). It is possible that it only actually presents an issue on some versions of .NET and furthermore it usually isn't an issue unless you are processing a lot of icons, but it is still present.  
  
The error is in `IconImage.cs` on line 106:  
  
```csharp
public unsafe Bitmap Transparent
{
    get {return Icon.ToBitmap();}
}
```
  
  
The issue here is that the Icon variable is dynamically generated and so isn't disposed when the Transparent property is accessed. The .NET GC does not seem to be able/want to clean them up.  
  
A simple addition of a using statement fixes it:  
  
```csharp
public Bitmap Transparent
{
    get
    {
        using (Icon icon = Icon)
            return icon.ToBitmap();
    }
}
```
  

#####  .NET 4.5 Upgrade Issues 
[Tide](https://www.codeproject.com/script/Membership/View.aspx?mid=5138624) | 30-Dec-17 18:16 |

I am having a problem after upgrading to .NET 4.5 or higher. MultiIcon.Load is not returning anything for DLL files.  

#####  My vote of 5 
[Mike Angel Martin](https://www.codeproject.com/script/Membership/View.aspx?mid=99672) | 2-Oct-17 10:40 |

Excellent !!!  

#####  Any plans to support .msi format? 
[Member 13063306](https://www.codeproject.com/script/Membership/View.aspx?mid=13063306) | 16-Mar-17 8:31 |

Great work! Only question I have is are you going to be providing support to extract icons from .msi files?  

#####  How to save an ico file with images from several pictureboxes? 
[**Mc Gwyn**](https://www.codeproject.com/script/Membership/View.aspx?mid=325552) | 16-Feb-17 7:11 |

Hello,  
  
first of all - really great work.  
I've the following question: How can I save a MultiIcon (.ico) which is based of several pictureboxes (like pictureBox256, pictureBox64, aso.)?  
Your guide is not clarifying that to me.  
  
Regards  
Tim  

##### Great library 
[StoneFactory](https://www.codeproject.com/script/Membership/View.aspx?mid=29425) | 4-Jul-16 10:44 |

Using your library, I was able to easily extract each image icon formats from existing exe file in 3 files of code. And each image is very well handled. Good work!  
  
Only a little drawback: I've not been able to use IconLib in a 64 bits process (crashes in Win32.EnumResourceNames).  

##### Bug? 
[gyrtenudre](https://www.codeproject.com/script/Membership/View.aspx?mid=5840492) | 15-Dec-15 3:46 |

First off, great work there, thanks!  
  
I had an issue with BMPs, when saving them I would get some weird result double the original height of the icon and I think I've found a bug. I think line 211 of IconImage.cs should read  
  
```csharp
switch (mEncoder.IconImageFormat)
{
    case IconImageFormat.BMP:
        iconDirEntry.bHeight = (byte)(mEncoder.Header.biHeight / 2);
        break;
    default:
        iconDirEntry.bHeight = (byte)mEncoder.Header.biHeight;
        break;
}
```  
  
instead of just  
  
```csharp
iconDirEntry.bHeight = (byte)mEncoder.Header.biHeight;
```


Is this correct or would it break something else?  
Thanks a lot  

##### [Re: Bug?](https://www.codeproject.com/Messages/5432920/Re-Bug) 
[pcbbc](https://www.codeproject.com/script/Membership/View.aspx?mid=1324664) | 8-Sep-17 8:39 |


The same height encoding bug is mentioned and fixed in [earlier post.](https://www.codeproject.com/Messages/3435541/Height-Encoding-Bug.aspx)  
  
The height in the ICONDIRENTRY structure is incorrectly being doubled for PNG as well as BMP, so I think the fix is required regardless of IconImageFormat? So the following change, as in the earlier post, will suffice:  

```csharp
iconDirEntry.bHeight = (byte)(mEncoder.Header.biHeight / 2);
```
  
Also it will need fixing in following GRPICONDIRENTRY code as well:  

C#
```csharp
groupIconDirEntry.bHeight = (byte) (mEncoder.Header.biHeight / 2);
```
  
The issue is that the BITMAPINFOHEADER structure has the height correctly doubled according to the spec. That's because the bitmap contains both the AND and XOR masks sequentially one after another. However the ICONDIRENTRY always contains the actual icon height.  

#####  Visual Studio 2012 
[paolo guccini](https://www.codeproject.com/script/Membership/View.aspx?mid=7645220) | 15-Jun-15 6:27 |

After conversion to Visual Studio 2015:  
Error of missing "Resource"  
Solution: to the file "PEFormat.cs" add instruction:  
using IconLib.System.Drawing.IconLib;  

#####  Nice 
[NikolaB](https://www.codeproject.com/script/Membership/View.aspx?mid=2319350) | 21-Apr-15 6:03 |

Very well described everything. Used comfortable. I can clarify the simplest way to use.  

##### My vote of 5 
[RedDk](https://www.codeproject.com/script/Membership/View.aspx?mid=660010) | 16-Apr-15 12:39 |

Very fine CodeProject Article ... will recommend and will really have to study the source. Thank-you!  

#####  very nice project 
[zhouhu](https://www.codeproject.com/script/Membership/View.aspx?mid=2192324) | 25-Dec-14 19:34 |

thanks you very much, it 's very useful  

##### My vote of 5 
[WLDNA](https://www.codeproject.com/script/Membership/View.aspx?mid=7724108) | 11-Aug-14 21:11 |

Good!  

#####  Can we get some documentation? 
[Will Pittenger](https://www.codeproject.com/script/Membership/View.aspx?mid=10965944) | 23-Jul-14 7:20 |

How it works is nice, but we could use "how to use it" too.  

##### My vote of 5 
[liruikuan](https://www.codeproject.com/script/Membership/View.aspx?mid=4423961) | 2-Jul-13 2:39 |

very good!  

##### My vote is 5 of 5
[Lil' D 'mBoss](https://www.codeproject.com/script/Membership/View.aspx?mid=6471674) | 25-May-13 13:06 |

The best!  

##### My vote of 5
[Tapirro](https://www.codeproject.com/script/Membership/View.aspx?mid=9565883) | 6-Feb-13 5:59 |

Respect and thx  

#####  Will Saving to a DLL be possible?
[lucaso](https://www.codeproject.com/script/Membership/View.aspx?mid=2659855) | 8-Dec-12 17:01 |

Cool Work. But.... Trying to have "ShortCuts" use homemade icons is a bit further. If one could update a persistent "empty.dll", kept in the same directory as where the shortcuts (".lnk") are, one could even have "snapshots" of files that are not supported by Win, for example PDF's, as a bit "shaky" replacement of "thumbs.db".  

##### My vote of 5
[malparry](https://www.codeproject.com/script/Membership/View.aspx?mid=3204964) | 17-Sep-12 0:22 |

Wow just spent hours trying to open my old icl libs with No success in Win7 nothing worked and I came across this app you have done tried it and hey presto there were my icons. I tried some commerical app trials and open source all failed except yours well done. Possibly one of the most useful little utilities I have ever come across. Definity should be part of Win framework.  

##### My vote of 5
[fenriv](https://www.codeproject.com/script/Membership/View.aspx?mid=7023269) | 13-Aug-12 7:26 |

Cool thing. Should be a part of framework.  

#####  License - CC-BY-SA
[CyberKnet](https://www.codeproject.com/script/Membership/View.aspx?mid=1039882) | 25-Jul-12 8:58 |

If we use your work in our programs - who do we give attribution to? This is a really large amount of work you've done here, and we have no problem letting people know that such an important piece of our program was developed by you! Feel free to let me know via email - s _dot_ blomfield -at- cox /dot/ net  
  
Video meliora, proboque; Deteriora sequor  

##### FT Icon Studio 2012
[fireHLF](https://www.codeproject.com/script/Membership/View.aspx?mid=5373203) | 1-Apr-12 5:02 |

First of all - very nice Lib!  
  
I've integrated this library into my programm [http://ftwebsite.de/downloads.php?Download=iconstudio2012](http://ftwebsite.de/downloads.php?Download=iconstudio2012)[[^](http://ftwebsite.de/downloads.php?Download=iconstudio2012 "New Window")] and must say: Very easy to use! Thanks! ![Thumbs Up | :thumbsup:](https://www.codeproject.com/script/Forums/Images/thumbs_up.gif)  

#####  IE8 or IE7 can't Icon files exported from IconLib
[CPMJim](https://www.codeproject.com/script/Membership/View.aspx?mid=7531347) | 25-Mar-12 21:15 |

I tried to open icon files exported from IconLib in IE7/IE8, but IE just shows the icon image as a red cross. But Firefox or Chrome can show the same icon file. After I opened the icon file in Visual Studio 2008 and make some changes, then save back to the same file. IE8/IE7 can show the icon file correctly. Why can't IE7/8 (IE9 works) show the file exported from IconLib 0.73 correctly?  
Thank you,  
Jim  

##### My vote of 5
[elea30](https://www.codeproject.com/script/Membership/View.aspx?mid=1141819) | 29-Dec-11 7:58 |

Excellent!  

##### My vote of 4
[leochow](https://www.codeproject.com/script/Membership/View.aspx?mid=7455784) | 7-Dec-11 1:22 |

very nice  

##### Error with .bmp; works somewhat with.png
[Member 8246511](https://www.codeproject.com/script/Membership/View.aspx?mid=8246511) | 29-Nov-11 22:00 |

I couldn't get a 256x256 32-bit Alpha channel .bmp to generate an icon. I double checked the RGB enumerator (0) and the size of the file (256*256*4+54=262198K). However when I converted it to .png, which seems like a much more feasible size for an icon (in this case 12508K->12530K), it worked fine, but only generated an .ico file consisting of one icon instead of seven. I plugged the icon into my app anyways and the single icon worked fine at various sizes. I used the hidden test <button1> in your source code to do all the work. Excellent research. I couldn't get my icons to work using MFC building blocks. I'm using VS10/SP1 on Windows 7. Thanks.  

##### My vote of 5
[ZotovBST](https://www.codeproject.com/script/Membership/View.aspx?mid=6821330) | 27-Sep-11 14:22 |

Demo app is exactly what I needed to extract icons.  

##### A Suggestion | ![mve](https://codeproject.freetls.fastly.net/App_Themes/CodeProject/Img/icn-MVE-16.gif "Most Valuable Expert") | [Rick York](https://www.codeproject.com/script/Membership/View.aspx?mid=196) | 6-Sep-11 16:20 |

Very good work !  
  
I have one suggestion: I would like to have the first entry in the icon list selected when a file selection is made or changed and the first entry in the images list to be selected when an icon selection is made or changed. As it is, it takes at least two clicks to see an icon. By automatically selecting these entries you could instantly see an image. This would make the program much easier to use when exploring icons in files.  
  
Thanks  

##### My vote of 5
[Filip D'haene](https://www.codeproject.com/script/Membership/View.aspx?mid=5174756) | 24-May-11 11:38 |

Excellent article!  
  
Thanks for sharing. ![Wink | ;)](https://codeproject.freetls.fastly.net/script/Forums/Images/smiley_wink.gif)  

##### My vote of 5
[Denis1](https://www.codeproject.com/script/Membership/View.aspx?mid=325570) | 20-May-11 6:47 |

clearly, well explained  

##### Bug report!
[faculacola](https://www.codeproject.com/script/Membership/View.aspx?mid=5825419) | 4-May-11 17:59 |

excellent work! thanks the author! Well, im need to extract an icon from netshell.dll(in system32 directory) with the ID 0, but the application told me "Format not recognized by IconLib"  
bug in PEFormat.cs :  

``
public unsafe MultiIcon Load(Stream stream)
{
    ....
    if (Win32.IS_INTRESOURCE(id))
        hRsrc = Win32.FindResource(hLib, int.Parse(id), (IntPtr) ResourceType.RT_GROUP_ICON);
    else
        hRsrc = Win32.FindResource(hLib, id, (IntPtr) ResourceType.RT_GROUP_ICON);

    if (hRsrc == IntPtr.Zero)
        throw new InvalidFileException();
    ...
```
  
  
so im fixed it with this:

```  
hRsrc = Win32.FindResource(hLib, id, (IntPtr) ResourceType.RT_GROUP_ICON);  

public unsafe MultiIcon Load(Stream stream)
{
    ....
    hRsrc = Win32.FindResource(hLib, id, (IntPtr) ResourceType.RT_GROUP_ICON);
    if (hRsrc == IntPtr.Zero)
    {
        if (Win32.IS_INTRESOURCE(id))
            hRsrc = Win32.FindResource(hLib, int.Parse(id), (IntPtr) ResourceType.RT_GROUP_ICON);
    }

    if (hRsrc == IntPtr.Zero)
        throw new InvalidFileException();
    ...
```
  
  
and it works correct! ![Laugh | :laugh:](https://codeproject.freetls.fastly.net/script/Forums/Images/smiley_laugh.gif)  

##### My vote of 5
[asugix](https://www.codeproject.com/script/Membership/View.aspx?mid=4506266) | 18-Mar-11 10:36 |

Useful lib  

##### Good Job
[Nueman](https://www.codeproject.com/script/Membership/View.aspx?mid=7353076) | 9-Feb-11 21:46 |

Thanks  

##### My vote of 5
[steve stokes](https://www.codeproject.com/script/Membership/View.aspx?mid=387641) | 24-Jan-11 10:33 |

Well written, incredible detail - all in once place! And source!!  

##### My vote of 5
[einismotic](https://www.codeproject.com/script/Membership/View.aspx?mid=5977112) | 28-Dec-10 9:19 |

The article and the program are like a whole encyclopaedia!  

##### Version .73 Can we get Version .74 [modified]
[Diamonddrake](https://www.codeproject.com/script/Membership/View.aspx?mid=3876069) | 15-Dec-10 16:23 |

Somewhere in the forum it says that there is a new version .74 Where can we find this? I guess it really doesn't matter. What i needed this for is there should be an application that creates icons super simple. your lib made it easy. I wrote a little app that accepts a png image path as a argument and throws up a save file dialog box and creates an XP icon. That's it. I added a resistry entry for .png images and now, after I make an icon in photoshop, I just rightclick on the png and click "Create XP Icon" Choose where to save it, and that's it. It just works.  
  
Thanks for sharing this lib. I can't believe I never found it til now.  
  

##### My vote of 4
[Sujith S B](https://www.codeproject.com/script/Membership/View.aspx?mid=6940051) | 9-Oct-10 2:18 |

Its really a good work, it will be great if providing support to MAC icons too  

##### My vote of 5
[Violet Tape](https://www.codeproject.com/script/Membership/View.aspx?mid=6050841) | 20-Sep-10 10:24 |

Easy to use and excelent result!  

##### My vote of 5
[Tukito](https://www.codeproject.com/script/Membership/View.aspx?mid=6838887) | 13-Aug-10 13:41 |

Excellent article!  
Only do I ask myself as he can add resources of information on the compiled Dll file?  

##### Height Encoding Bug
[.armin](https://www.codeproject.com/script/Membership/View.aspx?mid=2934008) | 12-Apr-10 7:02 |

When I saved Icons with the IconLib, in Win7 the wrong Icons were selected for display.  
It seems that height values are wrongly encoded, the patch below worked for me:  
  

```
--- System/Drawing/IconLib/IconImage.cs (revision 1697)
+++ System/Drawing/IconLib/IconImage.cs (working copy)
@@ -208,8 +208,10 @@
             {
                 ICONDIRENTRY iconDirEntry;
                 iconDirEntry.bColorCount    = (byte) mEncoder.Header.biClrUsed;
-                iconDirEntry.bHeight        = (byte) mEncoder.Header.biHeight;
-                iconDirEntry.bReserved      = 0;
+                // iconDirEntry.bHeight        = (byte) mEncoder.Header.biHeight;
+                               // armin: biHeight is always *2
+                               iconDirEntry.bHeight        = (byte) (mEncoder.Header.biHeight/2);
+                               iconDirEntry.bReserved = 0;
                 iconDirEntry.bWidth         = (byte) mEncoder.Header.biWidth;
                 iconDirEntry.dwBytesInRes   = (uint) (sizeof(BITMAPINFOHEADER) +
                                                 sizeof(RGBQUAD) * ColorsInPalette +
```

  

##### [Re: Height Encoding Bug](https://www.codeproject.com/Messages/4213906/Re-Height-Encoding-Bug)
[Florian Thurnwald](https://www.codeproject.com/script/Membership/View.aspx?mid=8720905) | 6-Apr-12 6:04 |


Thanks for this Patch! Works great ![Thumbs Up | :thumbsup:](https://www.codeproject.com/script/Forums/Images/thumbs_up.gif)  

##### so good!
[KoaQiu](https://www.codeproject.com/script/Membership/View.aspx?mid=2081563) | 16-Mar-10 8:56 |

**so good!**  

#####  How to use in Visual Basic?
[m477h35](https://www.codeproject.com/script/Membership/View.aspx?mid=6972764) | 23-Feb-10 12:09 |

Hi!  
  
I would like to use the IconLib in my Visual Basic 08 project, but I don´t know how. Is it possible to use a C#-Lib in a Visual Basic project?  
  

#### [Re: How to use in Visual Basic?](https://www.codeproject.com/Messages/3588255/Re-How-to-use-in-Visual-Basic)
[lipinho](https://www.codeproject.com/script/Membership/View.aspx?mid=6435577) | 4-Sep-10 20:05 |


Yup, it is possible. Just add the .dll as a Reference to your project, and convert the code you need from the app example to VB.net. If you need a code example using VB, ask for me and I'll be glad to help you ![Big Grin | :-D](https://www.codeproject.com/script/Forums/Images/smiley_biggrin.gif)  

##### Excellent work
[Aybe](https://www.codeproject.com/script/Membership/View.aspx?mid=2825266) | 23-Jan-10 17:30 |

Great job,  
  
When I draw my icons, they have jagged edges, do you know why ?  
  
Thank you, ![Big Grin | :-D](https://www.codeproject.com/script/Forums/Images/smiley_biggrin.gif)  


#####  Nice article,but one question
[Satchitananda](https://www.codeproject.com/script/Membership/View.aspx?mid=3785248) | 24-Nov-09 22:26 |

Is it possible to create a High res icon not from .ico file, but Icon Handle(HICON)?  
Very actual for me now!  

#####  Requires Full trust mode when running in ASP.NET
[Kobi Pinhasov](https://www.codeproject.com/script/Membership/View.aspx?mid=315027) | 3-Nov-09 15:23 |

In my opinion, this is the best icon extractor library out there. Very simple to use with absolutely powerful options. I was going to use it in my ASP.NET application and it works well on my development PC. However when I've deployed it to production site I always received error message "PolicyException: Required permissions cannot be acquired". In short this DLL (IconLib) requires a Full trust but unfortunately most of the hosting companies (shared hosting) configure asp.net for Medium trust. Because of this I cannot use it now. It requires the Full trust because it has a lot of variables defined with **unsafe** keyword.  
  
Is there any way to modify this code so it could run under at least Medium trust ?  
I have already started doing it but I wanted to know the author's opinion.  

##### Auto select first icon in ListBox
[Paresh B Joshi](https://www.codeproject.com/script/Membership/View.aspx?mid=3425939) | 8-Oct-09 13:53 |

I appreciate your hard work and R&D on icons.  
I run exe it looks great.  
I would like to propose one functionality to automatically select first icon in lbxIcons (Listbox)  
So that first icon is displayed by default if there are icons in list.  


##### [Re: Auto select first icon in ListBox](https://www.codeproject.com/Messages/3588254/Re-Auto-select-first-icon-in-ListBox)
[lipinho](https://www.codeproject.com/script/Membership/View.aspx?mid=6435577) | 4-Sep-10 20:03 |


You can do it by settings Multiicon.selectedindex = 0  

##### [Re: Auto select first icon in ListBox](https://www.codeproject.com/Messages/4015928/Re-Auto-select-first-icon-in-ListBox) | ![mve](https://codeproject.freetls.fastly.net/App_Themes/CodeProject/Img/icn-MVE-16.gif "Most Valuable Expert") | [Rick York](https://www.codeproject.com/script/Membership/View.aspx?mid=196) | 6-Sep-11 16:08 |


I agree - I like this idea. I would also like to have the first entry in the images list to be selected. As it is, it takes at least two clicks to see an icon. By automatically selecting entries you could instantly see an image. This would make the program much easier to use when exploring icons in files.  

##### Problem with Vista and Windows 7
[Lee Whitney](https://www.codeproject.com/script/Membership/View.aspx?mid=6590749) | 29-Sep-09 21:52 |

There are many cases where the library doesn't work on Vista and Win7. The scenario is accessing icons via registry info.  
  
It's common for the registry to tell where an icon is located by providing the file and id of an icon. However sometimes the .dll given has another "assocaited" .dll that actually contains the icon (see comments from seguso).  
  
You might ask, is it the job of a library to track down the proper associated dll for the icon? I would say yes since Visual Studio 2008 does the whole process correctly showing all associated icons.  
  
Example:  
- The registry says that the icon for the Control Panel applet "CardSpace" is located in the file C:\Windows\System32\icardres.dll  
- IconLib can't find any icon in this file, but Visual Studio correctly displays a 256x256 icon when this .DLL is loaded  
- The registry key example is HKEY_CLASSES_ROOT\CLSID\{78CB147A-98EA-4AA6-B0DF-C8681F69341C}\DefaultIcon  
  
  
  
>>seguso 20:06 23 Aug '09  
>>I have the path of a file and I need to get its 256x256 icon (Vista only). I think I need a function like SHGetFileInfo or ExtractAssociatedIcon. In other words, I don't need to >>extract an icon from a given DLL file, but a function which returns the 256x256 icon associated to a given file extension. Can this library do what I need? Thanks Wink  

##### [Re: Problem with Vista and Windows 7](https://www.codeproject.com/Messages/3598461/Re-Problem-with-Vista-and-Windows-7)
[JesseChisholm](https://www.codeproject.com/script/Membership/View.aspx?mid=110750) | 14-Sep-10 16:32 |


The "Associated" resource libraries are for Localization.  
If you have loaded

```
filename.dll
```
  
then for .NET resources, look in

```
en-US\filename.resources.dll
```
  
and for Win32 resources, look in

```
en-US\filename.dll.mui
```
  
  
The **en-US** is, of course, _my_ local **language-Culture**.  
  
The actual value comes from

System.Globalization.CultureInfo.CurrentUICulture.Name


##### Vista x64/Win7 x64: Generic error in GDI+ while trying to Add icon
[koldovsky](https://www.codeproject.com/script/Membership/View.aspx?mid=3276111) | 9-Sep-09 4:32 |

My code requests icon of running application, shows it with PictureBox and allows to Save to a file.  
I am using IconLib to perform file saving procedure, my code for this is below (I think, it is self-explanatory):  
  
```
MultiIcon mico = new MultiIcon();
SingleIcon sico = mico.Add("Default");
sico.Add(Win.Icon); 
sico.Save(sd.FileName);
```

It works well on 32-bit systems (XP/Vista/Win7) but on x64 (I tested Win7 and Vista only) it raises strange exception "A generic error occurred in GDI+".  
This error raises next line of code in SingleIcon.Add() method:  
  
`XORImage = Bitmap.FromHbitmap(iconInfo.hbmColor);`  
  
I suppose, there is something wrong with return result of Win32.GetIconInfo() and iconInfo.hbmColor is wrong somehow. Though, I can't find any reasonable info on this: there are some cases described on the Internet where people were stuck with this error, but I did not find any solution.  

##### Add support for .cur
[dB.](https://www.codeproject.com/script/Membership/View.aspx?mid=913212) | 24-Aug-09 23:37 |

Very good.  
  
You're missing support for RT_CURSOR, .cur files, which are essentially identical to .ico files but with 4 bytes of hotspot information on top of the image data. Just dealt with this for [http://resourcelib.codeplex.com](http://resourcelib.codeplex.com/) if you need to look at working code.  
  
While everything works for me (I can read and write cursors to .dll/.exe-s), I am stuck with cursors that report the wrong size (32x0), maybe you'll understand more about that format and can explain this issue.  
  

dB. - [dblock.org](http://www.dblock.org/)

  

##### Getting 256x256 icon from path?
[seguso](https://www.codeproject.com/script/Membership/View.aspx?mid=4667769) | 23-Aug-09 19:06 |

I have the path of a file and I need to get its 256x256 icon (Vista only). I think I need a function like SHGetFileInfo or ExtractAssociatedIcon. In other words, I don't need to extract an icon from a given DLL file, but a function which returns the 256x256 icon associated to a given file extension. Can this library do what I need? Thanks ![Wink | ;)](https://www.codeproject.com/script/Forums/Images/smiley_wink.gif)  

##### [Re: Getting 256x256 icon from path?](https://www.codeproject.com/Messages/3588253/Re-Getting-256x256-icon-from-path)
[lipinho](https://www.codeproject.com/script/Membership/View.aspx?mid=6435577) | 4-Sep-10 20:02 |


Try this:  

```
Private mmultiicon As New IconLib.MultiIcon()
Public Function loadicons(ByVal path As String) As Image
        If path.Contains(",") Then
            path = path.Split(",")(0)
        End If
        Try
            mmultiicon.Load(path)
            mmultiicon.SelectedIndex = 0
            Dim workimage As IconLib.IconImage = Nothing
            For Each iconImage As IconLib.IconImage In mmultiicon(0)
                If iconImage.Size.Width >= 48 AndAlso iconImage.Size.Height >= 48 AndAlso iconImage.PixelFormat = Imaging.PixelFormat.Format32bppArgb Then
                    If iconImage.Size.Width = 256 Then
                        workimage = iconImage
                        Exit For
                    ElseIf workimage Is Nothing Then
                        workimage = iconImage
                    ElseIf iconImage.Size.Width > workimage.Size.Width Then
                        workimage = iconImage
                    End If
                End If
            Next
            If Not workimage Is Nothing Then
                Return resizeicon(workimage.Icon.ToBitmap)
                Exit Function
            Else
                Return nothing 'here you can just return anyother image you want;
            End If
        Catch ex As Exception
            Return resizeicon(Icon.ExtractAssociatedIcon(path).ToBitmap)
        End Try
    End Function
```

  
This function will return the largest and heightest image found on a given .dll OR .EXE file. This is just an example.  

##### Exception Error: Image with same size and format already exists
[Dukov](https://www.codeproject.com/script/Membership/View.aspx?mid=994353) | 6-Aug-09 9:24 |

Dear CastorTiu,  
Really good work!  
I'm trying to use your library to add icons to a new library from PNG files.  
The code I wrote is :  

```
MultiIcon mIcon = new MultiIcon();  
SingleIcon sIcon1 = mIcon.Add("Icon1");  
SingleIcon sIcon2 = mIcon.Add("Icon2");  
SingleIcon sIcon3 = mIcon.Add("Icon3");  
  
sIcon1.CreateFrom("d:\\ferrari.png", IconOutputFormat.FromWin95);  
sIcon2.CreateFrom("d:\\ducati.png", IconOutputFormat.FromWin95);  
sIcon3.CreateFrom("d:\\fiat.png", IconOutputFormat.FromWin95);  
  
sIcon1.Save("d:\\output1.ico");  
sIcon2.Save("d:\\output2.ico");  
sIcon3.Save("d:\\output3.ico");  
  
mIcon.Add("Icon 1111").Load("d:\\output1.ico");  
mIcon.Add("Icon 2222").Load("d:\\output2.ico");  
mIcon.Add("Icon 3333").Load("d:\\output3.ico");  
  
mIcon.Save("d:\\library.icl", MultiIconFormat.ICL);  
```
  
If I use IconOutPutFormat.Vista The output is correct just for the first Icon "ferrari" the others are wrong. If I use IconOutPutFormat.FromWin95 the error is on line 325 of SingleIcon.cs "if (bitmap.Width > 256 || bitmap.Height > 256)" Unhandled Exception..Image with same size and format already exist  
  
Could you help me, please ? I would like to use your iconLib to create Vista like icons from png sources image and store alls in a ICL file.  
 

##### [Re: Exception Error: Image with same size and format already exists](https://www.codeproject.com/Messages/3588259/Re-Exception-Error-Image-with-same-size-and-format)
[lipinho](https://www.codeproject.com/script/Membership/View.aspx?mid=6435577) | 4-Sep-10 20:08 |


I think that you can only add one 256 sized image for 'icon folder' inside the .exe or .dll file. If you open any kind of application, you'll see that none of them has more than 1 256 icon per folder. Try to create a new "folder" to add your icon.  

##### [Re: Exception Error: Image with same size and format already exists](https://www.codeproject.com/Messages/3598464/Re-Exception-Error-Image-with-same-size-and-format)
[JesseChisholm](https://www.codeproject.com/script/Membership/View.aspx?mid=110750) | 14-Sep-10 16:39 |


**There can be only one** of ANY size depth combination.  
  
So if your PNG happens to be 48x48, then that slot is filled by the initial call:  
```
Add(bitmap)
```
  
The exception happens just a bit later, when this is done:  

```
bmp = new Bitmap(bitmap,48,48);
Add(bmp);
```

Code will have to be added to check for this case (for each size depth combination) and avoid the exception, or a try...catch to ignore it.  
  
More code the first way, uglier the second way.  
  

##### Holy freakin crap!
[DiscoJimmy](https://www.codeproject.com/script/Membership/View.aspx?mid=598284) | 16-Jul-09 15:07 |

excellent work man. a very thorough venture into a poorly documented area of windows tech. well done!  

##### Simply Superb 
[Kumar Shanmugam](https://www.codeproject.com/script/Membership/View.aspx?mid=1875746) | 18-May-09 1:11 |

Simply Superb  

##### rnID in TNAMEINFO exceeding 0xFF [modified]
[jennico](https://www.codeproject.com/script/Membership/View.aspx?mid=6132180) | 7-May-09 20:36 |

thank you for your outstanding, brilliant work.  
  
i am having a problem with extracting large icl files that contain more than 256 single resources (images/bitmaps).  
  
[solved, my mistake]  
  
##### Your are the men!!!
[Vider](https://www.codeproject.com/script/Membership/View.aspx?mid=598017) | 2-Apr-09 2:59 |

Hello friend, very thanks for share your code!! this is exactly what I was looking for, and it's the only one I found, (and is very good coded ;O)


##### Nice Article! & Question to retrieve icon names from a .ICL in a PE Format [modified]
[bxb](https://www.codeproject.com/script/Membership/View.aspx?mid=62149) | 31-Mar-09 7:16 |

Great job! With your article I can now easily extract icons!  
But I have some .ICL libraries that are stored in PE format, and currently PEFormat defines icon names by using the storage index.  
With ResHacker I've noticed there exists an unknown resource type "301" with a string into:  
http://i44.tinypic.com/2zixwxz.png  
This string contains all the icons names, but I didn't found how to retrieve it (maybe with the API EnumResourceNames?)  
  
When I open this ICL with "IconWorkshop" the proper icons names are displayed.  
  
Any way to handle this in PEFormat?
  

##### Simply Awesome
[Michael A. McCloskey](https://www.codeproject.com/script/Membership/View.aspx?mid=564301) | 26-Mar-09 14:55 |

Your article is truly a gift. After discovering the .NET Icon class doesn't provide much in the way of discerning what is contained in an ICO file and Graphics.DrawIcon method didn't seem to be very smart about picking the right icon to render based on the destination size, I went looking for something better. I came across the MSDN article "Icons In Win32" that you reference and prepared myself to roll up my sleeves and start parsing. Then I found your article and after reading about your experiences and the problems you solved for all of us I can't begin to express my appreciation. Thank YOU! I wish I could give your article a 10.  

##### Thank you
[amgadhs](https://www.codeproject.com/script/Membership/View.aspx?mid=3487490) | 12-Mar-09 7:18 |

Very nice article. Enjoyed reading it.  

Amgad Suliman  
My Blog: [CodeHill](http://codehill.com/)  

  

##### Incredible article and code
[cartfer](https://www.codeproject.com/script/Membership/View.aspx?mid=2669562) | 16-Feb-09 6:30 |

Thank you so much for the detailed explanation as well as the powerful functionality. I learned a lot.  

##### Resource Problem
[lrpauley](https://www.codeproject.com/script/Membership/View.aspx?mid=1375290) | 24-Jan-09 10:06 |

I have Visual Studio 2008. When I open the 2005 project, it attempts to convert it to 2008. There is a code line in the "PEFormat.cs" the does not convert correctly. The line is as follows:  
  
```
byte[] buffer = Resource.EmptyDll;  
```
  
This line throws an error saying the 'Resource cannot be found'. VS2008 Changes the Resource to a folder called Resources (I don't know why), so I changed the line to the following:  
  
```
byte[] buffer = Resources.EmptyDll;  
```
  
and added the DLL back to the Resources for the project, but then get 'EmpyDLL does not exist in the namespace `System.Resources`'.  
  
I am not a C# programmer and am having difficulty fixing this problem. You have any suggestions?  

##### [Re: Resource Problem](https://www.codeproject.com/Messages/4126949/Re-Resource-Problem)
[dparvin](https://www.codeproject.com/script/Membership/View.aspx?mid=2748015) | 12-Jan-12 17:11 |


It looks like Microsoft made some kind of change to the way namespaces work in 2008 and later. I had the same issue with trying to load it into 2010 and found that the fix was to add a using at the top.  
  
I Added:  
  
```csharp
using IconLib.System.Drawing.IconLib;
```

and then it compiled just fine.  

##### 8-bit PNG images [modified]
[Stratagem](https://www.codeproject.com/script/Membership/View.aspx?mid=5859931) | 22-Jan-09 18:30 |

While trying to implement Vista icon support in my program, I found an example at the Axialis site to use for testing called down.ico. It has 3 256x256 PNG icons in addition to the smaller BMPs. Only one of the PNG images is 32-bit RGB/A. The others are 256 and 16 colours.  
  
My implementation chokes on the 8-bit (256 colour) and 4-bit (16 colour) PNG compressed icon images by loading them as 32-bit RGB/A. Both the Axialis editor and another I tried report the colours as 256 and 16 with corresponding palettes.  
  
IconLib seems to have the same problem with these 256 and 16 colour PNGs. I believe the problem is with Bitmap/Image creation from a stream, as loading test PNG files results in a proper PixelFormat in the resulting Bitmap/Image.  
  
EDIT: Scratch the stream theory. I just wrote the 256 colour PNG bits contained in the icon file directly to a separate binary file without changes. The PNG file created is 32bppARGB - both icon editors and GDI+ agree the new PNG file is 32bpp. So...icon image headers and a bit of voodoo? What am I missing?  
  

##### thanks
[mugun](https://www.codeproject.com/script/Membership/View.aspx?mid=237885) | 7-Jan-09 2:43 |

thanks for the wonderful tool bro!  
  

##### lacks of documentation
[asugix](https://www.codeproject.com/script/Membership/View.aspx?mid=4506266) | 13-Nov-08 20:56 |

`Iconlib` is great library. I'm using for converting many freeware icon into PNG images for using in my application (batch converting). But your lib is lacks of documentation. It makes me confuse although I read your example source code. So, please add some documentation, especially IntelliSense documentation ![Roll eyes | :rolleyes:](https://codeproject.global.ssl.fastly.net/script/Forums/Images/smiley_rolleyes.gif)  
  

No one can prevent me to learn something

  

##### [Re: lacks of documentation](https://www.codeproject.com/Messages/3116057/Re-lacks-of-documentation)
[gymbrall](https://www.codeproject.com/script/Membership/View.aspx?mid=1727275) | 11-Jul-09 12:12 |


Irony - noun, see asugix's sig. "No one can prevent me to learn something"  

##### [Re: lacks of documentation](https://www.codeproject.com/Messages/3818034/Re-lacks-of-documentation)
[asugix](https://www.codeproject.com/script/Membership/View.aspx?mid=4506266) | 18-Mar-11 10:34 |


do you have problem with that?  

No one can prevent me to learn something

  

##### Some Microsoft executables, Iconlib does not recognize the format. Why?
[jmartinka](https://www.codeproject.com/script/Membership/View.aspx?mid=5117884) | 31-Oct-08 16:15 |

This has proved very useful. We've run issues sometimes as the iconlib does not recognize the images in some standard shipped Vista programs.  
  
Windows Contacts wab.exe  
found in c:\Program Files\Windows Mail\wab.exe  
  
Tablet PC Input TabTip.exe  
found in c:\Program Files\Microsoft Shared\ink\TabTip.exe  
  
The error message using the demo app pointing to these files says:  
"Format not recognized by IconLib"  
  
Is there something very unique about how images are store in these executables?  
  
Otherwise, very fine library. Thanks for your efforts to keep it up to date on Vista.  

##### Great article
[MedicinalProgramming](https://www.codeproject.com/script/Membership/View.aspx?mid=5638899) | 27-Oct-08 10:52 |

Very impressing article. Especially after you run the demo and actually see the separate icons that each file holds.  
  
1. I opened your code and had no problem converting to a visual studio 2008 project.  
2. I have resharped installed and noticed that you had a few warnings. Code that is never reached and stuff like that. Not very serious, thought you might want to take a look at that.  
3. I suggest you update the properties of your exe when you release it.

http://www.pc-kitchen.com


##### Loading from PNG
[Four13 Designs](https://www.codeproject.com/script/Membership/View.aspx?mid=2703496) | 14-Oct-08 4:47 |

This looks to be exactly what I'm after except for one thing. Is there any chance to add loading from a PNG file to create the ICO?  


##### [Re: Loading from PNG](https://www.codeproject.com/Messages/2764873/Re-Loading-from-PNG)
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 14-Oct-08 17:15 |

Yes, and that's the way to get the best quality for the output icon.  
  
You just input a PNG and to which OS you want to make compatible and IconLib will create all the formats automatically for that specific OS.  
  
IconLib will size reduction and color convertion to all formats needed. The optimal input format is a PNG24 256x256.  
  
I'm not sure if I added that funcionality on version .73 or .74, I was lazy for the last months to upload .74 in CodeProject, let me know if the current version .73 doesn't contain that.  


##### [Re: Loading from PNG](https://www.codeproject.com/Messages/2890008/Re-Loading-from-PNG)
[GZ1234](https://www.codeproject.com/script/Membership/View.aspx?mid=5869578) | 21-Jan-09 10:20 |


Your version .73 as posted does not seem to support it. Can you post .74?  
  
Are there any requirements how large the PNG files have to be? Our source files are rather large.  

##### [Re: Loading from PNG](https://www.codeproject.com/Messages/3207853/Re-Loading-from-PNG) 
[bobfox](https://www.codeproject.com/script/Membership/View.aspx?mid=35081) | 22-Sep-09 18:24 |


Could you please upload .74 with PNG support?  
  
Thanks!  

##### Documentation
[Silvyster](https://www.codeproject.com/script/Membership/View.aspx?mid=3371692) | 16-Sep-08 20:07 |

I need Documentation API for this.. i mean if it's going to be open source we need documentation on how to use it...  

##### Clicking on Icons - preview
[fweinrebe](https://www.codeproject.com/script/Membership/View.aspx?mid=1115417) | 15-Sep-08 2:07 |

Nice tool. !!  
  
I find it a bit irritating if I click on an item in the Icons list and then have to click in the Images list to see what the icon looks like. I suggest that if one click in the Icons list, automatically selects the first item in the images list. That way I can easily see which icon I am looking at before drilling down in the size of the icons.  

##### [[Message Removed]](https://www.codeproject.com/Messages/2732648/Message-Removed)
[nompel](https://www.codeproject.com/script/Membership/View.aspx?mid=5536352) | 20-Sep-08 20:04 |


Spam message removed  

##### wow 
[Abhishek Sur](https://www.codeproject.com/script/Membership/View.aspx?mid=4293807) | 2-Sep-08 15:26 |

What an Article... Very good for references...  
  
Thanks for your great article CastorTui  
  

##### Message Closed
[Jose Maria Estrade](https://www.codeproject.com/script/Membership/View.aspx?mid=5458212) | 26-Aug-08 1:40 |

Message Closed  

##### [[Message Removed]](https://www.codeproject.com/Messages/2697360/Message-Removed)
[ped2ped](https://www.codeproject.com/script/Membership/View.aspx?mid=4551823) | 28-Aug-08 0:23 |


Spam message removed  

##### Awesome +++++
[rfmauri](https://www.codeproject.com/script/Membership/View.aspx?mid=5387771) | 27-Jul-08 17:18 |

Gustavo,  
You have done an awesome job.  


##### ETA on New Release?
[jkuempel](https://www.codeproject.com/script/Membership/View.aspx?mid=4806885) | 8-Jul-08 10:22 |

Do you have any idea when a new version will be released?  

##### Null IconImage
[Deus ex Machina](https://www.codeproject.com/script/Membership/View.aspx?mid=3995293) | 4-Jul-08 12:36 |

i am making a bulk icon extractor and while testing my code i ran into a dll file that has a null IconImage in it so I added some null IconImage checking code to the library in the Save method of the IconFormat class. it should probably be in the Load method instead but I would not know where to put it.  
  

            <br />
public unsafe void Save(MultiIcon multiIcon, Stream stream)<br />
        {<br />
            if (multiIcon.SelectedIndex == -1)<br />
                return;<br />
<br />
            SingleIcon singleIcon = multiIcon[multiIcon.SelectedIndex];<br />
<br />
            //added code to check for nulls<br />
            for(int i = 0;i<=singleIcon.Count - 1;i++)<br />
            {<br />
                if (singleIcon[i].Encoder == null)<br />
                {<br />
                    singleIcon.RemoveAt(i);<br />
                    i -= 1;<br />
                }<br />
            }<br />
<br />

  

##### Needs more code samples [modified]
[The_Mega_ZZTer](https://www.codeproject.com/script/Membership/View.aspx?mid=2093160) | 19-Jun-08 10:13 |

The article provides nice insights into how you developed your code, but not how to USE it.  
  
Just a simple code sample showing how to package a bunch of different resolutions and color depths would help.  
  
Right now I can't load icons with the same size but different color depth into the same SingleIcon (I get an error that the same format and size icon is already loaded) even though your program can load existing icons like that just fine.  
  
[Edit: I was using SingleIcon.Add, now I'm trying SingleIcon.Load but it can't load PNGs. Wonderful. How else am I supposed to load an 8-bit or a 4-bit PNG? .CreateFile creates a SingleIcon, does .Load do the exact same thing but just more limited?  
  
Here is the closest thing I have managed to "working" code:  
  

```
Module PackIcon
    Public Sub Main()
        Dim mi As New IconLib.MultiIcon
        mi.Add("")
        mi("").Add(Image.FromFile("256-32.png"))
        mi("")(0).IconImageFormat = IconLib.IconImageFormat.PNG
        mi("").Add(Image.FromFile("48-32.png"))
        mi("")(1).IconImageFormat = IconLib.IconImageFormat.BMP
        mi("").Add(Image.FromFile("32-32.png"))
        mi("")(2).IconImageFormat = IconLib.IconImageFormat.BMP
        mi("").Add(Image.FromFile("16-32.png"))
        mi("")(3).IconImageFormat = IconLib.IconImageFormat.BMP
        mi("").Add(Image.FromFile("48-8.png"))
        mi("")(4).IconImageFormat = IconLib.IconImageFormat.BMP
        mi("").Add(Image.FromFile("32-8.png"))
        mi("")(5).IconImageFormat = IconLib.IconImageFormat.BMP
        mi("").Add(Image.FromFile("16-8.png"))
        mi("")(6).IconImageFormat = IconLib.IconImageFormat.BMP
        mi("").Add(Image.FromFile("48-4.png"))
        mi("")(7).IconImageFormat = IconLib.IconImageFormat.BMP
        mi("").Add(Image.FromFile("32-4.png"))
        mi("")(8).IconImageFormat = IconLib.IconImageFormat.BMP
        mi("").Add(Image.FromFile("16-4.png"))
        mi("")(9).IconImageFormat = IconLib.IconImageFormat.BMP
        mi("").Save("out.ico")
    End Sub
End Module
```
  
  
My problem is Image.FromFile and Icon.New seem to convert everything they load to 32-bits, so it is IMPOSSIBLE to load any icons with a lower color depth.]  
  
[Edit: I have come up with a workaround:  
  

```
Public Sub Main()
    Dim mi As New IconLib.MultiIcon
    mi.Add("")
    mi("").Add(Image.FromFile("256-32.png"))
    mi("")(0).IconImageFormat = IconLib.IconImageFormat.PNG
    mi("").Add(Image.FromFile("48-32.png"))
    mi("")(1).IconImageFormat = IconLib.IconImageFormat.BMP
    mi("").Add(Image.FromFile("32-32.png"))
    mi("")(2).IconImageFormat = IconLib.IconImageFormat.BMP
    mi("").Add(Image.FromFile("16-32.png"))
    mi("")(3).IconImageFormat = IconLib.IconImageFormat.BMP
    mi("").Add(Bitmap.FromFile("48-8.bmp"), Color.Fuchsia)
    mi("")(4).IconImageFormat = IconLib.IconImageFormat.BMP
    mi("").Add(Bitmap.FromFile("32-8.bmp"), Color.Fuchsia)
    mi("")(5).IconImageFormat = IconLib.IconImageFormat.BMP
    mi("").Add(Bitmap.FromFile("16-8.bmp"), Color.Fuchsia)
    mi("")(6).IconImageFormat = IconLib.IconImageFormat.BMP
    mi("").Add(Bitmap.FromFile("48-4.bmp"), Color.Fuchsia)
    mi("")(7).IconImageFormat = IconLib.IconImageFormat.BMP
    mi("").Add(Bitmap.FromFile("32-4.bmp"), Color.Fuchsia)
    mi("")(8).IconImageFormat = IconLib.IconImageFormat.BMP
    mi("").Add(Bitmap.FromFile("16-4.bmp"), Color.Fuchsia)
    mi("")(9).IconImageFormat = IconLib.IconImageFormat.BMP
    mi("").Save("out.ico")
End Sub
```

  
  
However it forces me to use the Bitmap.FromFile loader to allow me to get 8 and 4 bit images, so I lose all transparency information that I would have gotten if I had been able to use PNGs. It's quite annoying.]  
  

modified on Thursday, June 19, 2008 11:34 AM

  

##### [Re: Needs more code samples](https://www.codeproject.com/Messages/2620132/Re-Needs-more-code-samples)
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 1-Jul-08 14:32 |


I'm sorry for no reply before, I have been busy lately.  
  
About the samples you are 100% right, the library can do pretty much everything but without samples and documentation can be a little challenge to work with, I'm thinking I'll work on that soon.  
  
About your question, let me analyze what you are saying and I'll come back to you.  
 

##### Can you help me about that ? Icons in DLL
[MustafaMahmoud](https://www.codeproject.com/script/Membership/View.aspx?mid=3609978) | 24-May-08 5:53 |

I write a program than read and write custom file extensions and register these extensions in setup deployment project  
I want to specify icon for each file type  
I want to store these icons in dll format which can read from setup project  
Do you have any suggestion about these Dlls  


##### [Re: Can you help me about that ? Icons in DLL](https://www.codeproject.com/Messages/2571224/Re-Can-you-help-me-about-that-Icons-in-DLL)
[Vilx-](https://www.codeproject.com/script/Membership/View.aspx?mid=2349897) | 27-May-08 6:41 |


1. Create a DLL in your favourite IDE.  
2. Add icon resources to your DLL. Your IDE should allow this, but you can use any Resource Editor (like reshack).  
3. Compile the DLL  
4. Use the icons from the DLL (read MSDN on how to assign icons to filetypes).  

##### missing Feature
[Reamer](https://www.codeproject.com/script/Membership/View.aspx?mid=5126474) | 28-Apr-08 8:11 |

What i was really looking for, is a tool to delete single images out of an .ico file. I need to use icons at a tool and don´t want to save the bigger icon-formats, when i´m only using the 16x16 and 32x32 pictures. but deleting manually all the other copy´s would be very uneconomic.  
  
would be nice, if you could add this feature to your fantastic tool  

##### [Re: missing Feature](https://www.codeproject.com/Messages/2527416/Re-missing-Feature)
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 28-Apr-08 12:22 |

You can easily do that.  
  
1) Load the icon.  
2) Iterate through the images and remove the images you want.  
3) Save the icon again.  
  
Why you thought it couldn't be done, (outside that the weakest point right now is I'm not providing documentation ![Smile | :)](https://www.codeproject.com/script/Forums/Images/smiley_smile.gif) )  


##### failed to initizlize properly (oxc0000135). error
[ProsperousOne](https://www.codeproject.com/script/Membership/View.aspx?mid=5044448) | 4-Apr-08 13:12 |

I downloaded and installed the demo project on my desktop to a folder and it works fine. 
  
However, when I copy the same folder to my laptop, I get the following error:  
  
The application failed to initizlize properly (0x0000135). Click on OK to terminate the appplication.   
  
Both are running Win XP.  
  
Any suggestions?  
 

##### [Re: failed to initialize properly (oxc0000135). error](https://www.codeproject.com/Messages/2494107/Re-failed-to-initizlize-properly-oxc0000135-error)
[ProsperousOne](https://www.codeproject.com/script/Membership/View.aspx?mid=5044448) | 4-Apr-08 13:15 |


I did a google on the error, and apparently it's a sun java error (Java not installed)... Apparently, Java isn't on my laptop, and this needs java... 


##### [Re: failed to initizlize properly (oxc0000135). error](https://www.codeproject.com/Messages/2494265/Re-failed-to-initizlize-properly-oxc0000135-error)
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 4-Apr-08 15:27 |


I don't know where you looked for that error ![Smile | :)](https://www.codeproject.com/script/Forums/Images/smiley_smile.gif) , but IconLib 101% sure is not using Java. (this is a serious work).  
  
Probably that error is that you don't have .Net framework 2.0 installed on your notebook.  
  
Try installing .Net framework 2.0 and let me know how it goes.  
  
[http://www.microsoft.com/downloads/details.aspx?FamilyID=0856eacb-4362-4b0d-8edd-aab15c5e04f5&displaylang=en](http://www.microsoft.com/downloads/details.aspx?FamilyID=0856eacb-4362-4b0d-8edd-aab15c5e04f5&displaylang=en)[[^](http://www.microsoft.com/downloads/details.aspx?FamilyID=0856eacb-4362-4b0d-8edd-aab15c5e04f5&displaylang=en "New Window")]  


##### Like this!
[HankiDesign](https://www.codeproject.com/script/Membership/View.aspx?mid=2588890) | 29-Mar-08 11:58 |

Thanks!  

##### Release .74
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 17-Mar-08 12:35 |

Just I wanted to inform that probably I'll be releasing a new release in the following week.  
  
On this version there are one change and one feature.  
  
Change:  
* Pre multiply alpha channels for 32 bits source images when adding it to the SingleIcon to be color reduced to 256/16 colors.  
So far when a 32 bit image was added to the Icon (PNG24/BMP32) with color reduction to 8-4 bits, the mask was created with the incoming alpha channel but the images kept unchanged, this version will premultiply the image color information from the source bitmap with the alpha channel over a white background, that will produce very high quality color reduction for 256/16 colors from a PNG24/BMP32 format.  
(99% of the cases the premultiplication needs to be done over a white background, later version I'll add the background color as a optional parameter)  
  
Add:  
* Allow to create a mask from any bitmap format.  
Now IconImage supports a public static method, CreateMask, this method will create a mask from any bitmap, this mask can be used as parameter when adding an icon. This method is pretty usefull when you need to create a 256 color icon from a 32 bits image.  
When you use the mask from the 32 bit image to create a 8 bit color icon, it could produce a dark edge because the 32 bit image was used with shadows, now with this method the process to get a good output for a 256 color icon can be like this:  
1) Get a 256 color image from the 32 bits source image using ColorProcessing namespace.  
2) Get a new mask with IconImage.CreateMask() using as input the new 256 color image just created.  
3) Add both new image and mask to SingleIcon.Add(bmp256, mask)  
  

##### [Re: Release .74](https://www.codeproject.com/Messages/2473520/Re-Release-74)
[jkuempel](https://www.codeproject.com/script/Membership/View.aspx?mid=4806885) | 19-Mar-08 14:06 |


Any hope of a fix for the ICL support?  

##### [Re: Release .74](https://www.codeproject.com/Messages/2473559/Re-Release-74)
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 19-Mar-08 14:47 |


Which fix, still I'm adding some stuff for .74, if there is a bug and I can fix it, I'll include it.  
  
Let me know what is the problem and how to reproduce it.  
 

##### [Re: Release .74](https://www.codeproject.com/Messages/2474733/Re-Release-74)
[jkuempel](https://www.codeproject.com/script/Membership/View.aspx?mid=4806885) | 20-Mar-08 10:05 |


If you open an icl created by another application, and then save a copy of it using an application that uses IconLib, there will be some distinct differences between the two files that seems to break the copy and it won't be useable by other applications. In particular, there's a big difference in the resource size field of the metadata. The copy will often have a resource size that's less than half of the original (although the actual file size will be about the same). Also, there are some issues with loading an icl created with IconLib even with an application using IconLib in that there's some code that looks for an ID in the icons dictionary that's different than the one that was added to the dictionary (i.e. resource ID + offset vs. entry index - 1, 2, 3,...n). I was able to fix that so that my application at least didn't throw an exception when loading an icl, but it's a band-aid for right now. I look forward to any improvements in the library you make and I can email you with more information if you'd like.  

##### [Re: Release .74](https://www.codeproject.com/Messages/2475308/Re-Release-74)
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 20-Mar-08 23:19 |


Please send me an email with the code to repro the problem and the icl you want to copy from generated for another application, I'll analize the problem, if there is a problem with IconLib I'll solve it.  
  
From there if there is more problem I will work on them.  
  

##### [[Message Deleted]](https://www.codeproject.com/Messages/2479328/Message-Deleted) 
[jkuempel](https://www.codeproject.com/script/Membership/View.aspx?mid=4806885) | 25-Mar-08 9:33 |


[Message Deleted]  

##### [[Message Deleted]](https://www.codeproject.com/Messages/2479644/Message-Deleted) 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 25-Mar-08 13:15 |


[Message Deleted]  

##### [[Message Deleted]](https://www.codeproject.com/Messages/2479699/Message-Deleted) 
[jkuempel](https://www.codeproject.com/script/Membership/View.aspx?mid=4806885) | 25-Mar-08 14:19 |


[Message Deleted]  

##### [Re: Release .74](https://www.codeproject.com/Messages/2480363/Re-Release-74) 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 26-Mar-08 3:07 |

>> I’ve been working on an application that uses IconLib to create and edit icons and icon collections. There seems to be a bug when creating new ICL files and when saving copies of already existing ICL files. They are not useable by other applications.  
  
(Answer) I tried to open both ICLs you sent me and they looked fine, also I opened the original and made a copy with version .73 and got the same output size (966656 bytes).  
  
I can see in the file you sent me "ICandy - C2005 Foood-copy.icl" is using a Shift factor of 9, if that file is the copy made with IconLib, then you are using version .71, that version was the first released and was still not stable, on version .72 I changed the shift factor to 10 and should give you the same exactly size as the original you have.  
  
Please update to the last version, and let me know if you still have the same issue.  
  
(Please tell me which application are you using then I can try with those, Axialis doesn't support PNG compression for ICL formats, FYI if that is the case you are seeing)  

-----------------------  
  
>> Also, I had to make some changes to ensure that images were being encoded as PNG when they were supposed to be. It appears that `IconLib` wasn’t encoding 256x256 as PNG and that all images were BMP encoded as that was set as the default encoder. I added a parameter to the `IconImage` constructor that would set the encoder to either a PNG or BMP encoder depending on the `IconImageFormat` passed in.  
  
(Answer) Save an icon with compression PNG is an "option" and not a "requirement" for the new icons format, `IconLib` won't encode the icon unless you want to.  
  
You don't need to change `IconLib` source code.  
  
Here is an example how to do it:  
  
```
MultiIcon multiIcon = new MultiIcon();  
SingleIcon singleIcon = multiIcon.Add("Icon1");  
IconImage iconImage16 = singleIcon.Add(new Bitmap(16,16));  
IconImage iconImage256 = singleIcon.Add(new Bitmap(256,256));  
iconImage256.IconImageFormat= IconImageFormat.PNG;  
multiIcon.Save(@"c:\test.icl", MultiIconFormat.ICL);  
```
  
On .73 when you add an image that is 256x256 and is a PNG24 with alpha then it will set the IconImageFormat to PNG automatically as well.  
  
Also I recommend to reorder the way the images are stored in the icons, IconLib will honor the order you give, but that can make Windows to take a format that you are not expecting. (I saw on the copy that the order is altered)  
  
I have a function that I always execute before save the icon, it reorder the way we want to output the images in the icon, like Windows wants it.  
"Form low quality to high quality and form big size to small size."  
  
-------------------  
  
>> And finally, there was an issue with loading ICL files that were created using IconLib where an exception would be thrown. What was happening was that in RESOURCE_TABLE. SetIcons it was adding icons using TNAMEINFO.rnID as a key, but in RESOURCE_TABLE.GetIcons it was retrieving them based on the TNAMEINFO.ID property as a key, which was causing a key not found exception to be thrown because rnID didn’t equal ID most of the time. So my band-aid fix was to use ID for both.  
  
(Answer) Tomorrow, I'll study what you mean.  
  
.74, also fix a critical bug that some data on the icon directory was not saved properly, it should not be really important (it affected when Windows display the icon on the explorer window), but may be the application that you are using is relaying on this value.  
  

##### [Re: Release .74](https://www.codeproject.com/Messages/2481030/Re-Release-74) 
[jkuempel](https://www.codeproject.com/script/Membership/View.aspx?mid=4806885) | 26-Mar-08 9:59 |


>>(Answer) Save an icon with compression PNG is an "option" and not a "requirement" for the new icons format, IconLib won't encode the icon unless you want to.  
I suppose that's true. ![Smile | :)](https://codeproject.freetls.fastly.net/script/Forums/Images/smiley_smile.gif) At least I don't have to rewrite your code. I'll be sure to set the property instead of doing what I was doing before. Thanks for showing me that.  
  
>> >> And finally, there was an issue with loading ICL files that were created using IconLib where an exception would be thrown. What was happening was that in RESOURCE_TABLE. SetIcons it was adding icons using TNAMEINFO.rnID as a key, but in RESOURCE_TABLE.GetIcons it was retrieving them based on the TNAMEINFO.ID property as a key, which was causing a key not found exception to be thrown because rnID didn’t equal ID most of the time. So my band-aid fix was to use ID for both.  
  
>> (Answer) Tomorrow, I'll study what you mean.  
Now that I think of it, I am reordering the icons, so it's possible that the indexes weren't matching up because of that. I'll be sure to put them in the "expected" order and see if that fixes the problem. This may also be why some other applications (IconPackager, IconDeveloper) weren't able to open them.  
  
Thanks for your help!  

##### [Re: Release .74 (suggestion)](https://www.codeproject.com/Messages/2482865/Re-Release-74-suggestion) 
[Vilx-](https://www.codeproject.com/script/Membership/View.aspx?mid=2349897) | 27-Mar-08 10:35 |


It would be nice if the IconImage.Transparent property didn't rely on Icon.ToBitmap() but instead created the bitmap based on the masks itself. This would provide the following two benefits:  
  
1. The Icon.ToBitmap() function is documented as loosing transparency. For some reason it seems that it doesn't loose it after all, but who knows under what conditions it does loose it.  
  
2. Icon.ToBitmap() will not work for PNG compression under WinXP, and there is no workaround in your library for this.  
  
By the way - is IconImage.Image supposed to loose transparency information for 32bpp images? Because it does...  

##### [Re: Release .74 (suggestion)](https://www.codeproject.com/Messages/2485468/Re-Release-74-suggestion) 
[Vilx-](https://www.codeproject.com/script/Membership/View.aspx?mid=2349897) | 29-Mar-08 7:01 |


OK, I'm stumped... I just managed to open WinVista icons on my WinXP (256x256 PNG). I used imageres.dll from Vista and IconImage.Transparent property. Why does it work???? ![Confused | :confused:](https://codeproject.freetls.fastly.net/script/Forums/Images/smiley_confused.gif)  

##### [Re: Release .74 (suggestion)](https://www.codeproject.com/Messages/2485801/Re-Release-74-suggestion) 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 29-Mar-08 17:37 |


That's because when you think IconLib is using Icon.ToBitmap() as a method from .Net really is not, as you can see take a look in the previous property 'Icon' that property will return an Icon made by IconLib. and Transparent will create the transparent icon using the previous property 'Icon' from IconLib and not .Net Icon.  
  
```
public Icon Icon  
{  
....get {return mEncoder.Icon;}  
}  
  
public unsafe Bitmap Transparent  
{  
....get {return Icon.ToBitmap();}  
}  
```
  

##### [Re: Release .74 (suggestion)](https://www.codeproject.com/Messages/2485834/Re-Release-74-suggestion) 
[Vilx-](https://www.codeproject.com/script/Membership/View.aspx?mid=2349897) | 29-Mar-08 18:55 |


Hmm... well, the 'Icon' property is of type System.Drawing.Icon - which **is** a .NET icon. But the icon is created by your own code, and it seems, that it doesn't use a compression for that. So that takes care of the PNG compression. As for the size... I guess that Windows/.NET sees the 0x0 in the ICONDIRENTRY, so proceeds to take the actual size from the bitmap itself. Sort of like a failsafe, because (as you wrote) many icons out there do that.  
  
OK, I guess it is safe. :P But it would definately be faster to create the bitmap yourself by using the Bitmap.LockBits(). ![Big Grin | :-D](https://codeproject.freetls.fastly.net/script/Forums/Images/smiley_biggrin.gif) And the ToBitmap() method is still documented as loosing transparency for some reason. ![Sniff | :^)](https://codeproject.freetls.fastly.net/script/Forums/Images/smiley_sniff.gif)  
  
Speaking of which - why don't you use Bitmap.LockBits in your code, but instead create a binary BMP file in MemoryStream and then make the Bitmap load that? Is that faster?  

##### [Re: Release .74 (suggestion)](https://www.codeproject.com/Messages/2485855/Re-Release-74-suggestion) 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 29-Mar-08 19:50 |


Image property will give you a bitmap ready to be merge with the mask, it doesn't have alpha, independently of the input format.  
  
Mask property is the mask in B&W  
  
Transparent is a BMP32 basically is Image (XOR) merged with Mask (AND), Transparent always should give you a BMP32 with the alpha on it, if you can provide me with repro case when it doesn't then I can fix it. This can be done with LockBits, but means I have to merge all cases to get the transparent image.  

```
MASK + (XOR) 1bpp = 32ARGB  
MASK + (XOR) 4bpp = 32ARGB  
MASK + (XOR) 16bpp = 32ARGB  
MASK + (XOR) 32bpp = 32ARGB  
```

Which is a lot extra code that Icon.ToBitmap() is doing for me, unless it is not doing properly ofcourse.  
  
I see on the MSDN:  
"The transparent areas of the icon are lost when it is converted to a bitmap, and the transparent color of the resulting bitmap is set to RGB(13,11,12). The returned bitmap has the same height and width as the original icon."  
  
I see this as the orignal information in the bitmap is lost and is replaced by RGB(13,11,12), but the alpha channel will be set properly, so it doesn't harm that the trasparent pixels are changed, you still have a transparent bitmap because of the alpha channel.  
  
You lost me about the memoryStream, where in the code is that? do you mean when creating the icon? If that is the case the Icon needs MemoryStream where you can provide the XOR/AND image.  
  

##### [Re: Release .74 (suggestion)](https://www.codeproject.com/Messages/2486037/Re-Release-74-suggestion) 
[Vilx-](https://www.codeproject.com/script/Membership/View.aspx?mid=2349897) | 30-Mar-08 6:54 |


Well, then I don't understand how the Icon.ToBitmap() is supposed to work. It seems to keep the transparency anyway under all circumstances. When exactly does this RGB(13,11,12) kick in?  
  
As for the MemoryStream - I see you using it in IconImage.Image and IconImage.Mask as well - those were actually the places I was thinking about.  
  
As for the IconImage.Image and transparency - but if I have a 32bpp icon image, how can I then use the IconImage.Image for XOR merging, if there is no Alpha channel? In fact, the only way I can get to the alpha channel at all is through the IconImage.Transparent property.  

##### [Re: Release .74 (suggestion)](https://www.codeproject.com/Messages/2486286/Re-Release-74-suggestion) 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 30-Mar-08 15:54 |


Remember than in 32bpp you have 4 byes, ARGB (Alpha, Red, Green and Blue) alpha will set the transparency for the pixel between 0 and 255, if the value for alpha is set to 0, then the pixels is totally transparent independently from the value of RGB, you can have a pixel Red, Black or White and doesn’t matter the pixel is transparent, that means will never be visible.  
Icon.ToBitmap() for a transparent pixel set the ARGB value to (0, 13, 11, 12) particularly don’t know why they choose (13,11,12) for the RGB portion, I could have chosen (0,0,0) but again is a matter of choice, the pixel will never be visible anyway.  
Basically the MSDN is letting you know that some information will be changed in the case the original format is a BMP24 or BMP32, which is fine, because for a BMP1/4/8/16 needs to be changed anyway.  

---  

```
To create an icon I need to provide to the icon class a memory stream with the Image (non-transparent) and the Mask (B&W), .Net Icon class will take care of mix the two to give you a transparent Icon.  
I know there is another way to do it without the mask, something like:  
Bitmap bmp = new Bitmap(10,10);  
Icon icon = Icon.FromHandle(bmp.GetHicon());  
```

But that really is creating a mask from the input bitmap, the mask usually match with the alpha channel when is a BMP32, but for formats like BMP1/4/8/16 the mask will be created taking as transparent pixel the color from the position (0,0), which can give you really ugly results, basically that is not the right way to create an icon. Mask is the information that will be overlapped on the Image, All white pixels on the mask will be tranparent on the final image, All black pixles on the mask will keep the information in the original image.  
  
---  
  
If you want to create a transparent bitmap yourself, then you must use both, Image and Mask (take Image property and apply a AND operation with the mask), that is the way icons works, IconLib is used to work with icons, so it will give you the native format used for icons.  
  
If you are interested in get extra information about bitmaps then probably you need to do it yourself, still don’t know what you need that you think is missing? Tell me what you want to do then I could tell you what is the best way to do it with IconLib.  
  

##### [Re: Release .74 (suggestion)](https://www.codeproject.com/Messages/2486735/Re-Release-74-suggestion) 
[Vilx-](https://www.codeproject.com/script/Membership/View.aspx?mid=2349897) | 31-Mar-08 2:51 |


Oh, finally the RGB(13,11,12) makes sense! ![Smile | :)](https://codeproject.freetls.fastly.net/script/Forums/Images/smiley_smile.gif) I thought it lost the alpha channel as well.  
  
Well then... everything is fine. I guess there isn't anything to be added after all. I had just misunderstood some things. ![Smile | :)](https://codeproject.freetls.fastly.net/script/Forums/Images/smiley_smile.gif)  
  
Thanks a lot!

##### Exe Icons 
[Reuben2005](https://www.codeproject.com/script/Membership/View.aspx?mid=1745892) | 17-Mar-08 1:01 |

Let me start by saying that library is incredible! I just wanted to use Vista icons, and I've been going nuts trying to figure it out.  
I'd like to embed the png format icons (or even just some high color ico) in a .net executable, so that the icon whch appears for the exe is not limited to 8-bit color or whatever it is that the framework supports.  
Does anyone know how to compile the program using high quality icons, or how to modify an existing executable and embed the icon in it directly?  
  
Reuben  

##### [Re: Exe Icons](https://www.codeproject.com/Messages/2470214/Re-Exe-Icons) 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 17-Mar-08 12:16 |

Thank you for your comment, I was receiving several requests about inject icons on executables, that probably I'll try to support soon (don't know when), but basically it needs to put the new icons on the resource table and change the pointers where the next segments were pointing before to the new location, it has to take care of many details, I guess that the reason why there are no too many programs on the market that can do that.  
  
Good luck with the searching.  

##### Creating icons 
[snafoo](https://www.codeproject.com/script/Membership/View.aspx?mid=4927234) | 2-Mar-08 4:29 |

Hey, you did a great job.  
  
What I am looking for is a free application that is using your library to create icons with different sizes and color depths from image files. Some people here talked about using your library in an open source program, but didn't gave a link or anything. Do you know of any of such free programs?  

##### Well Done 
[Abolfazl Khusniddinov](https://www.codeproject.com/script/Membership/View.aspx?mid=1179445) | 18-Feb-08 7:56 |

Hi...  
I'm very thankful for this great project.I used your previous version in my B.Sc. final project.I have added all last features myself(Color space, etc) I have designed and programed 4 applications in this direction :  
1) Icon Editor  
2) Image 2 Icon  
3) Icon Librarian  
4) Icon Changer  
If you are interested in my work I can send you screen shots.  
You know will be nice to support jpg2000 format.It's used in icon creation.I think that's the only shortage in your library.  
Regards  
  

Nothing is impossible

  

##### I am interested in icon. 
[mrmengyi](https://www.codeproject.com/script/Membership/View.aspx?mid=4858029) | 1-Feb-08 20:06 |

I write a simple icon editor on November 2005. It has limited function at beginning.  
Half past one year, I rewrite it with WTL. I call it IMormor.  
  
If I knew your job earlier, I would use your works. 
  
Recently, I added xp-icon feature to IMormor. I maybe share the code when I finished it.  

##### I'm back... 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 19-Jan-08 2:29 |

Sorry for not reply to the questions in the forum for long time, I was working on some heavy projects.  
  
In the next following days I'll be releasing version 0.73, it fixed all the problems the people reported.  
  
Plus I added functionality that were requested and some things more.  
  
*Fixed a small problem with indexed 8bpp images.  
*Properly processing when adding PNG24 images.  
*Automatic Icon creation from a PNG or BMP32 for Vista, XP, W95 and Win31  
*Added a new namespace "ColorProcessing" which supports  
-Color Reduction  
-Dithering  
-Palette Optimization  
*Allow to save an IconImage as PNG or BMP32 with transparency.  
*Some code and method signatures changes but backward compatible.  
  
Regards,  
Gustavo  
  

--  
If you think the chess rules are not fair, first beat Anand, Kasparov and Karpov then you can change them.  
Moral is, don't question the work of others if you don't know the reason why they did it.

  

##### I have a few questions... 
[rossymiles](https://www.codeproject.com/script/Membership/View.aspx?mid=3955734) | 24-Oct-07 5:44 |

I am using your icon library to make a open-source icon converter. Instead of linking to the DLL or the project file, I have decided to create my own library based on the source code of IconLib. The reason for this is I wanted my library to include advanced features such as processing the raw PNG data in Vista Icons, and I wanted the library to be able to export the image to a System.Drawing.Bitmap.  
  
I have started to work on both the frontend of the application, and the icon processing library but I have a few questions:  
1. How can I tell a Windows XP icon apart from an old icon? I assume biCompression is BI_RGB and, according to Wikipedia, the XOR image contains the 32bpp image data and the AND image is not used.  
2. I read on Wikipedia that the XOR/AND bitmap format found in pre XP icons could be used to "invert the background and other tricks". I know how to invert the background, but what are the "other tricks"? Have you heard of any? It sounds interesting but I couldn't find any info in a google search.  
3. Does any part of IconLib or .NET calculate a palette for images that are less then 24bpp?  
4. In Vista PNG icons, is the whole PNG stream stored in the file, or only the important chunks? (eg "IDAT" and "PLTE")  
5. Are 256 color (8-bit) PNG compressed icons stored in PNG8 format.  
  
Thanks for releasing your library to open source, it is the only open source icon library I could find, and one of the few icon libraries/editors I know of that could read Windows Vista icons. Good job! ![Big Grin | :-D](https://www.codeproject.com/script/Forums/Images/smiley_biggrin.gif)  

##### Export To PNG 
[ajakblackgoat](https://www.codeproject.com/script/Membership/View.aspx?mid=1049269) | 17-Sep-07 21:08 |

Is it possible to save the selected icon to PNG file?  
  
Anyway good work.  

##### [Re: Export To PNG](https://www.codeproject.com/Messages/2231684/Re-Export-To-PNG) 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 18-Sep-07 0:21 |


Good question,  
  
I did the library some time ago, and I was sure it could be done, but watching in the object diagram looks like you can get the XOR image and save it as PNG, but it won't have transparency.  
  
Tomorrow I'll see the code, and if it is not implemented probably I'll do it because is a very good point, save the individual icons as PNG creating transparency from the AND image.  
  

##### Replace resources 
[XPero](https://www.codeproject.com/script/Membership/View.aspx?mid=2384903) | 16-Jul-07 15:01 |

Any news on the replace resources feature? I have something big ready and I'm waiting for your wonderful library to finish it 
  
Thank you for all your hard work!  


##### Is there an version under .Net Framework 1.1 
[Gx Lam](https://www.codeproject.com/script/Membership/View.aspx?mid=711388) | 14-Jul-07 10:44 |

I am working with vs2003.net, so is there an version of this excellent code under .Net Framework 1.1 ?  

##### Invalid .ICL-Output 
[J4yK](https://www.codeproject.com/script/Membership/View.aspx?mid=2202601) | 16-Jun-07 13:09 |

Hello,  
first I want to thank you again for writing this great library 
But unfortunately there is a problem with the ouput of ICL-files... Whenever I want to Save a MultiIcon instance in ICL format, the resulting file is not readable neither by Windows nor by my icon editor. The icon editor fails: No icons in this file! The only apps that can parse those created NE-files are apps that use IconLib.  
When I create DLLs everything is fine (despite the uppercase icon names, but looking into the code I see that you wrote it explicitely that way). I could also create .icl files which are actually DLLs but this is not the preffered way I guess ![Wink | ;)](https://www.codeproject.com/script/Forums/Images/smiley_wink.gif)  
I don't think it's because of invalid icon images. I copied some SingleIcons from a shell32.dll MultiIcon to another MultiIcon and saved the latter one as ICL: failure.  
But I wonder why nobody else seems to have noticed this issue or nobody posted it here... what can I do?  


##### Another Question 
[Abolfazl Khusniddinov](https://www.codeproject.com/script/Membership/View.aspx?mid=1179445) | 5-Jun-07 11:36 |

I have some png files how can i convert them into icon format by your library?  
thank you in foreward.  
  

##### Thanks one little question 
[Abolfazl Khusniddinov](https://www.codeproject.com/script/Membership/View.aspx?mid=1179445) | 5-Jun-07 11:04 |

I was interested so long time in icons and i have done something for myself.I designed an application with lovely user interface and I can share all that I've got with you.Your library is perfect.Is is possible to change an icon of execution files by this library?  
how can i contact you?  
my e-mail is : afaaz2004@yahoo.com  


##### Problem with Tools.RGBQUAD and 8bit images 
[Sepharo](https://www.codeproject.com/script/Membership/View.aspx?mid=4114893) | 4-Jun-07 11:36 |

RGBQUAD which creates the palette for the iconimage works fine for 4bit and 32bit images but when it takes an 8bit image and begins to cycle through the for loop the 256 iterations take too long and my program will lock up and not respond.  
  
There are 16 iterations for 4 bit but 256 iterations for 8bit... I guess it's during these 256 loops that the program stops responding.  

##### [Re: Problem with Tools.RGBQUAD and 8bit images](https://www.codeproject.com/Messages/2066383/Re-Problem-with-Tools-RGBQUAD-and-8bit-images)
[Sepharo](https://www.codeproject.com/script/Membership/View.aspx?mid=4114893) | 4-Jun-07 14:56 |


Got this solved right here on code project with tons of help and patience from Mark Salsbery  
  
[http://www.codeproject.com/script/comments/forums.asp?msg=2066214&forumid=387159#xx2066214xx](http://www.codeproject.com/script/comments/forums.asp?msg=2066214&forumid=387159#xx2066214xx)  
  
We had to separate the properties from the loop and that increased performance to what it should be when looping 256times.  
  
For example: instead of `bmp.Palette.Entries` we used `Color[] palcolors = palette.Entries;` and so on in that fashion. Full details of the fix can be found through the link.  

##### Excellent!
[Ron Cicotte](https://www.codeproject.com/script/Membership/View.aspx?mid=375291) | 24-May-07 11:55 |

Thank you very much for all your good work on this library. Like you I was looking for a quick solution to a problem when I came across this article. I wanted to use an existing photo for an icon for one of my applications but couldn't figure out how to convert the image to an icon. The explanation and illustrations in this article are the best I have seen on the format and structure of icon files and libraries. I really appreciate your work.  
  
I have now decided to use IconLIb to create a utility that converts images to icons. The basic prototype is complete and functioning but I still need to add the ability to specify image sizes and output formats. When this is done I am going to convert it to a web service and make it available on my web site [Summerstreet.com](http://www.summerstreet.com/)  

  ##### Great Info 
[LimeyRedneck](https://www.codeproject.com/script/Membership/View.aspx?mid=418776) | 15-May-07 9:39 |

Wonderful article - the NE stuff is great.  

##### I wish I had found this before I began 3 days of research
[Sepharo](https://www.codeproject.com/script/Membership/View.aspx?mid=4114893) | 10-May-07 12:00 |

You are lifesaver. This is EXACTLY what I've been looking for for days. I knew it had to exist out there somewhere and this whole time all I had to do was look right under my nose at the code project.  

##### PNG image loading...
[Jim D,](https://www.codeproject.com/script/Membership/View.aspx?mid=2140026) | 3-Apr-07 21:23 |

I was looking desparetly for something that will end vista Icons and found your article.. And I must say that it's AWESOME, EXCEPT, When loading an image from a PNG(256x256) into the icon, there is no way to tell the library to treat it as a PNG format before it butchers the transparency in the file. Do you have any suggestions on how to effectivly load PNG images?  

##### [Re: PNG image loading...](https://www.codeproject.com/Messages/1973756/Re-PNG-image-loading)
[Jim D,](https://www.codeproject.com/script/Membership/View.aspx?mid=2140026) | 3-Apr-07 21:41 |


Nevermind... Color.Transparent will keep the file from falling appart. But there is still no way to get it to treat the image as a PNG on the SingleIcon.Add() method.  

##### [Re: PNG image loading...](https://www.codeproject.com/Messages/2032635/Re-PNG-image-loading) 
[Sepharo](https://www.codeproject.com/script/Membership/View.aspx?mid=4114893) | 11-May-07 14:03 |


I think I might be having the same problem... Also I noticed that you can't extract the mask from a bitmap or png until after it becomes an icon and by that time it's already too late because the mask was not included in icon creation. At least I think that might be my problem.  
  
  
When I import a png the image translates perfectly when stored in a bitmap but when I then add it with singleIcon.Add(bmp) it loses some of it transparency... I say some because the constructor that's being used specifies that the 0,0 pixel will be the transparent color but simply doing that leaves jagged edges and just looks messy in icon form. I think this would be solved by including the mask [but what do I know?] but with the way the library is structured right now I wouldn't receive the mask until after it becomes an icon.  
  
Do I need to write my own code outside IconLib to produce a mask or am I just totally way off base with this entire post?  

##### [Re: PNG image loading...](https://www.codeproject.com/Messages/2032670/Re-PNG-image-loading) 
[Sepharo](https://www.codeproject.com/script/Membership/View.aspx?mid=4114893) | 11-May-07 14:23 |


Welp apparently I'm an idiot because all I had to do was do exactly what the guy above me said to do and add Color.Transparent to the parameters.  
  
I'm still interested in a better method for importing png's but this will work for now I hope.  

##### Wow. 
[Jerry Evans](https://www.codeproject.com/script/Membership/View.aspx?mid=9484) | 22-Mar-07 19:50 |

Just stumbled across this on a random trawl of CodeProject. Very nice and with an excellent article as well. Many thanks.  
  
I just rated this 5. It would have been 10 but you've used C#, not C++. 


##### License compatible with LGPL? 
[J4yK](https://www.codeproject.com/script/Membership/View.aspx?mid=2202601) | 3-Mar-07 16:42 |

I'm so glad I stumbled upon this library. Needing to provide export and import icon library features in my program... But it is licensed under the LGPL and your CC license reads "share-alike" ("if you ... build upon this work ... you may distribute the resulting work ... under the same _or similar_ license"). I don't know whether then it is possible to use and ship your library with my application under the LGPL (is the LGPL 'similar enough'?)... can I do so or do I have to re-license my project?  


##### [source file headers read another license so...](https://www.codeproject.com/Messages/1934621/source-file-headers-read-another-license-so) 
[J4yK](https://www.codeproject.com/script/Membership/View.aspx?mid=2202601) | 11-Mar-07 11:19 |

Hum. It helps reading also the .cs-file headers as these read a 2-clause BSD-like license ![Blush | :O](https://www.codeproject.com/script/Forums/Images/smiley_redface.gif)  
I think this solves the problem, I'm sorry for this inconvenient question ![Wink | ;)](https://www.codeproject.com/script/Forums/Images/smiley_wink.gif)  

##### [Re: License compatible with LGPL?](https://www.codeproject.com/Messages/1963608/Re-License-compatible-with-LGPL) 
[jinopmathew](https://www.codeproject.com/script/Membership/View.aspx?mid=3229921) | 28-Mar-07 10:06 |

Good Article  

##### How to convert 1 bmp to 16x16 ico? 
[SveinErik](https://www.codeproject.com/script/Membership/View.aspx?mid=1455794) | 2-Feb-07 15:59 |

Hi!  
Thank you very much for sharing this great resource!  
  
I'm wondering how I can convert 1 bmp, jpg or png to 16x16 ico?  
I'm a newbie to programming..  
  
I guess it's pretty easy, but could you please give me some hints on how to do this?  
  
The image file that's going to be converted to icon file, is selected by the user. It's not a dll file, it's just a plain image file.  
  
Thanks again!  

##### [Re: How to convert 1 bmp to 16x16 ico?](https://www.codeproject.com/Messages/1875341/Re-How-to-convert-1-bmp-to-16x16-ico) 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 2-Feb-07 17:18 |


This should do the trick.  

```
//bmp is the source bitmap and can be anything you want;  
Bitmap bmp = new Bitmap(50,50);  
  
Bitmap bmpIco = new Bitmap(bmp, new Size(16, 16));  
Icon icon = Icon.FromHandle(bmpIco.GetHicon());  
bmpIco.Dispose();  
```
  
or  

```
Icon icon = null;  
using(Bitmap bmpIco = new Bitmap(bmp, new Size(16, 16)))  
....icon = Icon.FromHandle(bmpIco.GetHicon());  
```

now u have a 16 by 16 icon and you can assign it to anything for example to your form.  
  
this.Icon = icon;  
  
Don't forget to vote for the article.  

##### [Re: How to convert 1 bmp to 16x16 ico?](https://www.codeproject.com/Messages/1875773/Re-How-to-convert-1-bmp-to-16x16-ico) 
[SveinErik](https://www.codeproject.com/script/Membership/View.aspx?mid=1455794) | 3-Feb-07 5:11 |


Thank you for your fast reply! 
  
The problem with your example is that it does not load the image from a file..  
I need to convert the file "image.bmp" to "image.ico" (16x16)  
  
I was thinking about using IconLib, but how can i load only one image and convert it?

##### [Re: How to convert 1 bmp to 16x16 ico?](https://www.codeproject.com/Messages/1876049/Re-How-to-convert-1-bmp-to-16x16-ico) 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 3-Feb-07 11:44 |

```
Bitmap bmp = (Bitmap) Bitmap.FromFile("D:\\Windows\\Coffee Bean.bmp");
Bitmap bmpIco = new Bitmap(bmp, new Size(16, 16));
Icon icon = Icon.FromHandle(bmpIco.GetHicon());
bmp.Dispose();
bmpIco.Dispose();
this.Icon = icon;
```

or 

```
Icon icon = null;
using (Bitmap bmp = (Bitmap) Bitmap.FromFile("D:\\Windows\\Coffee Bean.bmp"))
using (Bitmap bmpIco = new Bitmap(bmp, new Size(16, 16)))
    icon = Icon.FromHandle(bmpIco.GetHicon());
this.Icon = icon;
```

Here is the way to load from a file and convert to icon 16x16 size.  
  

##### [Re: How to convert 1 bmp to 16x16 ico?](https://www.codeproject.com/Messages/1876194/Re-How-to-convert-1-bmp-to-16x16-ico) 
[SveinErik](https://www.codeproject.com/script/Membership/View.aspx?mid=1455794) | 3-Feb-07 15:35 |


Thank you for your answer.  
  
But this produces an icon file that is not possible to open with any image programs..windows standard program, paint, adobe photoshop won't open the file..do you know why?  
  
is it possible to use IconLib to achieve my goal?  

##### [Re: How to convert 1 bmp to 16x16 ico?](https://www.codeproject.com/Messages/1876344/Re-How-to-convert-1-bmp-to-16x16-ico) 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 3-Feb-07 19:27 |


If you want a good answer formulate the right question.  
  
Adobe Photoshop does not understand .ico format for that reason you can't open any icon with it.  
  
The following code will save it on the file system.  
  
```
Bitmap bmp = (Bitmap) Bitmap.FromFile("D:\\Windows\\Coffee Bean.bmp");
Bitmap bmpIco = new Bitmap(bmp, new Size(16, 16));
Icon icon = Icon.FromHandle(bmpIco.GetHicon());
bmp.Dispose();
bmpIco.Dispose();

FileStream fs = new FileStream("c:\\icon.ico", FileMode.Create, FileAccess.Write);
icon.Save(fs);
fs.Close();
```
  
  
Testing for 16x16 icons looks like .Net Icon class save it to the file system with 4bpp format, and I think Paint cannot understand icon for 4bpp (16 colors)  
  
Anyway if you open the icon with another Icon Editor, for example IconLib demo from this article, you will see that the icon can be opened without problems.  
  
Now having a 4bpp icon is not useful because the quality is pretty bad, coming back to your previous question, yes... you can create the icon you are looking for with IconLib.  
  
Use the following piece of code, this will give you a nice looking icon.  
  
```
MultiIcon multiIcon = new MultiIcon();
SingleIcon icon = multiIcon.Add("Icon 1");
Bitmap bmp = (Bitmap) Bitmap.FromFile("D:\\Windows\\Coffee Bean.bmp");
Bitmap bmpIco = new Bitmap(bmp, new Size(16, 16));
icon.Add(bmpIco);
icon.Save("c:\\icon1.ico");
bmp.Dispose();
bmpIco.Dispose();
```

##### [Re: How to convert 1 bmp to 16x16 ico?](https://www.codeproject.com/Messages/1876352/Re-How-to-convert-1-bmp-to-16x16-ico) 
[SveinErik](https://www.codeproject.com/script/Membership/View.aspx?mid=1455794) | 3-Feb-07 19:38 |


I'm really sorry if I formulated the question badly.  
  
Thank you VERY much for your help, I really appreciate it! The code worked like a charm! 
Now I'm going to try to make it compatible with jpg and png files as well, I guess that's the same procedure 
  
Again, thank you very much.  
 

##### [Re: How to convert 1 bmp to 16x16 ico?](https://www.codeproject.com/Messages/1878635/Re-How-to-convert-1-bmp-to-16x16-ico) 
[SveinErik](https://www.codeproject.com/script/Membership/View.aspx?mid=1455794) | 5-Feb-07 13:29 |


I'm sorry to bother you more, but I've googled for hours without finding any hints on this one, maybe you can help me out?  
  
The code you posted works fine, but when i try to use the generated icon as favicon, it does not work. I've read that the favicon.ico needs to be max 16x16 pixels and no more than 16 colors.  
The pixels on the generated icons are correct, but I guess it must be the colors that is wrong, because it doesn't work to use it as a favicon.  
  
Do you know how to do this?  


##### Thank you for your great library. 
[PunCha](https://www.codeproject.com/script/Membership/View.aspx?mid=2294688) | 30-Jan-07 23:26 |

I have been searching the internet for a long time to find out how to extract 48*48 icons from the EXE file. And I think I find the solution now!!!  
Thanks again to your library. BTW, I almost fogot to vote for your article, and what reminded me was your demo application ![Laugh | :laugh:](https://www.codeproject.com/script/Forums/Images/smiley_laugh.gif)  


##### What has changed deeply in ExtractIconA ? 
[davdes5457](https://www.codeproject.com/script/Membership/View.aspx?mid=3683780) | 31-Dec-06 21:50 |

Hi,  
You are the first to show me that it was possible to continue to use .ICL files with Vista.  
  
My ICLs are built with Axialis IconWorkshop but with Vista I can't no more extract icons using "ExtractIconA".  
Maybe could you explain me what has changed into this function because :  
_98,2000,XP have no problem, i get the icon's handle  
_vista RC1 returns me 0 (zero)  
  
Note : with a true DLL like Shel32.DLL it works fine on all O/S.  
  
Very strange, and i didn't find any explanation on this.  
  
Then, if you still have any insomnia;) , you can try this challenge.  


##### [Re: What has changed deeply in ExtractIconA ?](https://www.codeproject.com/Messages/1823629/Re-What-has-changed-deeply-in-ExtractIconA) 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 31-Dec-06 22:17 |
  
I didn't follow what exactly what the problem is.  
  
>> My ICLs are built with Axialis IconWorkshop but with Vista I can't no more extract icons using "ExtractIconA".  
  
Are you using ExtractIconA to extract icons from a ICL? I didn't know you could do that.  
  
Also Axialis IconWorkshop cannot create a ICL with Windows Vista icons, it can store 256x256 but it cannot store PNG compression over a ICL, if you try it will be automatically converted from PNG to BMP, at least the last time I tried was version 6.0 and it could not do it.  
  
Could you give me more detail what you can’t extract with ExtractIconA?  
  
From OS? XP, Vista?  
Icon/s in the file? 64x64x, 256x256, 256x256PNG?  
Source File? DLL, ICL?  


##### [Re: What has changed deeply in ExtractIconA ?](https://www.codeproject.com/Messages/1823895/Re-What-has-changed-deeply-in-ExtractIconA) 
[davdes5457](https://www.codeproject.com/script/Membership/View.aspx?mid=3683780) | 1-Jan-07 10:37 |

With ExtractIconA (from Shell32) i can extract icon from a DLL,ICL,EXE giving the icon index into the file. What i can't extract from Vista are icons from ICL. All icons, even 16x16 16 colors.  What i can extract from Vista are icons from DLL or EXE. All icons, at least 48x48+alpha. But true DLLs only (not done with Axialis).  
  
Just for fun, i tried to use the "library.icl" you provided in you project ; it doesn't work too.  
  
With other OS (98,2000,XP), no problem : i can extract all icons from any type files.   

##### Nice work... but a problem! 
[oskar1111](https://www.codeproject.com/script/Membership/View.aspx?mid=3418966) | 24-Nov-06 18:17 |

Really nice work here, the application and library work flawlessly.  
  
Only one thing though: I'm trying to use an System.Drawing.Icon instance as returned by SingleIcon.Icon as a form icon by setting the form's Icon property. This works fine, but it's always the smallest image in the icon that is used. E.g. the 16x16 image in the top left of the window looks good, but the 32x32 image for the window when you press Alt+Tab is just the 16x16 image resized. In other words, it looks very "pixelized". I tried all possible ways, ExtractIcon, ExtractAssociatedIcon, ... but they all give just "one size" icons. It seems this is a limitation in Form it seems. But then again, if you use real static ".ico" files at compile-time, all images inside the icon are used properly.  
  
Oskar Liljeblad (oskar@osk.mine.nu)  
(author of icoutils, http://www.nongnu.org/icoutils/)  

##### [Re: Nice work... but a problem!](https://www.codeproject.com/Messages/1789993/Re-Nice-work-but-a-problem) 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 5-Dec-06 0:38 |
  
As you told, the Form behave properly if you link it with static Icons, try use a separate icon from the file system at the run-time, if the problem is the same as IconLib library then you can guaranty that the problem is happening inside Windows Form or .Net Ico and has nothing to do with the Icon generated for IconLib, if that is not the case, then I can see where the problem could be.  
  

##### [Re: Nice work... but a problem!](https://www.codeproject.com/Messages/1791223/Re-Nice-work-but-a-problem) 
[oskar1111](https://www.codeproject.com/script/Membership/View.aspx?mid=3418966) | 5-Dec-06 15:18 |


Yeah, same problem when using "extracted" icon files at run-time using other APIs (even Win32)... So it seem that the problem is not in IconLib.  
  
Oskar  

##### [Re: Nice work... but a problem!](https://www.codeproject.com/Messages/1947863/Re-Nice-work-but-a-problem) 
[GWSyZyGy](https://www.codeproject.com/script/Membership/View.aspx?mid=445180) | 19-Mar-07 11:21 |


I've noticed the same behavior. I've seen posting that suggest "packing" the multi-icon from largest to smallest (e.g. 48x48 1st icon in file, then 32x32, 16x16). Most utilities will pack them from smallest to largest.  
  
Personally, I've just resorted to keeping multiple separate embedded resources -- either just the largest, or largest + optimized 16x16 (used frequently in listviews, menu commands, etc.) and rezise to other resolutions as needed.  
  
Nice job, BTW .. certainly gets my 5! 

##### Export Icl 
[royan](https://www.codeproject.com/script/Membership/View.aspx?mid=293593) | 23-Nov-06 2:49 |

it will be nice if there was an option to export all icon at once 
  

##### [Re: Export Icl](https://www.codeproject.com/Messages/1789991/Re-Export-Icl) 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 5-Dec-06 0:34 |


I spent no more than 2 hours to create the front-end, the objective of the library is to give the set of APIs, the front-end is just a simple application to "show" what could be done with a few lines of code.  
  
To implement the funcionality that you tell in the front-end could be just 5 minutes more, the library gives all the tools to do it.  
  

##### I would have spent the $19 to $39 
[Kevin C Ferron](https://www.codeproject.com/script/Membership/View.aspx?mid=865123) | 10-Nov-06 17:37 |

just sayin  

##### [Re: I would have spent the $19 to $39](https://www.codeproject.com/Messages/1752374/Re-I-would-have-spent-the-19-to-39) 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 10-Nov-06 17:56 |


What's a life without fun and challenges? ![Smile | :)](https://codeproject.global.ssl.fastly.net/script/Forums/Images/smiley_smile.gif)  
  
Personally I think there are nothing more satisfactory than discover how some obscure thing is done and expose to the general public, where basically you tell ... you see... you were keeping it for yourself like if it was something impossible to do and somebody else gave it for free, so you lost the chance to be a good guy ![Smile | :)](https://codeproject.global.ssl.fastly.net/script/Forums/Images/smiley_smile.gif) , may be the next time you will share more...  
  
P.S.: If a product outthere could have give me a package for $20 exposing APIs, then probably I would not have needed to do this.  
  

##### [Re: I would have spent the $19 to $39](https://www.codeproject.com/Messages/1753113/Re-I-would-have-spent-the-19-to-39) 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 11-Nov-06 23:12 |


Although a second though I think no everything released to the general public always is for good.  
  
Albert discovered e=mc^2 which was fantastic and the first use was to kill 200.000 Japanese people.  
  

##### [Re: I would have spent the $19 to $39 [modified]](https://www.codeproject.com/Messages/1811066/Re-I-would-have-spent-the-19-to-39-modified) 
[Ilíon](https://www.codeproject.com/script/Membership/View.aspx?mid=1311446) | 19-Dec-06 11:13 |


"_Albert discovered e=mc^2 which was fantastic and the first use was to kill 200.000 Japanese people._"  
But how many Japanese and American lives were spared due to the abrupt end of the war?  
  
Would you prefer that the war had continued for another year or two? Would it have been better to carpet-bomb Japan (it surely would have been done), on the theory that "_Sure, more people will die, in total, but at least the deaths will be caused by many explosions, rather than by merely two?_"  
  
Would it have been better if the US had not fought Japan in the first place? Would it have been better to leave the Chinese, Koreans, Fillipinos, etc, and the Japanese people themselves, enslaved by the militaristic regime of the Japanese empire? Sure, many millions of people would have died due to that regime, and multiple tens of millions would lived entire lives of misery. But, hey! *Our* hands would have been "clean!"  
  
There is no such thing as a free lunch. Anywhere.  
  
All choices made, including the refusal to choose, eliminate other potential choices. All choices involve a trade-off of costs and benefits ... with it being quite often the case that before the choice is made we can't really and fully know the true scope of costs and benefits. And, once the one choice is made, it generally remains the case that we can't *really* know what the true costs and benefits would have been to have made another choice.  
  
  
-- modified at 12:19 Tuesday 19th December, 2006  
Edit for clarification:  
This is not meant as an attack on you. This is meant to point out to you that the all-too-common sentiment you expressed follows from very shallow thinking.  

##### [Re: I would have spent the $19 to $39](https://www.codeproject.com/Messages/1811080/Re-I-would-have-spent-the-19-to-39) 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 19-Dec-06 11:29 |


If they want to play war, ok, go to the artic and kill each other, but the 2 bombs were at civilian targets, basically terrorist hit two planes against the twin tower murdering 5000 civilians, and USA hit two cities and murder 200.000 and the majority were civilians.  
  
Basically there is no difference, between Japan attack and twin towers attack; a war should respect civilians, which USA and terrorist don’t do. I don’t want to create a thread of this; just I want to make clear that your trade-off is not fair because you are changing military for civilians, military can kill each other between countries and still is kind of "fair fight", execute and murder civilians is not.  
  

##### [Re: I would have spent the $19 to $39](https://www.codeproject.com/Messages/1824914/Re-I-would-have-spent-the-19-to-39) 
[Oakman](https://www.codeproject.com/script/Membership/View.aspx?mid=420158) | 2-Jan-07 10:08 |


I am impressed by your programming skill and appalled at your utter lack of knowledge of history.  
  
If you want to compare any WWII event to 9/11, there is no other choice except Pearl Harbor.  
  
There was no greater killer of civilians in WWII than Japan, except their closest ally: Germany. Ask the Chinese and Korean survivors of the Japanese invasion of their countries, if you need corroboration. But you will have to give up your naive and simplistic view of warfare before you can understand what happened then.  
  

##### [Re: I would have spent the $19 to $39](https://www.codeproject.com/Messages/1825099/Re-I-would-have-spent-the-19-to-39) 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 2-Jan-07 12:09 |


Pearl Harbor reinforces what I said. It was a military base and the attack was made with the intention of sink the most amounts of ships and military and not kills the most of the possible amount of civilians, the amount of people was about 2400 and about 70 of them were civilians.  
  
It is not a “naive and simplistic view of warfare”, I’m totally against when war involve civilians, call it “naïve” call it “simplistic”, probably you could think differently if you had one relative in the World Trade Center, where in the morning went to work and didn’t expect to be a target of war.  
  
If the two bombs had been throw on the sea close to Japan with a warning, they could have sh*t theirs pants and had surrender too.  
  
We could talk for hours about the past but I don’t want to create a thread about it.  
  
Also there are no simple or complex warfare, WAR is wrong in all meaning and as long we don’t understand that, we will never be able to achieve the world peace.  
  
A clear example is that Saddam Hussein was a sick bastard, and hang him won’t help at all, it will just create more chaos.  
  

##### [Re: I would have spent the $19 to $39](https://www.codeproject.com/Messages/1825341/Re-I-would-have-spent-the-19-to-39) 
[Oakman](https://www.codeproject.com/script/Membership/View.aspx?mid=420158) | 2-Jan-07 15:16 |


> CastorTiu wrote:
> 
> It is not a “naive and simplistic view of warfare”, I’m totally against when war involve civilians, call it “naïve” call it “simplistic”, probably you could think differently if you had one relative in the World Trade Center, where in the morning went to work and didn’t expect to be a target of war.

  
  
War has always involved civilians - ask the citizens of Troy or Carthage - thinking otherwise is, I am afraid, naive.  
  
How about seven fellow workers who were on Flight 11? One of 'em sat less than 25 feet from my cube. Are you saying that their lives were more valuable than the friends I've lost who were in uniform?  
  

> CastorTiu wrote:
> 
> Also there are no simple or complex warfare, WAR is wrong in all meaning and as long we don’t understand that, we will never be able to achieve the world peace.

  
  
War is infinitely preferable to a number of alternatives. The reasons for war can be complex and range from national survival to the eradication of evil.  
  

Jon  
Information doesn't want to be free.  
It wants to be sixty-nine cents @ pound.

  

##### [Re: I would have spent the $19 to $39](https://www.codeproject.com/Messages/1825463/Re-I-would-have-spent-the-19-to-39) 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 2-Jan-07 17:09 |


> Oakman wrote:
> 
> How about seven fellow workers who were on Flight 11?

  
  
I don't see your point, do you think that was fair for them? Probably leaving behind spouses and kids because "we are at war"?  
  

> Oakman wrote:
> 
> Are you saying that their lives were more valuable than the friends I've lost who were in uniform?

  
  
I NEVER said that, your friends in uniform chose to be there and they must be respected. They knew the risks that were taking everyday. That is a huge difference between military and civilians.  
  

> Oakman wrote:
> 
> War is infinitely preferable to a number of alternatives.

  
  
I could have written:  
  
There is an infinitely number of alternatives than war.  
  
I don’t know where you want to drive this; I’m not trying to tell you that your belief are wrong, I keep saying the same, in a war civilians must be respected.  
  
Of course there are hundred of examples where they were not respected and they were killed but it doesn’t mean “it is the way it is and will be”. If we look in the past then we can learn and do better, we could have drop an atomic bomb in Iraq and start over and repeat the history again.  
  
No atomic bomb happened and anyway 3000 USA soldiers and 650,000 Iraqis have die.  
In fact the risk of dying from violence after the invasion is more than 50 times than before.  
  
That’s very sad, if war was the preferable method over the alternative, ohh my god I don’t want to think even what the alternatives were.  
  

##### [Re: I would have spent the $19 to $39](https://www.codeproject.com/Messages/1961494/Re-I-would-have-spent-the-19-to-39) 
[shahp](https://www.codeproject.com/script/Membership/View.aspx?mid=2458502) | 27-Mar-07 8:59 |


says u...  
  
this is not the place for your personal rants and or beliefs mr pragmatic...  

#####  [Re: I would have spent the $19 to $39](https://www.codeproject.com/Messages/1962426/Re-I-would-have-spent-the-19-to-39) 
[Ilíon](https://www.codeproject.com/script/Membership/View.aspx?mid=1311446) | 27-Mar-07 20:21 |


And yet, you seem to think it the place for your personal rants and faulty opinions.  


#####  [Re: I wouldn't](https://www.codeproject.com/Messages/1910558/Re-I-wouldnt) 
[RomanskiSt](https://www.codeproject.com/script/Membership/View.aspx?mid=2618214) | 24-Feb-07 12:03 |


I've been using this free one: http://www.towofu.net/soft/e-aicon.php and it's been fine. Can do png to multi-icon alpha-transparent .ICO files. That's all I need.  
  
Roman  

#####  Wonderful, exactly what I had hoped to find on CodeProject 6 months ago 
[Pierre Arnaud](https://www.codeproject.com/script/Membership/View.aspx?mid=36474) | 7-Nov-06 2:28 |

Hi Gustavo,  
  
You've done a really great job of getting all these different icon file formats together under one single API. I'll probably consider integrating an export function for ICOs into my pet project, [Creative Docs .NET](http://www.creativedocs.net/) (www.creativedocs.net).  
  
Thanks!  
  
Pierre  

#####  [Re: Wonderful, exactly what I had hoped to find on CodeProject 6 months ago](https://www.codeproject.com/Messages/1745972/Re-Wonderful-exactly-what-I-had-hoped-to-find-on-C) 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 7-Nov-06 10:53 |


Hi Pierre,  
  
Your product looks really great, if you incorporate IconLib to export icons count with my support for the library.  
  
Regards,  
Gustavo.  
  

--  
If you think the chess rules are not fair, first beat Anand, Kasparov and Karpov then you can change them.  
Moral is, don't question the work of others if you don't know the reason why they did it.

  

#####  Good Work 
[CaptainOblivious](https://www.codeproject.com/script/Membership/View.aspx?mid=1339013) | 6-Nov-06 23:58 |

All I can say, is Very good work. Thanks  

#####  IconsReview.com - large list of icons (both commercial and free) [modified] 
[Warren Stevens](https://www.codeproject.com/script/Membership/View.aspx?mid=240) | 6-Nov-06 13:22 |

If anyone is looking for stock icons, I maintain a large list of stock icon collections at:  
  
[http://www.iconsreview.com/](http://www.iconsreview.com/)[[^](http://www.iconsreview.com/)]  
  
There are two large lists. One lists commercial (i.e. "pay") collections, and one for free (i.e. "no cost") collections.  
  
There are also lists of custom icon designers, articles on icons, and tools for working with icons.  
  
**Please note that I do NOT sell any of the icon collections, or design services, listed on IconsReview. It is just a central list of links to other sites that either sell, or give away, icons.**  
  
Warren  
  
  
-- modified at 12:13 Tuesday 7th November, 2006  
  

*****Need Icons?** Huge list of Stock Icon collections (free and commercial): [www.IconsReview.com](http://www.iconsreview.com/)

  

#####  [Re: IconsReview.com - large list of icons (both commercial and free)](https://www.codeproject.com/Messages/1744809/Re-IconsReview-com-large-list-of-icons-both-commer) 
[5150.Net](https://www.codeproject.com/script/Membership/View.aspx?mid=741345) | 6-Nov-06 23:05 |


BOOO!!! Not cool to try and sell your products here! We all could have done without knowing you have "pay" collections.  
  

_Thanks for firing me Darrell Eaker of Ringgold Telephone Company. I TRIPLED my salary and don't have to deal with douchebags like you. ![Smile | :)](https://codeproject.global.ssl.fastly.net/script/Forums/Images/smiley_smile.gif)_ 

  

##### [Re: IconsReview.com - large list of icons (both commercial and free)](https://www.codeproject.com/Messages/1745829/Re-IconsReview-com-large-list-of-icons-both-commer) 
[Warren Stevens](https://www.codeproject.com/script/Membership/View.aspx?mid=240) | 7-Nov-06 9:54 |


You obviously didn't even look at the site before commenting. ![Sigh | :sigh:](https://codeproject.global.ssl.fastly.net/script/Forums/Images/smiley_sigh.gif)  
  

> 5150.Net wrote:
> 
> Not cool to try and sell your products here!

  
  
I do **NOT** sell any icons or do any icon design work (on IconsReview.com or anywhere else). The site is simply a list of the many sources of icons. There is a list of commercial icon collections (again, I do NOT sell any of them) but there is also a huge list of free icons.  
  
Like most things in life, the commercial sets are generally of better quality than the free sets, but there are some exceptions to this rule. If you read the "Free Stock Icon Collections" paragraph on the home page, you will see a list of the free collections that are of high quality.  
  
I spent a lot of time making up the site, and I've had plenty of positive feedback and thanks for creating it. If you're not interested in using the site, please just ignore the post and move on...  
  

*****Need Icons?** Huge list of Stock Icon collections (free and commercial): [www.IconsReview.com](http://www.iconsreview.com/)

  

#####  [Re: IconsReview.com - large list of icons (both commercial and free)](https://www.codeproject.com/Messages/1745940/Re-IconsReview-com-large-list-of-icons-both-commer) 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 7-Nov-06 10:42 |


I'm really against trying to sell products on CodeProject, When I saw the post I opened the site and it doesn't sell anything, it is a web site of different URLs where you can find professional icons for sale and for free.  
  
For that reason I didn't reply to his first post, in fact I though to give thanks, because an icon library without icons to open is kind of empty.  


#####  [Re: IconsReview.com - large list of icons (both commercial and free)](https://www.codeproject.com/Messages/1746043/Re-IconsReview-com-large-list-of-icons-both-commer) 
[5150.Net](https://www.codeproject.com/script/Membership/View.aspx?mid=741345) | 7-Nov-06 11:26 |


Thanks for editing your post to indicate it's not a solicitation. I'm sure others would appreciate it as I have. I'm sorry if I came off so strongly but I'm against any solicitation of any kind unless I ask for it ![Big Grin | :-D](https://codeproject.global.ssl.fastly.net/script/Forums/Images/smiley_biggrin.gif)  
  
Great work on the list btw  


#####  [Re: IconsReview.com - large list of icons (both commercial and free)](https://www.codeproject.com/Messages/1790496/Re-IconsReview-com-large-list-of-icons-both-commer) 
[Pascal Ganaye](https://www.codeproject.com/script/Membership/View.aspx?mid=455825) | 5-Dec-06 6:22 |


To be fair, I thought this post was legitimate.  
It is directly related to the content of the article and the title is not misleading.  
Give him a break, it is hard to make a website when you have to compete with bigger and bigger corporation websites.  

#####  looks like xactly what I need... 
[Bjornar](https://www.codeproject.com/script/Membership/View.aspx?mid=163806) | 4-Nov-06 18:15 |

Let you now, when tried. ![Smile | :)](https://codeproject.global.ssl.fastly.net/script/Forums/Images/smiley_smile.gif)  

##### What about CUR and ANI? 
[dickinson.jonathan](https://www.codeproject.com/script/Membership/View.aspx?mid=2716244) | 4-Nov-06 4:38 |

Hey man... GOOD WORK! I'm very impressed. I've tried making a ICO library myself, but failed. Have you considered adding CUR and ANI files to the library? I'm trying to develop a free icon and cursor editor (Paint.Net with IcoCur works, but it's not dedicated) and I don't feel like writing the Ico and Cur libraries myself... Rather stick with the fun stuff ![Smile | :)](https://codeproject.global.ssl.fastly.net/script/Forums/Images/smiley_smile.gif) .  

##### [Re: What about CUR and ANI?](https://www.codeproject.com/Messages/1741870/Re-What-about-CUR-and-ANI) 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 4-Nov-06 14:25 |


Yes, I have the idea of support CUR and ANI too.  
  
Really CUR has the same format as ICO but with a hotspot, If I'll release Cursors then it must supports ANI too, I kind don't like things at half, and because still I don't know anything about ANI, the cursors for know will sleep for a while.  
  
As soon I research more about ANIs, and see how they work then probably I'll support them too.  
  
If you feel on create an Icon Editor then you are free to use IconLib like the core for load and save different icon formats.  
  

#####  [Re: What about CUR and ANI?](https://www.codeproject.com/Messages/1742503/Re-What-about-CUR-and-ANI) 
[dickinson.jonathan](https://www.codeproject.com/script/Membership/View.aspx?mid=2716244) | 5-Nov-06 13:05 |


Tops... Thanks alot man, I'll be sure to include you in the credits etc...  

#####  Brillant! 
[Mr. VB.NET](https://www.codeproject.com/script/Membership/View.aspx?mid=1389599) | 3-Nov-06 18:17 |

Suppose you did this on your lunch break? ![WTF | :WTF:](https://www.codeproject.com/script/Forums/Images/smiley_WTF.gif)  

#####  [Re: Brillant!](https://www.codeproject.com/Messages/1741858/Re-Brillant) 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 4-Nov-06 14:15 |


Really was more like 5 gallons of coffee ![Sleepy | :zzz:](https://www.codeproject.com/script/Forums/Images/smiley_snore.gif) at 1AM... every night ![Laugh | :laugh:](https://www.codeproject.com/script/Forums/Images/smiley_laugh.gif)  


#####  Fantastic work!!! 
[rick_montana](https://www.codeproject.com/script/Membership/View.aspx?mid=3509756) | 2-Nov-06 15:46 |

This is exactly what I was looking for.  
  
Definitely you got my 5.  
  
Thanks for it and keep the good work  
  
Rick.  

#####  [Re: Fantastic work!!!](https://www.codeproject.com/Messages/1739139/Re-Fantastic-work) 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 2-Nov-06 17:49 |


Thank you, I'll be updating the library with some extra features soon.  
  

#####  Good work! 
[cowlick](https://www.codeproject.com/script/Membership/View.aspx?mid=399291) | 1-Nov-06 21:22 |

Excellent article on a topic that is hard to find quality information for. I had to tackle this problem several months ago and wound up creating a library very similar to this...I suppose I could have saved you a lot of work by posting my solution! ![Poke tongue | ;-P](https://codeproject.global.ssl.fastly.net/script/Forums/Images/smiley_tongue.gif) Unfortunately most of my code is developed on the clock under a NDA so that's not usually an option. But thanks for posting your comprehensive solution, as I'm sure it will help a lot of people out.  

#####  [Re: Good work!](https://www.codeproject.com/Messages/1737443/Re-Good-work) 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 1-Nov-06 21:48 |


This started like something I needed for my job, just how to create a simple icon with two images on the fly, when I reached that objective with a very simple class at that time, then it got converted in my personal quest, so I kept the ownership of the library because was not created for my work, else now it is used in my work.  
  

#####  Im wondering... 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 1-Nov-06 9:00 |

I'm wondering who vote 1 for the project without leave any post at all and why a 1?  
  
I had 17 votes with 5, and the vote 18 was a 1, and now the average fall to 4.7.  
  
These are the things that make me sick, personally I think you should see who is voting, this kind of people shouldn't be allowed to vote.  
  
I could agree a 4 or a 3... but a 1 was because the guy wanted to be mean and probably didn't read the article at all... Probably his wife is cheating at him, he knows it and he come to codeproject to discharge voting at 1 with all new projects. ![Mad | :mad:](https://codeproject.global.ssl.fastly.net/script/Forums/Images/smiley_mad.gif)  
  

#####  [Re: Im wondering...](https://www.codeproject.com/Messages/1737003/Re-Im-wondering) 
[Eric Engler](https://www.codeproject.com/script/Membership/View.aspx?mid=337860) | 1-Nov-06 13:21 |


Please don't worry about the odd cases like that. Your work is incredible and I'm sure your rating will reflect that.  
  
You can never please everyone. Some people won't like your choice of language, some won't like Vista or Microsoft, some will be beginners with .NET or Windows and won't understand your article, some will be web developers who are pissed that these icons don't work in web pages, ...  
  
Just let this ride for a while and you'll see that you have one of the highest ratings here. I'm sure of that...  
  

#####  [Re: Im wondering...](https://www.codeproject.com/Messages/1784322/Re-Im-wondering) 
[henok](https://www.codeproject.com/script/Membership/View.aspx?mid=896861) | 30-Nov-06 17:58 |


it can also be a mistake.. they wanted to give a 5.. you got my 5.. i was trying to click on 6.. but could not ![Smile | :)](https://codeproject.global.ssl.fastly.net/script/Forums/Images/smiley_smile.gif)  

#####  [Re: Im wondering...](https://www.codeproject.com/Messages/1737090/Re-Im-wondering) 
[Jörgen Sigvardsson](https://www.codeproject.com/script/Membership/View.aspx?mid=15093) | 1-Nov-06 14:23 |


I just gave you a 5 to try to offset the 1. For what it's worth, it is a nice article. I loved the part with the "ICL mystery". ![Smile | :)](https://codeproject.global.ssl.fastly.net/script/Forums/Images/smiley_smile.gif)


#####  [Re: Im wondering...](https://www.codeproject.com/Messages/1744442/Re-Im-wondering) 
[qiuyl](https://www.codeproject.com/script/Membership/View.aspx?mid=60716) | 6-Nov-06 16:04 |


That's wonderful! I firmly believe it was a mistake vote made by mistake!! Never mind if someboby intendly vote for one.  

#####  [Re: Im wondering...](https://www.codeproject.com/Messages/1745925/Re-Im-wondering) 
[DaveJS](https://www.codeproject.com/script/Membership/View.aspx?mid=749981) | 7-Nov-06 10:37 |


Here is another 5 vote  

##### [Re: Im wondering...](https://www.codeproject.com/Messages/1745951/Re-Im-wondering) 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 7-Nov-06 10:45 |

Thanks my friend. 


##### [Re: Im wondering...](https://www.codeproject.com/Messages/1773125/Re-Im-wondering) 
[Anmum](https://www.codeproject.com/script/Membership/View.aspx?mid=895184) | 23-Nov-06 13:40 |


But maybe it was only a mistake from the person voting. In some countries the school degrees range from 1 to 5 or from 1 to 6 with 1 for the best and 5/6 for the worst grade. So don't worry and think of it in this way as long as you can prove the opposite 
![Big Grin | :-D](https://www.codeproject.com/script/Forums/Images/smiley_biggrin.gif)  

##### [Re: Im wondering...](https://www.codeproject.com/Messages/1824918/Re-Im-wondering) 
[Oakman](https://www.codeproject.com/script/Membership/View.aspx?mid=420158) | 2-Jan-07 10:10 |


There are jerks everywhere. Don't worry about it - and I gave you a 5, too.  
  

Jon  
Information doesn't want to be free.  
It wants to be sixty-nine cents @ pound.

  

##### [Re: Im wondering...](https://www.codeproject.com/Messages/3441297/Re-Im-wondering) 
[are_all_nicks_taken_or_what](https://www.codeproject.com/script/Membership/View.aspx?mid=85475) | 16-Apr-10 6:45 |


In many countries a grade of 1 is the best, whereas a grade of 5 is the worst. I'd say that the grading system is ambiguous!  

##### Excellent! 
[Alexey A. Popov](https://www.codeproject.com/script/Membership/View.aspx?mid=250564) | 1-Nov-06 6:56 |

Wow!.. I'm stunned!.. An AMAIZING piece of work! I'd give you 20 if I could!  

##### [Re: Excellent!](https://www.codeproject.com/Messages/1736569/Re-Excellent) 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 1-Nov-06 8:12 |

I'm ok with a 5 for now ![Big Grin | :-D](https://www.codeproject.com/script/Forums/Images/smiley_biggrin.gif)  
   

##### license 
[André Ziegler](https://www.codeproject.com/script/Membership/View.aspx?mid=212073) | 1-Nov-06 3:46 |

Thx, your IconLib is fantastic. I gave you 5 Points 
  
Can I use it in own projects?  


##### [Re: license](https://www.codeproject.com/Messages/1736563/Re-license) 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 1-Nov-06 8:10 |


Yes, it is a BSD style license, this means you can use it on your free/comercial projects as long the copyright headers stay where they are.  
  

##### [Re: license](https://www.codeproject.com/Messages/1736862/Re-license) 
[André Ziegler](https://www.codeproject.com/script/Membership/View.aspx?mid=212073) | 1-Nov-06 11:31 |

thx 
  

##### [Re: license](https://www.codeproject.com/Messages/1740637/Re-license) 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 3-Nov-06 12:52 |

I updated the article page with the license link and logo, but basically is like I told you before.  


##### Wonderful work! 
[scott_hackett](https://www.codeproject.com/script/Membership/View.aspx?mid=508756) | 1-Nov-06 0:10 |

Wow! I haven't tried the sample yet, just read through your article. I've done a lot of work with icons myself and it's way more challenging than it appears on the surface. Especially supporting ICL... I spend weeks trying to do the same thing and gave up in frustration. Your article is fantastic and the supporting diagrams are perfect. Great work, you definitely got my 5!  

##### [Re: Wonderful work!](https://www.codeproject.com/Messages/1735973/Re-Wonderful-work) 
[CastorTiu](https://www.codeproject.com/script/Membership/View.aspx?mid=240897) | 1-Nov-06 0:22 |


Thank you very much.  
  
I'll keep updating the library with new formats whenever I can get the internal design from some place, they are not easy to find 
