# 1、介绍 #

## **警告：驱动程序开发对不熟悉的人来说是危险的，潜在会造成硬件的损坏和数据的丢失** ##
## **请读者从这里开始三思而后行，如果造成以上描述的后果，笔者声明将不承担任何责任** ##

世界上程序员总共有三种不同的工作，一种是操作系统内核开发程序员（比如微软内核部门和Linus Tovalds所从事的工作），又称核心开发者；一种是应用程序开发程序员，又称应用开发者，让用户通过该程序能够完成各种工作中需要完成的任务；最后一种就是驱动程序开发程序员(Driver Developer)，又称接口开发者，为操作系统内核和应用程序之间提供硬件和用户交互的桥梁，让内核能够通过驱动驾驭硬件从而为应用程序提供控制和访问服务。

对于每一个操作系统来说，都有一套自己独特的驱动机制，虽然各有自己的风格和自己的安全保护、加载机制，但是对于程序员来说，所要解决的问题都是相同的，那就是：

  1. 在内核权限中拥有对硬件设备的访问权限;
  1. 初始化硬件设备到预定的功能模式;
  1. 为用户提供用户权限中操作该设备的各种指令和方法;
  1. 为卸载驱动程序提供安全的出口;

以我们天天在用的网络浏览器为例，当用户输入网站地址的时候，浏览器程序获取到该信息后立即会将访问网络的信息传递给操作系统下的驱动程序，在这里，驱动程序包有两种，含协议驱动（TCP/IP协议、Socket接口）以及硬件设备驱动（网卡驱动），协议将内容转化为固定的网卡操作序列，然后该序列被输入网卡驱动程序，最终网卡通过与外界连接的网线进行通讯从而获得必要的信息，然后该信息又被逆过来再次通过这些层最终到达浏览器，浏览器对获取的数据进行处理和显示，完成了一次用户浏览网页的操作。

从上面的例子我们可以看出，对硬件的操作主要涉及到三种软件：

  1. 访问驱动程序的应用程序;
  1. 我们的驱动程序;
  1. 操作系统内核;

在笔者长期的开发中，由于VC这个官方开发工具尺寸的庞大还有自己总是懒得装让系统变慢的.net Framework，同时MingW编译器已经足够强大到我个人在Windows上开发应用程序的每一个方面，因而已经有较长时间用MSYS+MingW环境替代掉了过去的VC 6.0开发环境。

后来由于某次项目中需要开发设备驱动，故开始研究起了XP DDK，这个工具包也是官方的，而且功能非常强大，但是问题同样是体积太大了，太占硬盘空间，用MingW + MSDN的开发环境用了多年比较顺手，所以研究了一下MingW中的Win32API结构，兴奋地发现了include目录下有ddk的目录，而且基本拥有了win2k ddk中的所有头文件和函数（少数调试指令如try，except仍然是缺失），所以开始研究起了Win32驱动的MingW编译器开发方法。

MingW中的ddk最早（而且现在也）是为ReactOS（一个开源的Windows操作系统克隆）操作系统开发中的驱动开发所准备的，由于ReactOS项目的目标是完全实现与Win32架构的操作系统二进制兼容，而且驱动也兼容，所以MingW中的ddk是可以用来在Windows上开发真正的本土Win32驱动的。

经过本人半年来的研究和探索，同时参考了多个论坛上的数篇文章，笔者在本文下面的篇幅中，将基于MSYS中的gcc编译器开发驱动开发中涉及到的三种程序的代码，并且将驱动开发中涉及到的各种技术和方法一一阐述，需要注意的是，本文比较怪异的介绍了Windows操作系统下用gcc（MingW）编译器开发Windows驱动的开发方法和调试方法，因而希望读者不要误以为我们是在开发Linux或者其他UNIX驱动 :)

# 2、Windows 内核模式驱动（KMD）的MSYS + MingW开发方法 #

首先我们需要编写驱动程序的代码，读者可以参考MSDN手册上，按照本系列教程中之前的多个教程的方法，通过对下面代码中所涉及到的多个函数搜索和研究，下面我们直接给出最简单的Windows WDM KMD驱动的框架代码：

```
/*****************************************************************************
 * Copyright (C) 2009, 唐华欣
 *
 * 本代码使用GPLv2协议，作者不对使用该代码导致的任何后果负任何责任
 *****************************************************************************/
#include <ddk/ntddk.h>

// 一些简单的声明，这些函数都在后面会进行详细的介绍
// 注意，所有被Windows呼叫的函数必须是STDCALL属性（DLL类似）
NTSTATUS STDCALL test_create(PDEVICE_OBJECT DeviceObject, PIRP Irp);
NTSTATUS STDCALL test_close (PDEVICE_OBJECT DeviceObject, PIRP Irp);
NTSTATUS STDCALL test_read  (PDEVICE_OBJECT DeviceObject, PIRP Irp);
NTSTATUS STDCALL test_write (PDEVICE_OBJECT DeviceObject, PIRP Irp);
NTSTATUS STDCALL test_ioctrl(PDEVICE_OBJECT DeviceObject, PIRP Irp);
VOID     STDCALL test_unload(PDRIVER_OBJECT DriverObject);

// Windows驱动程序入口函数（初始化加载时调用）
NTSTATUS STDCALL
DriverEntry(PDRIVER_OBJECT DriverObject, PUNICODE_STRING RegistryPath)
{
  PDEVICE_OBJECT DeviceObject;  // 设备对象句柄
  UNICODE_STRING DeviceName;    // 设备名称字符串
  UNICODE_STRING SymlinkName;   // 符号链接字符串（在文件系统中的位置）
  NTSTATUS Status;              // 本函数操作状态返回值

  // 驱动程序调试显示代码，功能与用法与应用程序开发中的printf函数相似
  // 在后面的代码中我们都将大量使用这个函数
  DbgPrint("DriverEntry invoked\n");

  // 注册几个主要的设备操作方法
  // 设备创建函数
  DriverObject->MajorFunction[IRP_MJ_CREATE]         = test_create;
  // 设备关闭函数
  DriverObject->MajorFunction[IRP_MJ_CLOSE]          = test_close;
  // 特殊控制模式函数
  DriverObject->MajorFunction[IRP_MJ_DEVICE_CONTROL] = test_ioctrl;
  // 设备写入函数（写入硬件）
  DriverObject->MajorFunction[IRP_MJ_WRITE]          = test_write;
  // 设备读出函数（从硬件读出）
  DriverObject->MajorFunction[IRP_MJ_READ]           = test_read;
  // 设备卸载函数
  DriverObject->DriverUnload                         = test_unload;

  RtlInitUnicodeString(&DeviceName, L"\\Device\\test");
  RtlInitUnicodeString(&SymlinkName, L"\\DosDevices\\test");

  Status = IoCreateDevice(DriverObject, 0, &DeviceName, FILE_DEVICE_UNKNOWN,
      0, FALSE, &DeviceObject);

  if (!NT_SUCCESS(Status)) {
    return Status;
  }

  // 设置设备访问模式
  //   在Windows驱动模型中，有两种IO访问模式，一种是直接IO访问（DO_DIRECT_IO）
  //   另外一种是带缓冲的IO（DO_BUFFERED_IO），从名字可知，前者比后者速度要快
  //   一点，但是如果数据过多会造成数据的紊乱和丢失，而后者则正好相反。
  //
  //   在这里，我们使用带缓冲的模式：
  DeviceObject->Flags |= DO_BUFFERED_IO;

  // 在文件系统中创建本驱动的操作符号连接（虚拟文件，代表硬件设备）
  // 这里其实就类似Linux下驱动开发的/dev/xxx的创建
  Status = IoCreateSymbolicLink(&SymlinkName, &DeviceName);

  // 如果创建不成功的话，退回以前所有操作
  if (!NT_SUCCESS(Status)) {
    IoDeleteDevice(DeviceObject);
    return Status;
  }

  return STATUS_SUCCESS;
}

// 下面我们首先编写设备驱动的卸载函数
//   当我们停止一个驱动的工作时，该函数将被内核调用，用于取消初始化中所作的
//   一切工作。
VOID STDCALL test_unload(PDRIVER_OBJECT DriverObject)
{
  UNICODE_STRING SymlinkName;

  // 用调试打印工具显示调用状态
  DbgPrint("test_unload invoked\n");

  // 初始化UNICODE表示的设备虚拟文件地址
  RtlInitUnicodeString(&SymlinkName, L"\\DosDevices\\test");

  // 删除该虚拟文件
  IoDeleteSymbolicLink(&SymlinkName);

  // 删除本驱动对象
  IoDeleteDevice(DriverObject->DeviceObject);
}

// 由于内核中如果数据操作有任何溢出缓冲区尺寸的情况都将导致危险，我们需要用
// 下面的函数来检查字符串是否溢出缓冲区
BOOLEAN test_IsStringTerminated(PCHAR src, UINT size)
{
  int i = 0;

  while(i < size) {
    if(src[i] == '\0') {
      return TRUE;
    }

    i++;
  }

  return FALSE;
}

// 然后我们集中精力在写入数据函数上，该函数在用户程序需要向设备中写入数据
// 时使用，我们这里让被写入的数据字符串从调试工具中打印出来
NTSTATUS STDCALL test_write (PDEVICE_OBJECT DeviceObject, PIRP Irp)
{
  // 首先准备所需要的各种数据（gcc编译器中也可以随时需要随时申请，必须要写在
  // 最前面）
  PIO_STACK_LOCATION pIoStackIrp = NULL;            // IRP堆栈位置
  PCHAR pWriteDataBuffer;                           // 写入的数据缓存

  // 用调试打印工具显示调用状态
  DbgPrint("test_write invoked\n");

  // 重新获取IRP堆栈的位置（每次进入本函数时都不同）
  pIoStackIrp = IoGetCurrentIrpStackLocation(Irp);

  // 如果获取成功，则进行如下操作：
  if(pIoStackIrp) {

    // 获得缓存地址，方便操作
    pWriteDataBuffer = (PCHAR)Irp->AssociatedIrp.SystemBuffer;

    // 如果缓存存在
    if(pWriteDataBuffer) {
      if(test_IsStringTerminated(pWriteDataBuffer, pIoStackIrp->Parameters.Write.Length)) {
        DbgPrint(pWriteDataBuffer);
      }
    }
  }

  return STATUS_SUCCESS;
}

// 下面的函数我们就不进行深入的研究了，只留一个函数原形在这里给有兴趣
// 深入研究的朋友参考

// 设备创建函数（被创建时调用）
NTSTATUS STDCALL test_create(PDEVICE_OBJECT DeviceObject, PIRP Irp)
{
  // 用调试打印工具显示调用状态
  DbgPrint("test_create invoked\n");

  return STATUS_SUCCESS;
}

// 设备关闭函数（关闭使用对象时调用）
NTSTATUS STDCALL test_close(PDEVICE_OBJECT DeviceObject, PIRP Irp)
{
  // 用调试打印工具显示调用状态
  DbgPrint("test_close invoked\n");

  return STATUS_SUCCESS;
}

// 读出数据函数
NTSTATUS STDCALL test_read  (PDEVICE_OBJECT DeviceObject, PIRP Irp)
{
  // 用调试打印工具显示调用状态
  DbgPrint("test_read invoked\n");

  return STATUS_SUCCESS;
}

// 特殊控制指令（read、write之外）用函数
NTSTATUS STDCALL test_ioctrl(PDEVICE_OBJECT DeviceObject, PIRP Irp)
{
  // 用调试打印工具显示调用状态
  DbgPrint("test_ioctrl invoked with code: %x\n", Irp);

  return STATUS_SUCCESS;
}
```

下面我们需要将这个代码编译出来，我们需要在MSYS下面使用如下的指令：
```
gcc -shared -Wl,--subsystem,native -Wl,--entry,_DriverEntry@8 -nostartfiles -nostdlib test.c -o test.sys -lntoskrnl
```

指令中具体的信息，希望读者自己去研究，如果您对DDK开发驱动比较熟悉的话，相信也不会很陌生 :)

编译完成后，我们就能够得到一个名叫test.sys的代码。

# 3、驱动加载器的开发 #

在上一部分中，我们完成了一个简单的Windows驱动的开发过程，其中代码与使用DDK开发方式并没有太大的差别，只有一些细微的编写方法有所不同，下面，为了测试编译出来的test.sys是否正确、能否使用，我们需要开发一个驱动加载器。在Linux下开发驱动的朋友一定都很熟悉了insmod这个命令，然而windows中没有这样的命令，所以我们下面要写的就是这样一个程序：

```
/*****************************************************************************
 * Copyright (C) 2009, 唐华欣
 *
 * 本代码使用GPLv2协议，作者不对使用该代码导致的任何后果负任何责任
 *****************************************************************************/
#include <windows.h>
#include <stdio.h>

// *读者注意*：请将下面的地址修改为您的test.sys存放的地址
char path[] = "D:\\Archive\\testdrv\\test.sys";

int main(void)
{
    SERVICE_STATUS ss;

    // 首先，我们申请服务管理器的句柄
    HANDLE hSCManager;
    hSCManager = OpenSCManager(NULL, NULL, SC_MANAGER_CREATE_SERVICE);
    
    // 如果申请成功
    if(!hSCManager) {
      return FALSE;
    }

    // 如果未为该驱动创建服务，尝试创建
    HANDLE hService;
    hService = CreateService(hSCManager, "Example", "Example Driver", 
        SERVICE_START | DELETE | SERVICE_STOP,
        SERVICE_KERNEL_DRIVER,
        SERVICE_DEMAND_START,
        SERVICE_ERROR_IGNORE,
        path, NULL, NULL, NULL, NULL, NULL);

    // 如果创建失败，则说明已经创建，我们直接打开
    if(!hService) {
      hService = OpenService(hSCManager, "Example",
          SERVICE_START | DELETE | SERVICE_STOP);
    }

    // 如果获取服务句柄成功，进行操作
    if(hService)
    {
      // 启动驱动
      printf("驱动注册成功，下面启动该驱动\n");
      StartService(hService, 0, NULL);

      // 等待用户按键，然后关闭驱动
      printf("按任意键停止该驱动\n");
      getchar();

      // 停止驱动的工作
      ControlService(hService, SERVICE_CONTROL_STOP, &ss);

      // 释放各种句柄
      CloseServiceHandle(hService);
      DeleteService(hService);
    }

    // 释放服务管理器句柄
    CloseServiceHandle(hSCManager);

    return 0;
}
```

如何察看DbgPrint输出的调试信息？笔者推荐读者使用www.sysinternals.com推出的DbgView工具，下载地址为：

http://technet.microsoft.com/en-us/sysinternals/bb896647.aspx

启动该工具，选中察看内核信息，然后反复使用几次load进行加载，大家是不是看到我们刚才打印的信息了？

# 4、应用程序访问 #

Windows下访问设备驱动与UNIX没有本质的区别，只需要对设备虚拟文件（符号连接）当作普通文件进行操作即可：
```
/*****************************************************************************
 * Copyright (C) 2009, 唐华欣
 *
 * 本代码使用GPLv2协议，作者不对使用该代码导致的任何后果负任何责任
 *****************************************************************************/
#include <windows.h>

// 在Windows下访问设备驱动也并不复杂，与Linux/UNIX下面一样，直接当作文件来进行
// 读写即可
int main(void)
{
    HANDLE hFile;
    DWORD dwReturn;

    // 打开设备驱动虚拟文件（对应test_create）
    hFile = CreateFile("\\\\.\\test", GENERIC_READ | GENERIC_WRITE, 0, NULL, OPEN_EXISTING, 0, NULL);

    // 如果打开文件成功
    if(hFile)
    {
      // 执行写入（对应test_write）
        WriteFile(hFile, "Hello from user mode!", sizeof("Hello from user mode!"),
           &dwReturn, NULL); 

        // 关闭设备虚拟文件句柄（对应test_close）
        CloseHandle(hFile);
    }
    
    return 0;
}
```

怎么样，是不是在DbgView中看到我们写入的Hello from user mode!字符串了？

# 5、后记 #

本文是笔者半年来的研究加上各种国内外论坛中的代码和各方文字资料编写而成的，特别是参考了www.codeprojects.com网站、看雪学院、kqemu代码、KmdKit等多个软件包的资料，为以后开源产业内大规模的MingW应用奠定了基础，希望能够对读者有所帮助同时欢迎大家与我交流，愿我们在技术领域共同成长。