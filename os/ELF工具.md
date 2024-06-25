# 可执行文件分析工具程序
{docsify-updated}

1. `objdump` -用来查看目标文件或库文件的信息 ，`objdump option <filename>`
   + `-h`: 查看目标文件的各个段的基本信息。
   + `-s`: 将各个段的内容以十六进制打印出来
   + `-d`: 将包含指令的段反汇编打印出来 
   + `-D`: 反编译所有段
   + `-r`: 查看重定位表
   + `-t`: 查看符号表
2. `dumpbin` -用来查看windows 下的目标文件格式
	+ `/SUMMARY`：查看段信息
	+ `/SYMBOLS`：查看符号表
	+ `/RELOCATIONS`：查看重定位表
	+ `/HEADERS`：
3. `size` -查看目标文件各个段的长度
4. `readelf` -查看elf 文件的信息
	+ `-h/--file-header`: Display the ELF file header  ELF文件头
	+ `-l/--program-headers`: Display the program headers 程序头表
	+ `-S/--section-headers`: Display the sections' header 节头表
	+ `-s/--syms`: Display the symbol table 符号表
	+ `-d --dynamic`: Display the dynamic section (if present) 动态信息表
	+ `-r -r --relocs`: Display the relocations 重定位表
5. `ar` -压缩/解压程序，通常用来将若干目标文件打包成一个库文件
   + `-t`：查看库文件中包含哪些目标文件 `ar -t libc.a`
   + `-x`:解压打包的文件 `ar -x libc.a`
6. `file`-查看文件的简要信息，`apt install file`
7. `hexdump -C -n 64 a.out` -观察二进制文件的二进制内容前64字节
8. `ldd hello` - 查看一个程序主模块或一个共享库依赖于哪些共享库
9. pax-utils-`apt install pax-utils`
   + `/usr/bin/dumpelf` – dump internal ELF structure
   + `/usr/bin/lddtree` – like ldd, with levels to show dependencies
   + `/usr/bin/pspax` – list ELF/PaX information about running processes
   + `/usr/bin/scanelf` – wide range of information, including PaX details
   + `/usr/bin/scanmacho` – shows details for Mach-O binaries (Mac OS X)
   + `/usr/bin/symtree` – displays a leveled output for symbols
10. [Radare2](https://www.radare.org/r/)
11. elfutils -`sudo apt install elfutils -y`
    + /usr/bin/eu-addr2line
	+ /usr/bin/eu-ar – alternative to ar, to create, manipulate archive files
	+ /usr/bin/eu-elfcmp
	+ /usr/bin/eu-elflint – compliance check against gABI and psABI specifications
	+ /usr/bin/eu-findtextrel – find text relocations
	+ /usr/bin/eu-ld – combining object and archive files
	+ /usr/bin/eu-make-debug-archive
	+ /usr/bin/eu-nm – display symbols from object/executable files
	+ /usr/bin/eu-objdump – show information of object files
	+ /usr/bin/eu-ranlib – create index for archives for performance
	+ /usr/bin/eu-readelf – human-readable display of ELF files
	+ /usr/bin/eu-size – display size of each section (text, data, bss, etc)
	+ /usr/bin/eu-stack – show the stack of a running process, or coredump
	+ /usr/bin/eu-strings – display textual strings (similar to strings utility)
	+ /usr/bin/eu-strip – strip ELF file from symbol tables
	+ /usr/bin/eu-unstrip – add symbols and debug information to stripped binary
12. elfkickers
    + /usr/bin/ebfc – compiler for Brainfuck programming language
	+ /usr/bin/elfls – shows program headers and section headers with flags
	+ /usr/bin/elftoc – converts a binary into a C program
	+ /usr/bin/infect – tool to inject a dropper, which creates setuid file in /tmp
	+ /usr/bin/objres – creates an object from ordinary or binary data
	+ /usr/bin/rebind – changes bindings/visibility of symbols in ELF file
	+ /usr/bin/sstrip – strips unneeded components from ELF file
13. prelink
    + /usr/bin/execstack – display or change if stack is executable
	+ /usr/bin/prelink – remaps/relocates calls in ELF files, to speed up the process
   
### objdump 和 readelf 的区别
/* The difference between readelf and objdump:

   Both programs are capabale of displaying the contents of ELF format files,
   so why does the binutils project have two file dumpers ?

   The reason is that objdump sees an ELF file through a BFD filter of the
   world; if BFD has a bug where, say, it disagrees about a machine constant
   in e_flags, then the odds are good that it will remain internally
   consistent.  The linker sees it the BFD way, objdump sees it the BFD way,
   GAS sees it the BFD way.  There was need for a tool to go find out what
   the file actually says.

   This is why the readelf program does not link against the BFD library - it
   exists as an independent program to help verify the correct working of BFD.

   There is also the case that readelf can provide more information about an
   ELF file than is provided by objdump.  In particular it can display DWARF
   debugging information which (at the moment) objdump cannot.  */

BFD（Binary File Descriptor）过滤器是 GNU Binutils 项目中的一个库，用于统一处理不同二进制文件格式（如 ELF、COFF、a.out 等）的抽象层。BFD 提供了一套通用接口，使得工具如链接器、反汇编器和调试器可以通过这个统一接口处理各种格式的二进制文件，而不必关心底层文件格式的具体细节。

通过 BFD 过滤器，工具可以：
+ 读取和写入不同格式的二进制文件：BFD 使得工具可以透明地处理不同格式的二进制文件，隐藏了文件格式的复杂性。
+ 转换和处理目标文件：BFD 可以处理各种目标文件的细节，包括符号表、重定位信息、调试信息等。
+ 保持内部一致性：所有通过 BFD 库处理的工具（如 objdump、链接器、汇编器）都使用相同的内部表示和常量，这确保了这些工具之间的行为一致。
由于 BFD 层的抽象和一致性处理，可能会出现一些情况下，实际的文件内容与通过 BFD 接口看到的内容存在差异。这就是为什么需要一个独立的工具如 readelf，它直接读取并显示 ELF 文件的实际内容，不经过 BFD 的处理，以便于验证和调试 BFD 库的正确性。  
总之，BFD 过滤器在处理二进制文件格式时提供了统一的抽象接口和一致性处理，但有时也需要像 `readelf` 这样的工具来验证其输出的准确性。

一般C语言的编译后执行语句都编译成机器代码，保存在.text段；已初始化的全局变量和局部静态变量都保存在.data段；未初始化的全局变量和局部静态变量一般放在一个叫“.bss”的段里.

最基本的代码段(.text)、数据段(.data)和BSS(.bss)段以外，还有3个段分别是只读数据段（.rodata）、注释信息段（.comment）和堆栈提示段（.note.GNU-stack），这3个额外的段的意义我们暂且不去细究。先来看看几个重要的段的属性，其中最容易理解的是段的长度（Size）和段所在的位置（File Offset），每个段的第2行中的“CONTENTS”、“ALLOC”等表示段的各种属性，“CONTENTS”表示该段在文件中存在。我们可以看到BSS段没有“CONTENTS”，表示它实际上在ELF文件中不存在内容。“.note.GNU-stack”段虽然有“CONTENTS”，但它的长度为0，这是个很古怪的段，我们暂且忽略它，认为它在ELF文件中也不存在
