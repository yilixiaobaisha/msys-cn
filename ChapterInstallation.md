# 安装方法 #

> 这里我们采用了7z格式压缩了所有需要解压缩的包，如果您打不开下载的文件，或者下载后解压缩出现问题，可能是您用的winrar版本过老，这些压缩包是没有问题的，强烈推荐您使用免费、开放源代码、压缩比更高的7z文件压缩工具：

> http://www.7-zip.org/

> 请先阅读本页最后的注意事项，然后安装下列基础包：
  * MSYS基础系统（已更新，2010.07.23）：http://msys-cn.googlecode.com/files/MSYS-Update.7z

# mpkg 包管理器使用方法 #

> mpkg --help打印的信息：
```
MSYS 包管理系统 版本 1.1
版权所有 (C) 2008, 合肥工业大学

使用:    mpkg 命令

命令:
  --install PACKAGE     安装 PACKAGE
  --remove  PACKAGE     删除 PACKAGE
  --add     PACKAGE     添加本地存储的.mpkg包，PACKAGE内容不包含".mpkg"扩展名
  --help                显示本消息
  --info                获得当前全部可下载、已安装的包列表

遇到缺陷请提交报告至: http://code.google.com/p/msys-cn/issues

```

# 附加软件包列表 #

> 经过2009.12.12更新，已经不再需要更新update-2包，请老用户注意！

> 扩展工具包
```
mpkg --install insight    # insight可视化调试器
mpkg --install SDL        # SDL游戏开发库
mpkg --install zlib       # zlib压缩库
mpkg --install mpfr       # GNU mpfr库
mpkg --install fftw       # FFTW库（快速傅立叶变换）
mpkg --install libsqlite  # Sqlite3库，本地SQL数据库支持
mpkg --install gmp        # GNU高精度浮点计算包
mpkg --install gsl        # GNU科学计算函数库
mpkg --install cmanual    # GNU C语言函数参考手册
mpkg --install libusb     # USB设备控制编程接口库
mpkg --install gnupg      # GNU PGP兼容软件
```

> 图形应用程序创建工具：
```
mpkg --install resedit    # 免费Windows可视化资源编辑器
mpkg --install fltk       # 跨平台图形程序库
mpkg --install fltk2      # 跨平台图形程序库2.0
mpkg --install wxWidgets  # 跨平台图形程序库
```

> 虚拟机包：
```
mpkg --install bochs      # Bochs虚拟机
mpkg --install qemu       # Qemu虚拟机
```

> 脚本开发
```
mpkg --install golang     # Google Go 语言
mpkg --install tcltk      # 跨平台脚本图形程序开发工具
mpkg --install tcltk86    # Tcltk脚本开发工具8.6版本
mpkg --install lua        # Lua嵌入式脚本工具包
```

> 版本控制程序
```
mpkg --install svn        # Subversion 版本控制软件
mpkg --install git        # Git 版本控制系统
```

> 文档生成工具
```
mpkg --install doxygen    # 自动代码文档生成工具
mpkg --install graphviz   # GNU方块图绘制软件
```

> 创建自己的发行包的实例教程
```
mpkg --install demo       # 演示
```

# 常见问题 #

  1. 如果您遇到任何问题可以首先加入我们与我们取得联系；
  1. 您的系统使用的用户名、目录路径不能有空格、中文，否则将导致安装失败；
  1. 如果您需要我们提供开源的其它类型的工具包，请提交到我们的问题报告中；
  1. 如果您使用代理服务器上网，要下载附加包请先输入命令：
```
export http_proxy=http://用户名:密码@代理地址
```