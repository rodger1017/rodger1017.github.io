---
title: Mach-O 文件解析
layout: posts
categories: iOS
tag: objc
---

## 引言
Mach-O，是Mach object文件格式的缩写，是一种用于记录可执行文件、对象代码、共享库、动态加载代码和内存转储的文件格式，是macOS/iOS上程序以及库的标准格式。类似于windows上的PE格式 (Portable Executable )， linux上的elf格式 (Executable and Linking Format)。

MacOS 支持三种可执行格式：解释器脚本格式、通用二进制格式和 Mach-O 格式。

| 可执行格式 | magic | 说明 |
| - | - | - |
| 脚本 | \x7FELF | 主要用于 shell 脚本，但是也常用语其他解释器，如 Perl, AWK 等。也就是我们常见的脚本文件中在 #! 标记后的字符串，即为执行命令的指令方式，以文件的 stdin 来传递命令 |
| 通用二进制格式 | 0xcafebabe 0xbebafeca | 包含多种架构支持的二进制格式 |
| Mach-O | 0xfeedface（32 位）0xfeedfacf（64 位）| macOS 的原生二进制格式 |

通用二进制文件(FatFile/FatBinary)是一个由不同的编译架构后的Mach-O产物所合成的集合体。一个架构的mach-O只能在相同架构的机器或者模拟器上用，为了支持不同架构需要一个集合体。

![Fat-File]({{ site.baseurl }}/images/mach-object-file/fat-file.jpg)

Mach-O 文件类型如下：

| 类型 | 作用 |
| - | - |
| Executable | 可执行二进制 |
| Dylib Library | 动态链接库(DSO/DLL) |
| Static Library | 静态链接库 |
| Bundle | 不能被链接的Dylib，只能在运行时使用dlopen()加载，可当做macOS的插件 |
| Relocatable Object File | 可重定向文件 |

## Mach-O执行步骤
可执行文件执行的主要步骤：
1. 操作系统读取可执行文件头部检验可执行文件的有效性，比如模数等；
2. 通过 LCL_LOAD_DYLINKER 结构，设置动态链接器的路径；
3. 根据 segment 结构开始对 Mach-o 进行映射
4. 初始化 Mach-o 进程环境
5. 将系统调用的返回地址设置为动态链接器的入口地址
6. 加载 LC_LOAD_DYLIB 中的动态库，进行转载时重定位
跳转到程序的入口地址开始执行

## Mach-O文件格式
Mach-O 为 Mach Object 文件格式的缩写，它是一种用于可执行文件，目标代码，动态库，内核转储的文件格式。文件基本格式如下图所示：

![Mach-O-Format]({{ site.baseurl }}/images/mach-object-file/mach-o-format.png)

每个 Mach-O 文件包括一个 Mach-O 头(Header)，然后是一系列的载入命令(Load commands)，再是一个或多个块(Segment)，每个块包括0到255个段(Section)。Mach-O 使用REL 再定位格式控制对符号的引用。Mach-O 在两级命名空间中将每个符号编码成“对象-符号名”对，在查找符号时则采用线性搜索法。

### Header
Mach-O 头描述了 Mach-O 的 CPU 架构、文件类型以及加载命令等信息；

```c
// /usr/include/mach-o/loader.h
/*
 * The 32-bit mach header appears at the very beginning of the object file for
 * 32-bit architectures.
 */
struct mach_header {
	uint32_t	magic;		/* mach magic number identifier */
	cpu_type_t	cputype;	/* cpu specifier */
	cpu_subtype_t	cpusubtype;	/* machine specifier */
	uint32_t	filetype;	/* type of file */
	uint32_t	ncmds;		/* number of load commands */
	uint32_t	sizeofcmds;	/* the size of all the load commands */
	uint32_t	flags;		/* flags */
};
/* Constant for the magic field of the mach_header (32-bit architectures) */
#define	MH_MAGIC	0xfeedface	/* the mach magic number */
#define MH_CIGAM	0xcefaedfe	/* NXSwapInt(MH_MAGIC) */
/*
 * The 64-bit mach header appears at the very beginning of object files for
 * 64-bit architectures.
 */
struct mach_header_64 {
	uint32_t	magic;		/* mach magic number identifier */
	cpu_type_t	cputype;	/* cpu specifier */
	cpu_subtype_t	cpusubtype;	/* machine specifier */
	uint32_t	filetype;	/* type of file */
	uint32_t	ncmds;		/* number of load commands */
	uint32_t	sizeofcmds;	/* the size of all the load commands */
	uint32_t	flags;		/* flags */
	uint32_t	reserved;	/* reserved */
};

/* Constant for the magic field of the mach_header_64 (64-bit architectures) */
#define MH_MAGIC_64 0xfeedfacf /* the 64-bit mach magic number */
#define MH_CIGAM_64 0xcffaedfe /* NXSwapInt(MH_MAGIC_64) */
```

- `magic`：魔数，用来标识 Mach-o 的平台属性的，确认文件的类型。操作系统在加载时会，确定魔数是否正确，不正确会被拒绝加载
- `cputype`：cpu 类型，比如arm
- `cpusubtype`：cpu 具体类型，比如armv7, arm64
- `filetype`：该文件的类型，比如 MH_EXECUTE 代表可执行文件，MH_DYLINKER 表明该文件是动态链接器程序文件
- `ncmds`：加载命令的数量
- `sizeofcmds`：所有加载命令的大小
- `flags`：标记该文件目前的状态信息，比如 MH_NOUNDEFS 表明该文件里没有未定义的符号引用，MH_DYLDLINK 表明该文件，已经完成了静态链接，不能再次被静态链接。

#### 示例
源码：helloword.c

```c
#include <stdio.h>

int main(int argc, const char * argv[]) {
    // insert code here...
    printf("Hello, World!\n");
    return 0;
}
```

编译：`clang -g helloworld.c`
输出：a.out
通过 [MachOView](https://github.com/gdbinit/MachOView) 查看a.out文件

![a-out]({{ site.baseurl }}/images/mach-object-file/a-out.png)

可以用 otool 命令进行分析：`otool -h 可执行文件`

```bash
➜  Test otool -h a.out
Mach header
      magic cputype cpusubtype  caps    filetype ncmds sizeofcmds      flags
 0xfeedfacf 16777223          3  0x80           2    15       1296 0x00200085
```

查看详细信息：`otool -hv 可执行文件`

```bash
➜  Test otool -hv a.out
Mach header
      magic cputype cpusubtype  caps    filetype ncmds sizeofcmds      flags
MH_MAGIC_64  X86_64        ALL LIB64     EXECUTE    15       1296   NOUNDEFS DYLDLINK TWOLEVEL PIE
```
 
### Load Commands
Load Commands 是紧跟 Header 之后的命令加载区，所有 commands 的大小总和即为 Header->sizeofcmds 字段，共有 Header->ncmds 条加载命令。对应定义如下：

```c
struct load_command {
    uint32_t cmd;        /* type of load command */
    uint32_t cmdsize;    /* total size of command in bytes */
};
```

Command 以 LC 开头，不同命令的结构体有差异，共有字段是命令类型(cmd)和命令长度(cmdsize)。这些加载命令告诉系统应该如何处理后面的二进制数据，对系统内核加载器和动态链接器起指导作用。

| 命令 | 作用 |
| - | - |
| LC_UUID | 唯一UUID，128位 |
| LC_SEGMENT | 可以加载到内存并映射到地址空间 |
| LC_SEGMENT_64 | 同上，64位 |
| LC_SYMTAB | 符号表信息 |
| LC_DYSYMTAB | 动态符号表信息 |
| LC_THREAD LC_UNIXTHREAD | 主线程初始化状态信息 |
| LC_LOAD_DYLIB | 动态库加载信息，包括动态库地址、名称、版本号等 |
| LC_ID_DYLIB | 动态库名称 |
| LC_PREBOUND_DYLIB | 依赖动态库信息 |
| LC_LOAD_DYLINKER | 指定加载该文件的动态链接器 |
| LC_ID_DYLINKER | 指定当前文件为动态链接器 |
| LC_ROUTINES | 初始化程序地址 |
| LC_ROUTINES_64 | 同上，64位 |
| LC_TWOLEVEL_HINTS | 两级命名空间查询提示表 |
| LC_SUB_FRAMEWORK | 标识为 `umbrella framework` 的 `subframework` |
| LC_SUB_UMBRELLA | 标识为 `umbrella framework` 的 `subumbrella` |
| LC_SUB_LIBRARY | 标识为 `umbrella framework` 的 `sublibrary` |
| LC_SUB_CLIENT | `framework` 或 `bundle` 可以通过 `LC_SUB_CLIENT` 显示链接一个 `subframework` |

可以用 otool 命令进行分析：`otool -l 可执行文件`

#### Segment & Section
Segment 名字一般全部大写，且以双下划线开头，例如 `__TEXT`，Section 名字全部为小写，例如 `__text`。 每个 Segment 以页单位对齐。Segment 有以下几种类型：

| 类型 | 说明 |
| - | - |
| __PAGEZERO | 空指针陷阱段，映射到虚拟内存空间的第一页，用于捕捉对 NULL 指针的引用 |
| __TEXT | 包含了执行代码以及其他只读数据。该段数据可以 VM_PROT_READ(读)、VM_PROT_EXECUTE(执行)，不能被修改 |
| __DATA | 程序数据，该段可写 VM_PROT_WRITE/READ/EXECUTE |
| __LINKEDIT | 链接器使用的符号以及其他表 |

Segment 数据结构定义如下：

```c
/*
 * The 64-bit segment load command indicates that a part of this file is to be
 * mapped into a 64-bit task's address space.  If the 64-bit segment has
 * sections then section_64 structures directly follow the 64-bit segment
 * command and their size is reflected in cmdsize.
 */
struct segment_command_64 { /* for 64-bit architectures */
	uint32_t	cmd;		/* LC_SEGMENT_64 */
	uint32_t	cmdsize;	/* includes sizeof section_64 structs */
	char		segname[16];	/* segment name */
	uint64_t	vmaddr;		/* memory address of this segment */
	uint64_t	vmsize;		/* memory size of this segment */
	uint64_t	fileoff;	/* file offset of this segment */
	uint64_t	filesize;	/* amount to map from the file */
	vm_prot_t	maxprot;	/* maximum VM protection */
	vm_prot_t	initprot;	/* initial VM protection */
	uint32_t	nsects;		/* number of sections in segment */
	uint32_t	flags;		/* flags */
};
```

字段说明：
- `cmd`：Load Command 类型
- `cmdsize`：Load Command 大小
- `segname`：名称
- `vmaddr`：段的虚拟内存地址
- `vmsize`：段的虚拟内存大小
- `fileoff`：段在文件中的偏移量
- `filesize`：段在文件中的大小
- `maxprot`：段最大操作权限
- `initprot`：段初始操作权限
- `nsects`：段中 section 数量
- `flags`：虚拟内存相关标志

Section 数据结构定义如下：

```c
struct section_64 { /* for 64-bit architectures */
	char		sectname[16];	/* name of this section */
	char		segname[16];	/* segment this section goes in */
	uint64_t	addr;		/* memory address of this section */
	uint64_t	size;		/* size in bytes of this section */
	uint32_t	offset;		/* file offset of this section */
	uint32_t	align;		/* section alignment (power of 2) */
	uint32_t	reloff;		/* file offset of relocation entries */
	uint32_t	nreloc;		/* number of relocation entries */
	uint32_t	flags;		/* flags (section type and attributes)*/
	uint32_t	reserved1;	/* reserved (for offset or index) */
	uint32_t	reserved2;	/* reserved (for count or sizeof) */
	uint32_t	reserved3;	/* reserved */
};
```

说明：
- `sectname`：名称，如 `__text`
- `segname`：所属 `segment`, 如 `__TEXT`
- `addr`：虚拟内存地址
- `size`：大小
- `offset`：文件偏移
- `align`：字节对齐信息，以2为底指数
- `reloff`：重定位入口文件偏移
- `nreloc`：重定位入口数量
- `flags`：类型与属性信息
- `reserved1`：保留字段，用于标识偏移或索引信息
- `reserved2`：保留字段，用于标志数量或大小
- `reserved3`：保留字段

##### 示例
- `__PAGEZERO` 段
![Segment-Pagezero]({{ site.baseurl }}/images/mach-object-file/segment-pagezero.png)
`__PAGEZERO` 是越界访问保护段，映射到虚拟内存地址0x0处，64位系统大小为4G（32位系统为4K）, 访问权限为 `VM_PROT_NONE`, 即不可读，不可写，不可执行，如果程序视图访问该段则会引起崩溃。由于它不需要真是的物理内存，所以 `filesize` 为0。

---
- `__TEXT` 段
![Segment-Text]({{ site.baseurl }}/images/mach-object-file/segment-text.png)
`__TEXT` 段紧随 `__PAGEZERO` 段，映射到虚拟地址空间0x100000000处，大小为4KB，权限为可读，可执行，不可写。示例中包含了5个 `section`：
- `__text`: 包含程序的机器码;
- `__stubs` 和 `__stub_helper`: 用来帮助 DYLD 绑定符号;
- `__cstring`: 记录了文件中的常量字符串信息(包含在双引号中)，我们可以依据此信息找到字符串的地址;
- `__unwind_info`: 用于确定异常发生时栈所对应的信息，包括栈指针、返回地址、寄存器信息等，它同样包含相应的处理函数来支持像 catch、final 等特性。

---
- `__DATA` 段
![Segment-Data]({{ site.baseurl }}/images/mach-object-file/segment-data.png)
`__DATA` 段紧随 `__TEXT` 段，存储程序内的数据，主要包含以下 section：
- `__nl_symbol_ptr`：包含的符号指针需要在加载时绑定；
- `__got`：用于存放non-lazy符号的最终地址值。
- `__la_symbol_ptr`：包含的符号指针则是在其第一次被程序使用时绑定。

---
- `__LINKEDIT` 段
![Segment-Linkedit]({{ site.baseurl }}/images/mach-object-file/segment-linkedit.png)
`__LINKEDIT` 将与动态链接相关的信息映射到虚拟地址空间，包括 rebase、bind、lazy bind 等信息。其中 `File Offset` 对应 `Dynamic Loader Info` 起时地址，如下图所示：
![Dynamic-Loader-Info]({{ site.baseurl }}/images/mach-object-file/dynamic-loader-info.png)

#### `LC_DYLD_INFO_ONLY`
`LC_DYLD_INFO_ONLY` 记录了有关链接的重要信息，数据结构定义：

```c
struct dyld_info_command {
    uint32_t   cmd;             /* LC_DYLD_INFO or LC_DYLD_INFO_ONLY */
    uint32_t   cmdsize;         /* sizeof(struct dyld_info_command) */
    uint32_t   rebase_off;      /* file offset to rebase info  */
    uint32_t   rebase_size;     /* size of rebase info   */
    uint32_t   bind_off;        /* file offset to binding info   */
    uint32_t   bind_size;       /* size of binding info  */
    uint32_t   weak_bind_off;   /* file offset to weak binding info   */
    uint32_t   weak_bind_size;  /* size of weak binding info  */
    uint32_t   lazy_bind_off;   /* file offset to lazy binding info */
    uint32_t   lazy_bind_size;  /* size of lazy binding infs */
    uint32_t   export_off;      /* file offset to lazy binding info */
    uint32_t   export_size;     /* size of lazy binding infs */
};
```

根据它所记录的偏移量，我们便可以找到在 Dynamic Loader Info 中的相关信息。它的 ONLY 后缀表明这是程序运行所必须的，如果链接器不支持，那么加载过程就会终止。

![Dyld-Info-Only]({{ site.baseurl }}/images/mach-object-file/dyld-info-only.png)
![Dynamic-Loader-Info]({{ site.baseurl }}/images/mach-object-file/dynamic-loader-info.png)

#### `LC_SYMTAB`
`LC_SYMTAB` 记录了程序的符号表以及字符串表的偏移量及大小，符号表中记录了程序用到的函数以及全局变量的信息，符号表条目的数据结构定义在 nlist.h 中。

```c
/*
 * The symtab_command contains the offsets and sizes of the link-edit 4.3BSD
 * "stab" style symbol table information as described in the header files
 * <nlist.h> and <stab.h>.
 */
struct symtab_command {
	uint32_t cmd;       /* LC_SYMTAB */
	uint32_t cmdsize;   /* sizeof(struct symtab_command) */
	uint32_t symoff;    /* symbol table offset */
	uint32_t nsyms;     /* number of symbol table entries */
	uint32_t stroff;    /* string table offset */
	uint32_t strsize;   /* string table size in bytes */
};

/*
 * This is the symbol table entry structure for 64-bit architectures.
 */
struct nlist_64 {
    union {
        uint32_t  n_strx; /* index into the string table */
    } n_un;
    uint8_t n_type;        /* type flag, see below */
    uint8_t n_sect;        /* section number or NO_SECT */
    uint16_t n_desc;       /* see <mach-o/stab.h> */
    uint64_t n_value;      /* value of this symbol (or stab offset) */
};
```

说明：
- `n_un`：符号名字，记录的是符号在 string table 中的起始偏移地址。

#### `LC_DYSYMTAB`
`LC_DYSYMTAB` 记录了动态加载时需要的符号信息，数据结构定义如下：

```c
struct dysymtab_command {
    uint32_t cmd;           /* LC_DYSYMTAB */
    uint32_t cmdsize;       /* sizeof(struct dysymtab_command) */
    uint32_t ilocalsym;     /* index to local symbols */
    uint32_t nlocalsym;     /* number of local symbols */
    uint32_t iextdefsym;    /* index to externally defined symbols */  
    int32_t nextdefsym;     /* number of externally defined symbols */
    uint32_t iundefsym;     /* index to undefined symbols */
    uint32_t nundefsym;     /* number of undefined symbols */
    uint32_t tocoff;        /* file offset to table of contents */
    uint32_t ntoc;          /* number of entries in table of contents */
    uint32_t modtaboff;     /* file offset to module table */
    uint32_t nmodtab;       /* number of module table entries */
    uint32_t extrefsymoff;  /* offset to referenced symbol table */
    uint32_t nextrefsyms;   /* number of referenced symbol table entries */

    uint32_t indirectsymoff; /* file offset to the indirect symbol table */
    uint32_t nindirectsyms;  /* number of indirect symbol table entries */
    uint32_t extreloff;     /* offset to external relocation entries */
    uint32_t nextrel;       /* number of external relocation entries */
    uint32_t locreloff;     /* offset to local relocation entries */
    uint32_t nlocrel;       /* number of local relocation entries */
};
```

说明：
- `ilocalsym`：本地符号仅用于调试
- `iextdefsym`：可执行文件定义的符号
- `iundefsym`：可执行文件里未定义符号
- `tocoff`: 目录偏移，该内容只有在动态分享库中存在。主要作用是把符号和定义它的模块对应起来。
- `modtaboff`：为了支持“模块”（整个对象文件）的动态绑定，符号表必须知道文件创建的模块。该内容只有在动态分享库中存在。
- `extrefsymoff`：为了支持动态绑定模块，每个模块都有一个引用符号表，符号表里存放着每个模块所引用的符号（定义的和没有定义的）。该表指针动态库中存在。
- `indirectsymoff`：如果 section 中有符号指针或者桩(stub)，section 中的 reserved1 存放该表的下标。间接符号表，只是存放一些32位小标，这些下标执行符号表。
- `extreloff`：每个模块都有一个重定义外部符号表。仅在动态库中存在
- `locreloff`：重定义本地符号表，由于只是在调试中用，所以不必更加模块分组。

#### `LC_LOAD_DYLIB`
`LC_LOAD_DYLIB` 指定了动态库加载信息，包括路径、创建时间、版本信息等，数据结构定义如下：

```c
struct dylib {
    union lc_str name;			/* library's path name */
    uint32_t timestamp;			/* library's build time stamp */
    uint32_t current_version;		/* library's current version number */
    uint32_t compatibility_version;	/* library's compatibility vers number*/
};

struct dylib_command {
    uint32_t cmd;               /* LC_ID_DYLIB, LC_LOAD_{,WEAK_}DYLIB, LC_REEXPORT_DYLIB */
    uint32_t cmdsize;           /* includes pathname string */
    struct dylib dylib;	        /* the library identification */
};
```

#### `LC_UUID`
`LC_UUID` 指定了文件唯一描述符，数据结构定义如下：

```c
struct uuid_command {
    uint32_t cmd;        /* LC_UUID */
    uint32_t cmdsize;    /* sizeof(struct uuid_command) */
    uint8_t uuid[16];    /* the 128-bit uuid */
};
```

#### `LC_VERSION_MIN_MACOSX`
`LC_VERSION_MIN_MACOSX` 指定了支持的最低系统版本信息，数据结构定义如下：

```c
struct version_min_command {
    uint32_t cmd;		/* LC_VERSION_MIN_MACOSX or
				   LC_VERSION_MIN_IPHONEOS or
				   LC_VERSION_MIN_WATCHOS or
				   LC_VERSION_MIN_TVOS */
    uint32_t cmdsize;	/* sizeof(struct min_version_command) */
    uint32_t version;	/* X.Y.Z is encoded in nibbles xxxx.yy.zz */
    uint32_t sdk;		/* X.Y.Z is encoded in nibbles xxxx.yy.zz */
};
```

#### `LC_MAIN`
`LC_MAIN` 指定了主函数地址以及栈信息，数据结构定义如下：

```c
struct entry_point_command {
    uint32_t cmd;       /* LC_MAIN only used in MH_EXECUTE filetypes */
    uint32_t cmdsize;   /* 24 */
    uint64_t entryoff;  /* file (__TEXT) offset of main() */
    uint64_t stacksize; /* if not zero, initial stack size */
};
```

### Lazy Binding 过程
Mach-O 文件文件中通过 dyld 加载的 Lazy Binding 表并没有在加载过程中直接确定地址列表，而是在第一次调用该函数的时候，通过 PLT(Procedure Linkage Table) 来进行一次 Lazy Binding。继续分析helloworld.c。

```c
#include <stdio.h>

int main(int argc, const char * argv[]) {
    // insert code here...
    printf("Hello, World!\n");
    return 0;
}
```

otool 查看汇编指令： `otool -v a.out -s __TEXT __text`

```armasm
; Section __text
0000000100000f50	pushq	%rbp
0000000100000f51	movq	%rsp, %rbp
0000000100000f54	subq	$0x20, %rsp
0000000100000f58	movl	$0x0, -0x4(%rbp)
0000000100000f5f	movl	%edi, -0x8(%rbp)
0000000100000f62	movq	%rsi, -0x10(%rbp)
0000000100000f66	leaq	0x35(%rip), %rdi
0000000100000f6d	movb	$0x0, %al
0000000100000f6f	callq	0x100000f82
0000000100000f74	xorl	%ecx, %ecx
0000000100000f76	movl	%eax, -0x14(%rbp)
0000000100000f79	movl	%ecx, %eax
0000000100000f7b	addq	$0x20, %rsp
0000000100000f7f	popq	%rbp
0000000100000f80	retq
```

MachO View 中对应 `__TEXT.__text` secton信息。
![Section-Text]({{ site.baseurl }}/images/mach-object-file/section-text.png)

红框中汇编指令对应上面的 `callq	0x100000f82`，地址 `0x100000f82` 就是 `__TEXT.__stub` 偏移地址，如下图所示：

![Section-stub]({{ site.baseurl }}/images/mach-object-file/section-stub.png)
通过 `otool -v a.out -s __TEXT __stubs` 解析成汇编指令如下：

```armasm
; Section __stubs
0000000100000f82	jmpq	*0x88(%rip)
```

相对地址寻址(RIP)方式计算地址方式为当前指令后一条指令的地址加上相对偏移地址。
上面 stub 对应的跳转指令最终跳转的地址为`[0x00000f82 + 6 + 0x88]`地址保存数据对应的地址。
其中 `0x100000f82` 为当前指令地址，`6` 为当前指令 `FF2588000000` 长度为，两者相加即为下一条指令地址，`0x88` 为偏移地址，三者相加结果为 `0x00001010`，该地址就是 `__DATA.__la_symbol_ptr` 符号表中对应 `__printf` 符号的地址，该地址存储的数据 `0x100000F98` 就是最终跳转的地址。

通过 `otool -v a.out -s __TEXT __stub_helper` 查看汇编代码：

```armasm
; Section __stub_helper
0000000100000f88	leaq	0x71(%rip), %r11
0000000100000f8f	pushq	%r11
0000000100000f91	jmpq	*0x71(%rip)
0000000100000f97	nop
0000000100000f98	pushq	$0x0
0000000100000f9d	jmp	0x100000f88
```

![Section-stub-helper]({{ site.baseurl }}/images/mach-object-file/section-stub-helper.png)

通过 `_stub_helper` 来调用 `dyld_stu_binder` 方法计算 `printf` 函数的真实地址。其具体信息可以看出，`jmpq 0x100000f88` 就是在 `pushq` 参数 `$0x0` (link 过程中的标记值)后跳转到这个 section 的头部，并调用 binder 方法计算 `printf` 函数地址并进行绑定。

## 参考资料
- [Mach-O](https://zh.wikipedia.org/wiki/Mach-O)
- [Mach-O Runtime Architecture](http://math-atlas.sourceforge.net/devel/assembly/MachORuntime.pdf)
- [OS X ABI Mach-O File Format Reference](http://nicolascormier.com/documentation/macosx-programming/MachORuntime.pdf)
- [Overview of the Mach-O Executable Format](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/CodeFootprint/Articles/MachOOverview.html#//apple_ref/doc/uid/20001860-BAJGJEJC)
- [Mach-O Programming Topics](https://developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/MachOTopics/0-Introduction/introduction.html#//apple_ref/doc/uid/TP40001827-SW1)
- [解读 Mach-O 文件格式](https://amywushu.github.io/2017/02/21/基础知识-解读-Mach-O-文件格式.html)
- [Mach-O文件](https://www.jianshu.com/p/d8c6458270db)
- [Mach-O 文件探索](https://www.jianshu.com/p/ca01f3d622c7)
- [Mach-O 与动态链接](https://zhangbuhuai.com/post/macho-dynamic-link.html#section-data-got)
- [Mach-O 文件格式探索](https://www.desgard.com/iosre-1/)