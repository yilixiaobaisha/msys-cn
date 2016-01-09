[ENGLISH VERSION](EnglishHomepage.md)

---

# 项目简介 #

  * MSYS不是一个操作系统，而是一个通过将Linux源代码在Win32上编译而成的UNIX工作环境；
  * MSYS类似于Cygwin，但是由于工作原理的不同，速度更快、体积更小、功能强大、便于携带；
  * 使用MSYS可以完整的取代商业的VC环境，同样开发出商业版权的程序而不受任何版权限制；
  * 本项目开发了mpkg包管理程序，添加对各种开源库支持，提供MSYS发行版的方便下载服务；
  * 默认提供捆绑在一起的ASM/C/C++/ObjC/ObjC++/Fortran编译器，便捷的适用于各种用户；
  * 编译器中预先配置好了所需的DDK、DirectX 9 SDK、Pthread库，无需再自己搜集组合；
  * 更新的4.3.2版本GCC编译器为C/C++/Fortran用户提供了内建的OpenMP并行计算支持；
  * 本项目的初衷，是为Phoenix操作系统开发项目提开发环境，因而适合OS开发爱好者使用；
  * 项目通过GNU开发的info帮助系统提供了所有命令的帮助手册，开发者应当用MSDN作为函数手册；

  * **声明**：_本网站提供的内容全部遵循原作者的版权和发布协议；_
  * **注意**：_欢迎转载本站任何内容，不过请注明本官方网站位置，谢谢；_

> ## QQ群: 130103022，欢迎各位开发者、学习者加入一起学习！ ##
> ## [新系列 Win32开发教程](http://code.google.com/p/afx/) ##

---

# 近期更新 #

  * 2010.08.19 - 给 MSYS-CN 添加了中文显示支持，修改了几个有问题的地方
  * 2010.08.18 - 添加 GIT 版本控制系统支持，包名称为 git
  * 2010.07.31 - 添加Google的Golang编译器支持
  * 2010.07.23 - 移除了用处不大的perl，修正了.bz2文件染色bug
  * 2010.02.22 - 按用户指出的问题，修复了VIM不带help帮助的问题
  * 2010.01.12 - 操作系统开发技术交流活动通知，详见[Wiki活动页面](http://code.google.com/p/msys-cn/wiki/act_osdev)
  * 2010.01.11 - 由于某防火墙原因，取消Groups项目邮件列表，特此说明。
  * 2009.12.12 - MSYS系统更新完毕，更换了Rxvt和Vim至最新版，添加Perl, ssh支持。

---

# 高手速成教程 #

  1. [介绍、下载与安装方法](ChapterInstallation.md)
  1. [操作系统历史回顾](ChapterOne.md)
  1. [MSYS开发环境与开发工具](ChapterTwo.md)
  1. [基于MSYS环境的精短C语言教程](Chapter9.md)
  1. [基于MSYS、MingW的Windows编程教程](ChapterThree.md)
  1. [基于MSYS的 Flex & Bison （编译器开发工具）使用教程](ChapterFour.md)
  1. [基于MSYS的Win32动态链接库DLL开发](Chapter5.md)
  1. [基于MSYS的Windows驱动程序开发](Chapter10.md)
  1. [通用PID算法例子](Appendix1.md)
  1. [数据结构与算法快速教程](Appendix2.md)
  1. [让kqemu加速模块可以在windows的mingw上编译的代码补丁](Appendix3.md)
  1. [通用数学矢量类算法封装](Appendix4.md)
  1. [手写词法分析教程](Appendix5.md)
  1. [通用多源代码格式支持、依赖关系支持Makefile](Appendix6.md)

# 推荐链接 #

  * [纯Win32 API编程教程大全](http://winprog.org/tutorial/zh/start_cn.html)
  * [纯Win32 API教程系列](http://www.catch22.net)


---

# 截图 #

## 系统中各工具程序版本： ##
> http://msys-cn.googlecode.com/files/overview-1.JPG

## 显示官方仓库中现存的包： ##
> ![http://msys-cn.googlecode.com/files/overview-2.jpeg](http://msys-cn.googlecode.com/files/overview-2.jpeg)

## 输入函数名时，按下 Ctrl+x Ctrl+o 进行名称自动提示： ##
> ![http://msys-cn.googlecode.com/files/overview-3.jpg](http://msys-cn.googlecode.com/files/overview-3.jpg)

## 按下 F12 键更新数据库，然后输入 . -> 时进行自动提示： ##
> ![http://msys-cn.googlecode.com/files/overview-4.jpg](http://msys-cn.googlecode.com/files/overview-4.jpg)

## 可视化Windows窗体设计： ##
> ![http://msys-cn.googlecode.com/files/overview-5.jpg](http://msys-cn.googlecode.com/files/overview-5.jpg)

_最后更新：2013年10月24日_