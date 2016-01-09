# 教程辅导 #

如果您对教程中的任何地方存在任何疑惑、或者问题，请加入我们的QQ群寻求帮助：804824578

# 1、开始前的准备 #

MSYS开发环境可以实现完整的Windows应用程序的开发，基于其内部的Windows API函数，我们可以实现任何我们想要实现的函数而不需要安装庞大的VC、.net开发环境。至于MS为什么要推出.net开发环境呢？凭我的个人理解，主要的原因是：
  1. 由于GNU推动的自由软件的发展，创造出了多门语言（包括TclTk、Python等）能够实现多种语言的专业人才能够同时开发同一个项目，实现多语言联合；
  1. 由于SUN公司的Java虚拟机得到了广泛的应用；
  1. 由于MFC传统对Windows API的封装效率过低，而且类的结构不符合人性化设计；
当然，这些也许只是一些片面的原因，但是最终微软设计出了.net代码开发框架，实现了以下的功能：
  1. 基于.net的应用程序，实际上都是虚拟机应用程序，将代码以脚本形式封装在程序内解析执行；
  1. 多种语言（VB、VC、Java、C#）的联合开发同一个项目；
  1. 极大的增加了代码的体积、开发工具的尺寸和光盘的数量，感觉起来多了许多新东西；
总之，我本人对于.net是没有任何好感的，基于我们的MSYS环境同样可以实现Fortran、C、C++、Java、ADA、Objective C、Objective C++、Clisp等等语言的联合开发，同时可以更好地兼容Python、Tcltk、Ruby、Lua等脚本语言的嵌入，因为MSYS环境使用的正式开发这些语言用的gcc编译器。

MSYS的gcc中包含了完整的Windows SDK、DDK、DirectX SDK等开发包，因此我们不用像VC那样安装各种附加的SDK、DDK，同时我们可以使用微软官方的MSDN作为这些库的函数的手册，MSYS也不需要再额外的提供技术手册或者info文件。首先我们需要下载一份MSDN，如果您英语比较好或者适应了英文版的MSDN，请到这里下载徐艺波所精简的MSDN版本：

> http://www.xuyibo.org/download/14.htm

您也可以上微软官方的 VC Express 版本主页下载中文版的 MSDN，不过下载可能要花费一定的时间。
装好MSYS后，请确保您已经安装了resedit附加包：
```
mpkg --install resedit
```

安装好以后需要进行一次重启，然后下面我们就开始Windows应用程序开发的旅程。

# 2、第一个Win32版本Hello World #

在我们通常学习一样彻底不了解的东西的时候，通常有两种途径，一个是将该技术所涉及的所有官方文档、手册、定义都弄到手，然后专心的去研究这些手册中的东西，还有一种就是读一本别人编写的教程跟着一步一步的做。按照我以往的经验来看，这二者各有优点和缺点：

  * 采用标准或文档进行学习，缺乏有效的实践方式，不能从实践上去操作和解决问题，但是优点是能够全面的掌握知识体系，并且能够看清各个部分之间的关系。这种学习方式就好比正统的教育模式，将所有需要了解的东西一下全部灌输给学生，宁可死记硬背也要把内容都记下来，但是实际动手的时候就缺乏动力和想象力，没有激情。

  * 采用跟着某教程走的方式进行学习，优点就是动手能力强，并且在做的过程中能够提出更好的想法来解决同样的问题，利于产生创造的激情和锻炼实践能力，但是缺点也同样存在，那就是教程成了做事情必备的参考文档，一旦脱离教程就无法进行操作，同时不能逾越教程中作者个人所关注的一些领域，不利于对新领域的开拓，同时不能完整、系统的掌握知识。

当然，本系列文章当属教程类型，期待通过各种实际动手的例子让读者能够了解使用MSYS进行Windows操作系统开发的方法，同时明白控制台开发方式的优点并善于利用IDE所不拥有的优势来开发新型应用程序。下面我们本着动手为主，辅助提供各种参考资料的使用方法，让读者能够在进行教程式学习的同时能够弥补这种方式的不足。

首先我们打开MSYS，进入控制台界面，基于在OS开发系列文章第二篇（如果没有阅读过的读者请在我的博客中阅读）中的基础操作技术，我们开始Windows上最简单版本Hello World的开发方法。我们创建一个目录winhello包含我们的代码，并创建名为main.c的文件，我们开始编写代码：

首先我们需要知道，在Windows开发中我们多数时候用到的是Windows API，而这些接口的定义都可以通过简单的包含Windows.h来实现，当然，如果读者需要使用到printf函数进行辅助调试的话，同样可以包含控制台程序通常用的stdio.h、stdlib.h等头文件。
```
#include <windows.h>
```

下面我们需要开始我们的Windows版本main函数，在Windows上主函数叫做WinMain，它的定义比较复杂，我们很难准确记住其参数，因此从现在开始，只要您开发Windows，无论是在VC中还是MSYS、MingW中，都会发现MSDN是一个很好的帮手。

我们打开MSDN，在左边的选项卡中选择索引（Index），然后在其中输入搜索‘winmain’（大小写无所谓），然后我们能打开对WinMain函数定义进行说明的页面，我们照抄其中的参数定义方式如下：
```
int WINAPI WinMain(HINSTANCE hInst, HINSTANCE hPrev, LPSTR lpCmd, int nCmd)
{
    return 0;
}
```

下面我们通过一个Windows中常见的消息框来显示一个Hello World消息，消息框产生的函数是MessageBox，在MSDN索引中搜索到后，在WinMain中输入如下的代码：
```
MessageBox(0, "Hello World, MSYS !", "caption", 0);
```

现在复制我们前面用到的Makefile，或者凭您对Makefile的理解手工编写，得到如下内容的Makefile：
```
# 配置工具链
AS=as     # 汇编器
CC=gcc    # C编译器
LD=gcc    # 链接器

# 配置工具链的调用参数
ASFLAGS=-g
CCFLAGS=-g
LDFLAGS=

# 我们目标，可用'\'换行
OBJECT=main.o
TARGET=winhello.exe

# 如何编译各种目标的过程

# 默认目标（make 不带目标对象，或者make all）
all: $(TARGET)

$(TARGET) : $(OBJECT)
    $(LD) $(OBJECT) -o $(TARGET) $(LDFLAGS)

# 指定从.c文件生成.o文件的方法，$@代表输出文件，$<代表输入文件
%.o : %.c
    $(CC) $(CCFLAGS) -c -o $@ $<

%.o : %.s
    $(AS) $(ASFLAGS) -o $@ $<
```

然后我们执行make命令，就得到了最简单形式的Windows版本Hello World。

# 3、第二个Hello World #

下面我们开始实现一个基于Windows窗体的Hello World，首先我们需要创建一个新的目录来承载我们的新项目，起名为winhello2：
```
mkdir winhello2
cd winhello2
```

然后我们首先用resedit工具来绘制一个我们要创建的Windows窗体：
```
resedit resource.rc &  # 注意：&符号用于让控制台可以继续做其他事情
```

现在弹出了一个类似VC的窗体，正在编辑我们新创建的resource.rc资源文件，之所以我们要起名为resource.rc是因为resedit程序默认保存针对resource.rc的介面定义文件为resource.h，而且一个windows程序只允许拥有一个资源文件，所以我们一般创建Windows应用程序只需要命名为resource.rc即可，不需要太多的考虑。

在窗体编辑器中，我们在左上角的工具框中点击右键，弹出一个菜单，我们选择 Add Resource... -> Dialog，然后一个崭新的默认对话框界面在右面展示在了我们眼前。上面默认包含了OK和CANCEL两个对话框按钮，我们先不去管它，现在我们留意左上角工具框中对话框资源的名称IDD\_DIALOG1，这个字符串将成为我们创建这个对话框时所使用的标识符，用左上角的工具栏中的RC保存，我们就得到了resource.rc和resource.h文件，用以下的方式察看，切换到MSYS界面，输入：
```
$ ls
resource.h  resource.rc
```

下面我们开始编写main.c文件包含我们的主程序（其中的各个Windows函数请参考MSDN）：
```
#include <windows.h>
#include "resource.h"

// Windows消息机制（参见下面的讲述）的精髓--消息回叫函数。
INT_PTR CALLBACK DlgProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
        // 对消息类别进行判断
        switch(uMsg) {
        // 如果是窗口初始化消息
        case WM_INITDIALOG:
                // 这是我们上一个版本中的Hello World
                MessageBox(hwnd, "Hello World!", "From MSYS", 0);
                break;

        // 如果窗口被关闭
        case WM_CLOSE:
                // 要求系统关闭这个程序
                PostQuitMessage(0);
                break;
        }

        // 默认返回0
        return 0;
}

// Windows系统的主函数
int WINAPI WinMain(HINSTANCE hInst, HINSTANCE hPrev, LPSTR lpCmd, int nCmd)
{
        // 用这个系统函数创建我们刚刚绘制好的窗体IDD_DIALOG1，设置回叫函数为DlgProc
        DialogBox(hInst, MAKEINTRESOURCE(IDD_DIALOG1), 0, DlgProc);

        return 0;
}
```

在这个例子中，我们首次接触到了Windows的消息机制。类似Linux上的信号机制，消息机制是Windows系统采用的内核与应用程序通讯机制。由于Windows的图形系统是实现在内核层的，为了突破应用程序和内核层之间的保护，让二者可以互相通讯，Windows上设计了异步的消息的方式实现这种通讯。

当内核向应用程序发送一个消息时，消息首先被缓存在了应用程序的消息池中，然后程序自身用消息泵的方式将消息一个一个取出（同时实现过滤），然后呼叫消息回叫函数来执行各个消息相应的动作，比如按钮被按下，或者我们这里的对话框初始化，显示hello world消息框。

为了编译这个程序，我们可以采用如下的手工编译模式，也可以采用下面的Makefile自动编译：
```
gcc -c main.c                             # 编译main.c程序
windres resource.rc resource.o            # 编译资源文件
gcc main.o resource.o -o winhello2.exe    # 进行链接
./winhello2.exe                           # 执行
```
```
# 配置工具链
AS=as      # 汇编器
CC=gcc     # C编译器
RC=windres # 资源编译器
LD=gcc     # 链接器

# 配置工具链的调用参数
ASFLAGS=-g
CCFLAGS=-g
RCFLAGS=
LDFLAGS=

# 我们目标，可用'\'换行
OBJECT=main.o \
       resource.o
TARGET=winhello2.exe

# 如何编译各种目标的过程

# 默认目标（make 不带目标对象，或者make all）
all: $(TARGET)

$(TARGET) : $(OBJECT)
    $(LD) $(OBJECT) -o $(TARGET) $(LDFLAGS)

# 指定从.c文件生成.o文件的方法，$@代表输出文件，$<代表输入文件
%.o : %.c
    $(CC) $(CCFLAGS) -c -o $@ $<

%.o : %.s
    $(AS) $(ASFLAGS) -o $@ $<

%.o : %.rc
    $(RC) $(RCFLAGS) $< $@
```

这是Windows上最简单的程序，我们直接绘制窗口、编写一个回叫函数、调用DialogBox函数，就可以实现窗体的创建，不用我们自己去关心消息泵的实现和消息的分配。下面我们对这个例子进行一点复杂化，让我们更加明确Windows中各种窗体控件的应用。

在刚才打开的resedit窗口中，我们继续编辑，如果刚才不小心关闭了，我们可以执行同样的命令继续打开编辑。现在我们从右边的工具栏中选择"Edit Control"，小心的绘制在我们窗体的任意位置，然后再选择Static Text控件，绘制在其中（这些新绘制的控件也有标识符，在左下角属性选择框中的其中Misc组中的ID属性我们可以看到和更改，我们将Static Text的ID改为IDC\_DISPLAY）。下面我们来编写自己的程序，当按下OK按钮后，读出这个Edit Control中用户输入的内容，然后通过printf打印其中的内容，并将其中的内容写入Static Text之中。

编辑并修改main.c为如下：
```
#include <windows.h>
#include <stdio.h>      // 注意这里添加了控制台的头文件
#include "resource.h"

INT_PTR CALLBACK DlgProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
        switch(uMsg) {
        case WM_INITDIALOG:
                MessageBox(hwnd, "Hello World!", "From MSYS", 0);
                break;
        // 如果是命令事件（按钮等）
        case WM_COMMAND:
                // 对wParam低字节进行判断，其中是我们的ID（参见MSDN）
                switch(LOWORD(wParam)) {
                // OK按钮
                case IDOK:
                        {
                                // 分配空间作为临时存储
                                char buf[512];

                                // 获得Edit Control内容
                                GetDlgItemText(hwnd, IDC_EDIT1, buf, 512);

                                // 用printf打印我们获得的内容
                                printf("%s\n", buf);

                                // 设置Static Text内容为我们刚获得的
                                SetDlgItemText(hwnd, IDC_DISPLAY, buf);
                        }
                        break;
                // 如果是Cancel按钮被按下
                case IDCANCEL:
                        // 这里读者可以充分发挥想象力，控制台函数可以调用 :)
                        break;
                }
                break;
        case WM_CLOSE:
                PostQuitMessage(0);
                break;
        }
        return 0;
}

int WINAPI WinMain(HINSTANCE hInst, HINSTANCE hPrev, LPSTR lpCmd, int nCmd)
{
        DialogBox(hInst, MAKEINTRESOURCE(IDD_DIALOG1), 0, DlgProc);
        return 0;
}

```

到这里，我们第二个Hello World版本也结束了，到这里我们已经学习了如何创建Windows对话框，初步的使用控件和消息的处理，可能大家还有一个疑问，为什么我们编译出来的程序如果在Windows下直接双击执行，会出现一个后面的黑色控制台，这不是很丑？其实这是MSYS开发Windows应用程序的独有优势，我们可以利用后面的控制台显示我们需要的printf内容，也就是传统开发者可以尽情的使用自己熟悉的控制台显示命令进行调试工作，我们当然有办法将这个控制台关闭，通过修改Makefile如下一行，或者手工编译命令：
```
LDFLAGS=-mwindows
```
```
# 链接的时候添加参数如下：
gcc main.o resource.o -mwindows  # -mwindows说明用windows窗口模式链接
```

好了，我们第二个Windows版本Hello World结束了，相信大家有了不少的收获，因为读到这里并且充分理解了的话，我们已经正式从C语言步入了Windows编程的领域 :)

# 4、第三个版本的Hello World #

在上一节中，我们领略了简单的对话框的创建与开发，虽然我们接触了消息机制，但是我们并没有实质性的接触消息机制的实现，在这个例子中我们将用另一种完全手工设计窗体的模式（不用resedit），实现一个用绘图函数写上的Hello World程序。

首先我们需要理解，在Windows窗体程序设计中，极大多数时候我们需要的是界面编程，因此上一节中的方法是界面编程的框架，我们现在使用的手工编程方法，是用于更加自主、裁剪化的方法设计窗体，一般对于做纯图形程序、游戏之类的可能用的比较多一些，比如3D编程（用OpenGL或者DirectX，我们将在后面的例子中进行介绍）。

下面我们开始winhello3，创建目录后我们就可以开始写入代码了，代码、分析如下：
```
#include <windows.h>

// 这是窗体版本的回叫函数，注意，与对话框的回叫函数有所不同
// 在窗体中，比如我们设计一些特殊控件、图形显示、3D模块等等，都是通过修改这个回叫函数实现的
LRESULT CALLBACK WndProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
	switch(uMsg) {
        // 窗口创建消息
	case WM_CREATE:
		MessageBox(hwnd, "Hello World!", "From MSYS", 0);
		break;
        // 对话框重新绘制消息，每当窗口更新时产生
	case WM_PAINT:
		{
			PAINTSTRUCT ps;
			HDC         hdc;    // 绘制设备句柄

                        // 开始绘制
			hdc = BeginPaint(hwnd, &ps);

                        // 用这个函数实现字符串的打印
			TextOut(hdc, 10, 10, "hello world from MSYS", 21);

                        // 结束绘制
			EndPaint(hwnd, &ps);
		}
		break;
	case WM_DESTROY:
		PostQuitMessage(0);
		break;
	default:
                // 窗口回叫函数中，不进行处理的消息，Windows需要回收，对话框不需要
		return DefWindowProc(hwnd, uMsg, wParam, lParam);
	}
        // 这里永远也不会走到
	return 0;
}

int WINAPI WinMain(HINSTANCE hInst, HINSTANCE hPrev, LPSTR lpCmd, int nCmd)
{
	WNDCLASS wc;     // 窗体类数据结构，这里我们说的类，与C++中的不是一个概念
	MSG      msg;    // 消息存储结构
	HWND     hwnd;   // 创建的窗口的句柄保存位置

        // 初始化设置窗口类
	wc.style         = CS_VREDRAW | CS_HREDRAW;
	wc.lpfnWndProc   = WndProc;
	wc.cbClsExtra    = 0;
	wc.cbWndExtra    = 0;
	wc.hInstance     = hInst;
	wc.hIcon         = LoadIcon(NULL, IDI_APPLICATION);
	wc.hCursor       = LoadCursor(NULL, IDC_ARROW);
	wc.hbrBackground = (HBRUSH)COLOR_WINDOW;
	wc.lpszMenuName  = NULL;
	wc.lpszClassName = "MyWndClass";    // 注意这里，这是我们任意定义的用于标识该自定义窗口类的方式

        // 向内核注册我们新创建的窗体类
	RegisterClass(&wc);

        // 用这个函数生成我们刚创建的窗体类，注意其第一个参数为我们的名字
	hwnd = CreateWindow("MyWndClass", "Hello from MSYS",
			WS_OVERLAPPEDWINDOW | WS_VISIBLE,
			CW_USEDEFAULT,
			CW_USEDEFAULT,
			CW_USEDEFAULT,
			CW_USEDEFAULT,
			NULL,
			NULL,
			hInst,
			0);

        // 这是经典的Windows消息泵，在这里消息被不断取出然后进行翻译和分配
	while(GetMessage(&msg, 0, 0, 0)) {
                // 翻译的主要工作是对不同的键盘扫描码进行转换
		TranslateMessage(&msg);

                // 这只是隐含方式的对WndProc的调用，将消息发送过去
		DispatchMessage(&msg);
	}

	return 0;
}
```

下面是我们的编译过程，在这里我们就不介绍Makefile的修改方法了，相信有心的读者已经不需介绍而明白了Makefile是怎么工作的，我们通过手工编译的指令介绍一些更多的深入的MSYS使用方法：
```
gcc main.c -o winhello2.exe
```

慢着，输入之后您会发现报告一个错误：
```
$ gcc main.c -o winhello2.exe
C:/DOCUME~1/Martin/LOCALS~1/Temp/cc6s8ih3.o:main.c:(.text+0x5b): undefined reference to `TextOutA@20'
collect2: ld returned 1 exit status
```

这个警告是说，在链接的过程中，LD链接器找不到我们调用的TextOutA函数，这是正确的，因为这个函数在编译时并没有默认的包含进Windows程序必要的库之中。这里通过这个例子我们介绍一个MSYS下的小技巧，我们可以通过grep搜索命令搜索gcc编译器所包含的所有的库，来找到具体哪个库包含有这个函数：
```
$ grep TextOutA /mingw/lib/*
grep: /mingw/lib/gcc: Invalid request code
Binary file /mingw/lib/libgdi32.a matches
Binary file /mingw/lib/libuser32.a matches
```

我们找到了两个库：libgdi32.a, libuser32.a，然后我们对这两个库进行链接，就可以解决这个错误：
```
gcc main.c -o winhello2.exe -lgdi32 -luser32
```

注意，包含外部的库，即时在VC上也有同样的工作需要做，所以不是MSYS的问题，一般库找不到的问题都可以通过这种搜索来解决，同时我们看到了作为控制台环境，编程所拥有的更多优势，我们甚至不需要让手指离开键盘就可以做任何事情 :)

这一章关于Windows编程的教程结束了，在下面我们会继续介绍Windows上用MSYS写DLL、设备驱动以及OpenGL、DirectX程序的方法。