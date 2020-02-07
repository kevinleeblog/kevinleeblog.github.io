---
layout: post
title: "使用Autotool產生Makefile"
auther: Kevin Lee
category: project1
tags: [OpenBMC]
subtitle:
visualworkflow: true
---

### 為何做？

這幾天開始專研OpenBMC時，常看到一個範例是使用./configure產生makefile，在利用產生出來的makefilef去make compile，這種生成makefile的過程就是使用Autoconf和Automake來完成的。

OpenBMC內所有的軟體Packages幾乎都是先使用這套工具產生makefile，然後在為它寫Yocto Recipes，既然要專研OpenBMC，也需要去熟悉如何編寫或修改。

#### 安裝

```
$ sudo apt install autoconf
```



自行製作demo範例，目錄結構如下

```
$ tree demo_code/
demo_code/
└── demo.c

0 directories, 1 file
```

*demo.c*

```c
#include <stdio.h>

int main()
{
	printf("Hello!OpenBMC\n");
	return 0;
}
```



首先是以後想要使用./configure產生Makefile
先使用autoscan掃描專案目錄內所有程式檔產生configure.scan

```
$ cd demo_code
$ autoscan 
Unescaped left brace in regex is deprecated, passed through in regex; marked by <-- HERE in m/\${ <-- HERE [^\}]*}/ at /usr/bin/autoscan line 361.
$ ls
autoscan.log  configure.scan  demo.c
```

*configure.scan*

```
$ cat configure.scan 
#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_PREREQ([2.69])
AC_INIT([FULL-PACKAGE-NAME], [VERSION], [BUG-REPORT-ADDRESS])
AC_CONFIG_SRCDIR([demo.c])
AC_CONFIG_HEADERS([config.h])

# Checks for programs.
AC_PROG_CC

# Checks for libraries.

# Checks for header files.

# Checks for typedefs, structures, and compiler characteristics.

# Checks for library functions.

AC_OUTPUT
```

把configure.scan替換成configure.ac並修改部分內容

```
$ mv configure.scan configure.in
$ vim configure.ac
#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_PREREQ([2.69])
#AC_INIT([FULL-PACKAGE-NAME], [VERSION], [BUG-REPORT-ADDRESS])
AC_INIT([my_demo], [1.0], [https://kevinleeblog.github.io/])
AC_CONFIG_HEADERS([config.h])
AM_INIT_AUTOMAKE([subdir-objects -Wall -Wno-portability -Werror foreign])

# Checks for programs.
AC_PROG_CC

# Checks for libraries.

# Checks for header files.

# Checks for typedefs, structures, and compiler characteristics.

# Checks for library functions.
AC_CONFIG_FILES([Makefile])
AC_OUTPUT
```

`AC_PROG_CC` :checks for a C compiler.

`AC_CONFIG_FILES([Makefile src/Makefile])`:Declare `Makefile` and `src/Makefile` as output files

產生Makefile.am並編寫內容

*Makefile.am*

```
AUTOMAKE_OPTIONS=foreign
bin_PROGRAMS=demo
demo_SOURCES=demo.c
```

使用`autoreconf -fmi` 自動產生所需要的檔案

```
$ autoreconf -fmi
configure.ac:11: installing './compile'
configure.ac:8: installing './install-sh'
configure.ac:8: installing './missing'
Makefile.am: installing './depcomp'
$ automake
```

```
$  man autoreconf
...

-f, --force
              consider all files obsolete

       -i, --install
              copy missing auxiliary files

       --no-recursive
              don’t rebuild sub-packages

       -s, --symlink
              with -i, install symbolic links instead of copies

       -m, --make
              when applicable, re-run ./configure && make

...
```

到此有了configure後，就可以產生Makefile

```
$ ./configure 
checking for a BSD-compatible install... /usr/bin/install -c
checking whether build environment is sane... yes
checking for a thread-safe mkdir -p... /bin/mkdir -p
checking for gawk... gawk
checking whether make sets $(MAKE)... yes
checking whether make supports nested variables... yes
checking for gcc... gcc
checking whether the C compiler works... yes
checking for C compiler default output file name... a.out
checking for suffix of executables... 
checking whether we are cross compiling... no
checking for suffix of object files... o
checking whether we are using the GNU C compiler... yes
checking whether gcc accepts -g... yes
checking for gcc option to accept ISO C89... none needed
checking whether gcc understands -c and -o together... yes
checking for style of include used by make... GNU
checking dependency style of gcc... gcc3
checking that generated files are newer than configure... done
configure: creating ./config.status
config.status: creating Makefile
config.status: creating config.h
config.status: executing depfiles commands
```

開始編譯並執行

```
$ make
make  all-am
make[1]: Entering directory '/home/kevinlee/demo_code'
depbase=`echo demo.o | sed 's|[^/]*$|.deps/&|;s|\.o$||'`;\
gcc -DHAVE_CONFIG_H -I.     -g -O2 -MT demo.o -MD -MP -MF $depbase.Tpo -c -o demo.o demo.c &&\
mv -f $depbase.Tpo $depbase.Po
gcc  -g -O2   -o demo demo.o  
make[1]: Leaving directory '/home/kevinlee/demo_code'

$ ./demo
Hello!OpenBMC
```

如果是要使用Cross-Compiler

```
$ source /usr/local/oecore-x86_64/environment-setup-arm1176jzs-openbmc-linux-gnueabi 

$ ./configure --host=x86_64
configure: loading site script /usr/local/oecore-x86_64/site-config-arm1176jzs-openbmc-linux-gnueabi
checking for a BSD-compatible install... /usr/bin/install -c
checking whether build environment is sane... yes
checking for x86_64-strip... arm-openbmc-linux-gnueabi-strip
checking for a thread-safe mkdir -p... /bin/mkdir -p
checking for gawk... gawk
checking whether make sets $(MAKE)... yes
checking whether make supports nested variables... yes
checking for x86_64-gcc... arm-openbmc-linux-gnueabi-gcc  -marm -mcpu=arm1176jz-s --sysroot=/usr/local/oecore-x86_64/sysroots/arm1176jzs-openbmc-linux-gnueabi
checking whether the C compiler works... yes
checking for C compiler default output file name... a.out
checking for suffix of executables... 
checking whether we are cross compiling... yes
checking for suffix of object files... o
checking whether we are using the GNU C compiler... yes
checking whether arm-openbmc-linux-gnueabi-gcc  -marm -mcpu=arm1176jz-s --sysroot=/usr/local/oecore-x86_64/sysroots/arm1176jzs-openbmc-linux-gnueabi accepts -g... yes
checking for arm-openbmc-linux-gnueabi-gcc  -marm -mcpu=arm1176jz-s --sysroot=/usr/local/oecore-x86_64/sysroots/arm1176jzs-openbmc-linux-gnueabi option to accept ISO C89... none needed
checking whether arm-openbmc-linux-gnueabi-gcc  -marm -mcpu=arm1176jz-s --sysroot=/usr/local/oecore-x86_64/sysroots/arm1176jzs-openbmc-linux-gnueabi understands -c and -o together... yes
checking whether make supports the include directive... yes (GNU style)
checking dependency style of arm-openbmc-linux-gnueabi-gcc  -marm -mcpu=arm1176jz-s --sysroot=/usr/local/oecore-x86_64/sysroots/arm1176jzs-openbmc-linux-gnueabi... gcc3
configure: creating ./config.status
config.status: creating Makefile
config.status: creating config.h
config.status: config.h is unchanged
config.status: executing depfiles commands

$ make
make  all-am
make[1]: Entering directory '/tmp/tt'
depbase=`echo demo.o | sed 's|[^/]*$|.deps/&|;s|\.o$||'`;\
arm-openbmc-linux-gnueabi-gcc  -marm -mcpu=arm1176jz-s --sysroot=/usr/local/oecore-x86_64/sysroots/arm1176jzs-openbmc-linux-gnueabi -DHAVE_CONFIG_H -I.     -O2 -pipe -g -feliminate-unused-debug-types  -MT demo.o -MD -MP -MF $depbase.Tpo -c -o demo.o demo.c &&\
mv -f $depbase.Tpo $depbase.Po
arm-openbmc-linux-gnueabi-gcc  -marm -mcpu=arm1176jz-s --sysroot=/usr/local/oecore-x86_64/sysroots/arm1176jzs-openbmc-linux-gnueabi  -O2 -pipe -g -feliminate-unused-debug-types   -Wl,-O1 -Wl,--hash-style=gnu -Wl,--as-needed -o demo demo.o  
make[1]: Leaving directory '/tmp/tt'
```

