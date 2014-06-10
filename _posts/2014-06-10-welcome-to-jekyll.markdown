---
layout: post
title:  "ELF Format for OpenCL Executables"
author: Nathan Lilienthal
date:   2014-06-10 15:07:04
categories: software C
---

There are a few object formats in common use. The most common format I encounter is the Mach-O format, given I run on Mac OS X. On Linux however the standard is ELF. Here I'll be discussing the ELF format, and what data it contains within the scope of running a OpenCL program.

### Device Program Object (*.bin)
The device program binary is much smaller than a typical executable. At least the binaries we are creating do not have dynamic libraries, or data sections. There are other things missing too, but those 2 are the most noteworthy to me.

#### ELF Header
Along with a bunch of other data the ELF header gives us a hint that we aren't operating on a standard system. Hypothetically 0x7d2 means something to someone.

| Field   | Value                  |
| ------- | ---------------------- |
| Type    | EXEC (Executable file) |
| Machine | \<unknown\>: 0x7d2     |

#### Sections

`.shstrtab` Holds section header names.

`.strtab` Holds symbol names.

`.comment` @(#) OpenCL 1.2 AMD-APP (923.1).  Driver version: 2.0 (sse2)

`.llvmir`
This section contains the LLVM intermediate representation of the program. This is used to create either the CPU or GPU specific representation. There are flags you can pass to compile this object file without saving this information.

`.text`

{% highlight sh %}
# I need to look into the contents on the .text section.
$ readelf <PROGRAM>.bin -S | grep '\.text'
  [ 6] .text             PROGBITS        00000000 000c00 00143b 00  AX  0   0  1

# Ok so it's at offset 000c00 (aka 3072 bytes) and it's 00143b (aka 5179 bytes) long.

# Copy the text section to a file.
$ dd if=<PROGRAM>.bin of=text.dump bs=1 skip=3072 count=5179

$ readelf text.dump -h
ELF Header:
  ...
  Type:                              DYN (Shared object file)
  Machine:                           Intel 80386
  ...

# Holy crap it's another ELF. But it's for Intel 80386, that doesn't sound
# like our GPU... Oh wait, we compiled our OpenCL code for the CPU!
{% endhighlight %}

#### Symbols

Each __kernel function gets 3 FUNC entries in the symbol table.

- `__OpenCL_<FUNCTION NAME>_kern`
- `__OpenCL_<FUNCTION NAME>_stub`
- `__OpenCL_<FUNCTION NAME>_meta`

My limited understanding of these sections is mostly a guess, but since the host calls these functions using the combination of calls to `clSetKernelArg` and `clCreateKernel` it is going outside the traditional function call methodologies. This requires the information about the kernel function to be passed to the host in another way. I'm assuming these 3 functions make up the needed information for the host to type check and handle value assignment/allocation. It's very hard to read through the `disas` for these functions due to their non standard instruction sets. But I'm also guessing that the stub function is a function that takes no arguments and sets up the call to the actual function with it's arguments initialized. `clSetKernelArg` is effectively setting up the values the stub function will pass to the actual kernel. I'm guessing this also handles delegating to the proper device code, since the `.text` section has dynamic libraries for specific architectures.

I'm not sure if it is possible to expose a variable to the symbol table. Trying a few things like defining a variable in global space simply gave me compile errors.

#### Program Headers

None.


### Host Program Object (no extension)
This is the result of compiling the host program. In our case the target of the Makefile. Being a typical C program on a UNIX system, it makes use of a few shared libraries, including OpenCL. This means it's runtime and ELF format are more complex.

#### ELF Header
Cool we are working on a typical Intel system.

| Field   | Value                  |
| ------- | ---------------------- |
| Type    | EXEC (Executable file) |
| Machine | Intel 80386            |

#### Sections

`.shstrtab` Holds section header names.

`.strtab` Holds symbol names.

`.dynstr` Holds symbols from dynamic libraries.

`.comment` GCC: (Ubuntu/Linaro 4.6.1-9ubuntu3) 4.6.1.

`.debug_*` Debugging sections because I compile with -g.

`.plt`
Procedure Linkage Table, This table stores the addresses of functions in dynamic libraries as defined by the dynamic loader. These values are updated in the `.got` as the functions are called to save time loading addresses that may never be used. This area is read only.

`.got` See above.

`.text` Program data, read only.

`.data`  
To look into this section a little bit I added a variable to the top of the program in global memory.

{% highlight c++ %}
int nixpulvis = 1337;

int main(int argc, char const *argv[]) {
  ...
}
{% endhighlight %}

Next I looked into the resulting executable after compiling.

{% highlight bash %}
# The ELF header tells us about how to read this data.
$ readelf <PROGRAM> -h | grep 'Data'
#>  Data:                              2's complement, little endian

# Ok so what is the data?
$ readelf <PROGRAM> -x .data

#> Hex dump of section '.data':
#>   0x0804c098 00000000 00000000 39050000          ........9...

# 3905 (little endian) -> 0539 (big endian) == 1337 base 10.

# Looking in the symbol table will show us the full story.
$ readelf <PROGRAM> -s | grep 'nixpulvis'
#>    96: 0804c0a0     4 OBJECT  GLOBAL DEFAULT   24 nixpulvis

# That's exactly the 4 bytes in the data section where our variable is. Cool!
{% endhighlight %}

#### Symbols
The host program also has many more symbols. Some are dynamically loaded. These symbols are not loaded and accessible to the program until runtime. These symbols are defined in .dynsym. Whereas the symbols defined in .symtab are part of the program itself. In order for the compiler to execute successfully the program must have the headers and those symbols are included, but marked as UND for undefined. Symbols for functions defined in shared libraries seem to have '@@' delimit the function name, and the library name, for example `free@@GLIBC_2.0`.

#### Program Headers

`INTERP`
/lib/ld-linux.so.2 i.e. the dynamic loader. This is the entry point for this program to allow it to first load the shared libraries, then it passes control back to the original entry point of the program.
