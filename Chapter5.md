# 教程辅导 #

如果您对教程中的任何地方存在任何疑惑、或者问题，请加入我们的QQ群寻求帮助：804824578

# 1、介绍 #

动态链接库，是一个很古老，但又很常用的技术。在我们平时的代码编译中，通常是将静态库链接到程序中，最终静态库成为程序二进制的一部分而融合成一体，但是，这样的静态链接面临的一个问题就是，如果一个软件的各个部分是通过协商好接口，由不同的小组所编写的，那么如何能够让每一个小组独立的维护自己所属的部分，而不用重新编译其他人所开发的代码，动态链接库由此而生。

动态链接库，可以让每一个人维护的部分以独立的库的形式存在而不用与应用程序融为一体，也就是dll文件，应用程序调用其中的函数时，需要动态的加载和链接其中的函数，然后可以正常的与静态链接一样的进行操作。

实际上在后来的动态链接应用技术的发展之中，许多新的应用被人们挖掘出来，由于多个应用程序可以同时加载同一个动态链接库，就为基于动态链接库的多个程序通过共享内存进行进程间通讯（IPC）提供了可能。

在下面的文章中，我们只从初步角度介绍动态链接库以及在MSYS环境下，如何取代VC同样实现动态链接库的开发和使用。

# 2、启动时加载的动态链接库 #

这种类型的动态链接库，实际上是动态链接库的最初应用形式，其除了可以通过简单的替换dll文件以对库进行更新而不用重新编译主程序之外，没有太大的区别，下面是一个代码例子：

我们创建一个目录hellodll1，以包含我们的项目文件。下面我们首先编写动态链接库自身，起名为lib.c，写入以下普通静态库类似的内容：
```
#include <stdio.h>
#include "lib.h"

int foo(char *str)
{
        printf("%s\n", str);
        return 0;
}
```

下面是这个库所对应的介面头文件lib.h：
```
#ifndef __LIB_H__
#define __LIB_H__

extern int foo(char *str);

#endif
```

然后我们对这个动态库进行编译，使用如下的命令（对于Makefile的编译方式，推荐读者自己通过修改以前的Makefile内容得到答案，这里我就不明确阐述了）：
```
gcc --share lib.c -o lib.dll
```

下面我们编写调用这个动态链接库的程序main.c：
```
#include "lib.h"
int main(void)
{
        foo("hello world, dll from MSYS");
        return 0;
}
```

然后我们编译出调用lib.dll的主程序：
```
gcc main.c lib.dll -o hellodll1.exe
```

运行hellodll1.exe之后，我们就能看到我们所期待的结果，如何证明我们的hellodll.exe是运行的dll.exe中的内容呢？简单的将lib.dll删除，再次运行会报告找不到lib.dll的错误，或者我们通过修改lib.dll的代码，让他产生不同的输出以验证：
```
#include <stdio.h>
#include "lib.h"

int foo(char *str)
{
        printf("new dll: %s\n", str);
        return 0;
}
```

再次编译出lib.dll：
```
gcc --share lib.c -o lib.dll
```

然后不编译main.c而直接运行hellodll1.dll，我们发现，输出的结果已经改变，说明hellodll1.exe的确是动态链接的lib.dll文件，并执行的里面的foo函数，而不是静态链接。

在下一节我们将介绍另外一种按需加载的动态链接库类型。

# 3、按需加载的动态链接库 #

人们在对dll技术发展的过程中发现既然dll文件可以被动态的加入到程序执行的内存中并实现链接成为程序的一部分，我们是否扩展该应用实现程序的动态扩展插件呢？于是人们基于操作系统内核技术发明了按需加载的动态链接库技术，现在多用于各种插件应用。

下面我们创建hellodll2项目，开始新的旅程。我们注意到在hellodll1中，一个动态链接库似乎与一个静态链接库无论是从代码上还是效果上都没有什么差别，无非只是二进制的隔离和独立，但是按需加载的动态链接库就开始有了区别。

动态链接库逐渐发展成了“程序的扩展模块”，因此一个模块需要有启动时的初始化和退出时的反初始化以及数据保存过程，于是人们在这种动态链接库中添加了一个专有函数来解决这个问题，也就是类似程序中main函数的DllMain函数，下面是我们新动态链接库lib.c的代码：
```
#include <windows.h>
#include <stdio.h>

// 这就是按需加载的dll的主函数，dll被加载、卸载时，系统都回调用这个函数，通过dwReason判断
BOOL WINAPI DllMain(HINSTANCE hModule, DWORD dwReason, LPVOID lpvReserved)
{
        switch(dwReason) {
        // 如果是进程加载本dll
        case DLL_PROCESS_ATTACH:
                printf("process attached\n");
                break;
        // 如果是进程卸载本dll
        case DLL_PROCESS_DETACH:
                printf("process detached\n");
                break;
        // 如果是线程加载本dll
        case DLL_THREAD_ATTACH:
                printf("thread attached\n");
                break;
        // 如果是线程卸载本dll
        case DLL_THREAD_DETACH:
                printf("thread attached\n");
                break;
        }
        // 如果返回FALSE，则说明加载失败，不会继续被加载，也不可使用
        return TRUE;
}

int foo(char *str)
{
        printf("%s\n", str);
        return 0;
}
```

用下面的命令进行编译：
```
gcc --share lib.c -o lib.dll
```

下面是我们用于加载这个dll的程序的main.c的内容：
```
#include <windows.h>
#include <stdio.h>

typedef int (*FOO)(char *str);

int main()
{
        HMODULE mod;
        FOO     foo;

        mod = LoadLibrary("lib.dll");

        printf("from main\n");

        if((foo = (FOO)GetProcAddress(mod, "foo")) != NULL) {
                foo("hello world");
        }

        FreeLibrary(mod);

        return 0;
}
```

用下面的命令进行编译：
```
gcc main.c -o hellodll2.exe
```

运行hellodll2.exe的结果如下：
```
$ ./hellodll2.exe 
process attached
from main
hello world
process detached
```

相信从输出的文字的顺序，大家可以看到DllMain中的dwReason是按照什么顺序发送的。

笔者在这里比较浅显的介绍了MSYS中dll的开发和使用方法，除了具体的命令调用方法以外，与VC上开发dll没有区别，因此如果读者希望更加深入的研究Windows上的DLL机制的话，请参考MSDN文献，其中的例子都可以拿到MSYS上来进行尝试而不用怀疑MSYS是否能行。