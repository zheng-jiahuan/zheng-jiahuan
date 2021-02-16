Thread Control Block  TCB

https://docs.oracle.com/cd/E23824_01/html/819-0690/gentextid-22428.html#scrolltoc


## Chapter 2 Link Editor
### Symbol Processing
> During input file processing, local symbols are copied from any input relocatable object files to the output object being built, **without examination.**

> The global symbols from all input relocatable objects, and the global symbols from any external dependencies, are analyzed and combined in a process known as **symbol resolution**. The link-editor **places  each symbol in an internal symbol table** in the order that the symbols are encountered. If a symbol with the same name was contributed by an earlier object, and already exists in the symbol table, the symbol resolution process **determines which** of the two symbols to keep. As a side effect of this process, the link-editor determines how to establish references to external object dependencies.

#### Symbol Resolution
> The most common simple resolutions involve binding symbol references from one object to symbol definitions within another object. This binding can occur between two relocatable objects, or between a relocatable object and the first definition found in a shared object dependency.

three basic symbol types

1. **Undefined** – Symbols that have been referenced in a file but have not been assigned a storage address.

2. **Tentative** – Symbols that have been created within a file but have not yet been sized, or allocated in storage. These symbols appear as uninitialized C symbols, or FORTRAN COMMON blocks within the file.

3. **Defined** – Symbols that have been created, and assigned storage addresses and space within the file.

>In its simplest form, symbol resolution involves the use of a **precedence** relationship. This relationship has **defined** symbols dominate **tentative** symbols, which in turn dominate **undefined** symbols


## Runtime Linker 

> As part of the initialization and execution of an executable, an **interpreter** is called to complete the binding of the application to its dependencies. In the Oracle Solaris OS, this interpreter is referred to as the **runtime linker**.
During the link-editing of an executable, **a special .interp section**, together with an associated program header, are created. This section contains a path name specifying the program's interpreter. The default name supplied by the link-editor is the name of the runtime linker: /usr/lib/ld.so.1 for a 32-bit executable and /usr/lib/64/ld.so.1 for a 64-bit executable.
During the process of **executing a dynamic object**, the kernel loads the file and reads the program header information. See “Program Header” on page 435. From this information, **the kernel locates the name of the required interpreter. The kernel loads, and transfers control to this interpreter, passing sufficient information to enable the interpreter to continue executing the
application.**


runtime linker的功能

The runtime linker performs the following actions.
1. Analyzes the executable's dynamic information section (.dynamic) and determines what
dependencies are required.
2. Locates and loads these dependencies, analyzing their dynamic information sections to
determine if any additional dependencies are required.
3. Performs any necessary relocations to bind these objects in preparation for process
execution.
4. Calls any initialization functions provided by the dependencies.
5. Passes control to the application.
6. Can be called upon during the application's execution, to perform any delayed function binding.
7. Can be called upon by the application to acquire additional objects with dlopen(3C), and bind to symbols within these objects with dlsym(3C).

runtime linker查找路径：
1. 首先查找的是 LD_LIBRARY_PATH这个环境变量指定的目录下的so
2. 如果步骤1中没有搜索到，那则搜索runpath目录下的so。link-edit阶段写入object中.dynamic段中的某个entry，这个entry的tag是RUNPATH，指定了搜索路径
3. 如果步骤2中也没有搜索到，那么就搜索默认路径，32bit系统默认搜索/lib and /usr/lib, 64bit系统默认搜索/lib/64 and /usr/lib/64路径

the use of LD_LIBRARY_PATH is strongly discouraged in production software

### Relocation Processing

>After the runtime linker has loaded all the dependencies required by an application, the linker processes each object and performs all necessary relocations.

>During the link-editing of an object, any relocation information supplied with the input
relocatable objects is applied to the output file. However, when creating a dynamic object,
many of the relocations **cannot be completed** at link-edit time. These relocations require logical
addresses that are known only when the objects are loaded into memory. In these cases, the
**link-editor** generates **new relocation records** as part of the output file image. The **runtime linker**
must then **process** these new relocation records.

Two basic types of relocation exist.
1. Non-symbolic relocations
2. Symbolic relocations







With this model, the first occurrence of the required symbol satisfies the search. Therefore, if more than one instance of the same symbol exists, the first instance interposes on all others.

### Initialization and Termination Routines

Dynamic objects can supply code that provides for runtime initialization and termination
processing. The initialization code of a dynamic object is executed once each time the dynamic
object is loaded in a process. The termination code of a dynamic object is executed once each
time the dynamic object is unloaded from a process or at process termination.


Before transferring control to an application, the runtime linker processes any initialization
sections found in the application and any loaded dependencies. If new dynamic objects are
loaded during process execution, their initialization sections are processed as part of loading the
object. The initialization sections .preinit_array, .init_array, and .init are created by the
link-editor when a dynamic object is built.

An executable can provide pre-initialization functions in a .preinit_array section. These
functions are executed after the runtime linker has built the process image and performed
relocations but before any other initialization functions. Pre-initialization functions are not
permitted in shared objects.
Note - Any .init section within the executable is called from the application by the process
startup mechanism supplied by the compiler driver. The .init section within the executable is
called last, after all dependency initialization sections are executed.

## Shared Objects
> Shared objects are one form of output created by the link-editor and are generated by specifying
the -G option.
A shared object is an indivisible unit that is generated from one or more relocatable objects.
Shared objects can be bound with executables to form a runnable process. As their name
implies, shared objects can be shared by more than one application

> Any input shared objects become dependencies of this output file. A small amount of
bookkeeping information is maintained within the output file to describe these dependencies.
The runtime linker interprets this information and completes the processing of these shared
objects as part of creating a runnable process.


During the link-edit of a shared object, its runtime name can be recorded within the shared
object itself by using the -h option. In the following example, the shared object's runtime name
libfoo.so.1, is recorded within the file itself. This identification is known as an soname.

#### Relocation Sections
>**Relocation is the process of connecting symbolic references with symbolic definitions**. For
example, when a program calls a function, **the associated call instruction must transfer control to the proper destination address at execution**. Relocatable files must have information that
describes how to modify their section contents. This information allows dynamic object files to **hold the right information** for a process's program image. **Relocation entries are these data**.

sys/elf.h
```
typedef struct {
        Elf32_Addr      r_offset;
        Elf32_Word      r_info;
} Elf32_Rel;

typedef struct {
        Elf32_Addr      r_offset;
        Elf32_Word      r_info;
        Elf32_Sword     r_addend;
} Elf32_Rela;

typedef struct {
        Elf64_Addr      r_offset;
        Elf64_Xword     r_info;
} Elf64_Rel;

typedef struct {
        Elf64_Addr      r_offset;
        Elf64_Xword     r_info;
        Elf64_Sxword    r_addend;
} Elf64_Rela;
```
**r_offset**
This member gives the location at which to apply the relocation action. Different object files have slightly different interpretations for this member.

For a relocatable file, the value indicates a section offset. The relocation section describes how to modify another section in the file. Relocation offsets designate a storage unit within the second section.
就是说，这个offset描述了如何修改另一个段中offset处的一块内存区域。

For a dynamic object, the value indicates the virtual address of the storage unit affected by the relocation. This information makes the relocation entries more useful for the runtime linker.
对于动态对象来说，这个值指示了受重定位影响的存储单元的虚拟地址，对runtime linker来说，该重定位条目非常有用。


Although the interpretation of the member changes for different object files to allow efficient access by the relevant programs, the meanings of the relocation types stay the same.


### Program Loading and Dynamic Linking

Executables and shared objects **statically represent** application programs. To execute such programs, the system uses the objects to create **dynamic program representations, or process images**. A process image has segments that contain its text, data, stack, and so on.


####  Base Address
Dynamic objects have a base address, which is the lowest virtual address associated with the memory image of the program's object file. One use of the base address is to **relocate** the memory image of the program during dynamic linking.

**The base address of a dynamic object is calculated during execution from three values**: the memory load address, the maximum page size, and the lowest virtual address of a program's loadable segment. The virtual addresses in the program headers might not represent the actual virtual addresses of the program's memory image.
在执行期间，一个动态对象的基址是根据这三个值计算出来的*****

#### Segment Contents
An object file segment consists of one or more sections, though this fact is transparent to the program header. Whether the file segment holds one section or many sections, is also immaterial to program loading. 

### Program Loading

As the system creates or augments a process image, the system **logically** copies a file's segment to a virtual memory segment. When, and if, the system **physically** reads the file depends on the program's execution behavior, system load, and so forth.

A process does not require a physical page **unless** the process **references** the logical page during execution. Processes commonly leave many pages **unreferenced**. Therefore, delaying physical reads can improve system performance. To obtain this efficiency in practice, dynamic objects
must have segment images whose file offsets and virtual addresses are **congruent**, **modulo** the page size.


















----------------------------

1. Historically, weak symbols have been used to **circumvent interposition**, or test for optional functionality. 
2. Although this archive extraction can be
achieved by specifying multiple -u options to the link-edit, this example also shows how the **eventual scope** of a symbol can be reduced to local.

These environment variables **are well suited to** debugging purposes, such as forcing an application to bind to a local dependency.

However, this compensation can undermine the advantages of a lazy loading.

Neither the link-editor nor the runtime linker interprets any file by virtue of its file name

Whether the file segment holds one section or many sections, is also
**immaterial** to program loading.






error: RPC failed; curl 56 GnuTLS recv error (-9): Error decoding the received TLS packet.

fatal: The remote end hung up unexpectedly. fatal: early EOF fatal: index-pack failed

乱码的解决方案
alias tree='tree --charset ASCII'

驱动处理方式
http://blog.leanote.com/post/yangtianrui95@gmail.com/AOSP-%E5%BC%80%E5%8F%91%E5%B9%B6%E5%88%B7%E5%85%A5Pixel


tar zxvf qcom-walleye-qq3a.200705.002-b7555c81.tgz

tar   zxvf  google_devices-walleye-qq3a.200705.002-42602ba1.tgz


m -j8











