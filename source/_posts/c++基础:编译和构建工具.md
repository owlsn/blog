title: c++基础：编译和构建工具
date: 2019-11-14 12:00:53
tags: [cpp]
categories: 学习笔记

---
### 背景  
最近又开始学c++了，因为项目中有用到，要开发项目，首先要搭环境，要先熟悉一下编译过程和构建工具

### 编译工具gcc和g++
#### 代码编译过程   
![](https://raw.githubusercontent.com/owlsn/blog/master/source/source/GCC_CompilationProcess.png)
1.预处理,引入#include头文件以及宏定义展开，得到文本文件   
cpp hello.c > hello.i   
2.编译，将预处理后的文件转换成汇编代码,得到程序的汇编代码     
gcc -S hello.i   
3.汇编,将上一步汇编语言代码转换成机器码,得到二进制目标文件      
as -o hello.o hello.s   
4.链接,将多个目标文件以及所需的库文件链接成最终的可执行文件    
ld -o hello.exe hello.0 ---libraries...   
<!-- more -->
#### 静态和动态库文件 
1.静态库文件在unix系统以.a(archive file)结尾，在windows系统以.lib(library)结尾,静态库文件中的代码会被拷贝进自己的程序中，通过gcc的ar命令可以生成静态库文件   
2.动态库文件在unix系统以.so(shared objects)结尾，在windows系统以.dll(dynamic link library)结尾,只有一部分映射关系会被记录在可执行文件中，当最后的可执行文件开始运行的时候，操作系统才会根据映射关系去加载动态库文件，这个操作就叫做动态链接，最后生成的可执行文件体积就会小很多，而且动态库文件可以被很多程序共享同一份拷贝，所以可以节约空间，另外动态库文件代码更新后也不会影响其他的调用程序   

##### include头文件和加载库文件
1.-Idir或者CPATH环境变量可以指定编译时寻找include头文件的路径   
2.-Ldir或者LIBRARY_PATH环境变量可以指定寻找库文件的路径   
3.unix系统可以通过-lxxx指定libxxx.a库文件的名称，windows系统可以通过-lxxx.lib指定库文件的名称
4.没有指定时的默认include-paths和library-paths以及libraries，通过cpp -v 命令可以查看，gcc编译时-v 参数可以查看library-paths(-L)和libraries(-l) 
  
		> cpp -v
		......
		#include "..." search starts here:
		#include <...> search starts here:
		 /usr/lib/gcc/x86_64-linux-gnu/5/include
		 /usr/local/include
		 /usr/lib/gcc/x86_64-linux-gnu/5/include-fixed
		 /usr/include/x86_64-linux-gnu
		 /usr/include
		End of search list.
		
		
		> g++ -v -o hello.exe helloworld.cc
		......
		#include "..." search starts here:
		#include <...> search starts here:
		 /usr/include/c++/5
		 /usr/include/x86_64-linux-gnu/c++/5
		 /usr/include/c++/5/backward
		 /usr/lib/gcc/x86_64-linux-gnu/5/include
		 /usr/local/include
		 /usr/lib/gcc/x86_64-linux-gnu/5/include-fixed
		 /usr/include/x86_64-linux-gnu
		 /usr/include
		End of search list.
		......
		COMPILER_PATH=/usr/lib/gcc/x86_64-linux-gnu/5/:/usr/lib/gcc/x86_64-linux-gnu/5/:/usr/lib/gcc/x86_64-linux-gnu/:/usr/lib/gcc/x86_64-linux-gnu/5/:/usr/lib/gcc/x86_64-linux-gnu/
		LIBRARY_PATH=/usr/lib/gcc/x86_64-linux-gnu/5/:/usr/lib/gcc/x86_64-linux-gnu/5/../../../x86_64-linux-gnu/:/usr/lib/gcc/x86_64-linux-gnu/5/../../../../lib/:/lib/x86_64-linux-gnu/:/lib/../lib/:/usr/lib/x86_64-linux-gnu/:/usr/lib/../lib/:/usr/lib/gcc/x86_64-linux-gnu/5/../../../:/lib/:/usr/lib/
		COLLECT_GCC_OPTIONS='-v' '-o' 'hello.ext' '-shared-libgcc' '-mtune=generic' '-march=x86-64' /usr/lib/gcc/x86_64-linux-gnu/5/collect2 -plugin /usr/lib/gcc/x86_64-linux-gnu/5/liblto_plugin.so -plugin-opt=/usr/lib/gcc/x86_64-linux-gnu/5/lto-wrapper -plugin-opt=-fresolution=/tmp/cckcI1Gr.res -plugin-opt=-pass-through=-lgcc_s -plugin-opt=-pass-through=-lgcc -plugin-opt=-pass-through=-lc -plugin-opt=-pass-through=-lgcc_s -plugin-opt=-pass-through=-lgcc --sysroot=/ --build-id --eh-frame-hdr -m elf_x86_64 --hash-style=gnu --as-needed -dynamic-linker /lib64/ld-linux-x86-64.so.2 -z relro -o hello.ext /usr/lib/gcc/x86_64-linux-gnu/5/../../../x86_64-linux-gnu/crt1.o /usr/lib/gcc/x86_64-linux-gnu/5/../../../x86_64-linux-gnu/crti.o /usr/lib/gcc/x86_64-linux-gnu/5/crtbegin.o -L/usr/lib/gcc/x86_64-linux-gnu/5 -L/usr/lib/gcc/x86_64-linux-gnu/5/../../../x86_64-linux-gnu -L/usr/lib/gcc/x86_64-linux-gnu/5/../../../../lib -L/lib/x86_64-linux-gnu -L/lib/../lib -L/usr/lib/x86_64-linux-gnu -L/usr/lib/../lib -L/usr/lib/gcc/x86_64-linux-gnu/5/../../.. /tmp/ccNcB9ML.o -lstdc++ -lm -lgcc_s -lgcc -lc -lgcc_s -lgcc /usr/lib/gcc/x86_64-linux-gnu/5/crtend.o /usr/lib/gcc/x86_64-linux-gnu/5/../../../x86_64-linux-gnu/crtn.o

#### gcc环境变量
1.PATH,寻找动态库文件路径   
2.CPATH,寻找include头文件路径   
3.LIBRARY_PATH,寻找库文件路径  

### make
gcc和g++命令编译几个文件还可以应付，但是大型项目成千上万个文件就无法处理了，make就像是执行gcc/g++命令的脚本工具，make应该是应用最广泛的c/c++编译工具了   
使用make要编写makefile，要先学习makefile的语法   

### bazel
make的功能应该是很强大了，基本上对于大型项目来说已经够用了，但是make也有一些不足，当项目逐渐变大的时候，makefile也会变得越来越臃肿了，bazel是谷歌开源的一个构建工具，用起来比make更加方便，而且编译时可以只编译更新的部分。

### 参考文献
[1].[Compiling, Linking and Building
C/C++ Applications](https://www3.ntu.edu.sg/home/ehchua/programming/cpp/gcc_make.html)