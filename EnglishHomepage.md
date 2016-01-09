[中文版本](http://code.google.com/p/msys-cn/)

# INTRODUCTION #

  * This site provides a full Minimal SYStem originally from www.mingw.org, we programmed a package management system _mpkg_ to manage and distribute packages.
  * The MSYS is compatible from user's view with Linux and UNIX development environments. Users can do Linux/UNIX shell works for educational and development purpose.
  * The compiler package supplies C/C++/Fortran compilers by defalt.

  * If you are interested providing us other tools such as packages and cross compiler support, or even interested in becoming a developer, please contact us through ways listed below.
  * If you have project or tools hosted on your space and wanted to be part of the distribution for better support, please contact us too.

  * **NOTICE: ALL binary packages supplied with this project has been compiled from their unmodified sources downloaded directly from each official site, and follows the same license as published by the author, if you need the source code you should go directly to their official site, if we have violated your rights, please contact us, we will arrange a deletion of the specified package, thanks.**

# CONTACT #

  1. QQ IM Group    : 18266455
  1. Technical Forum: http://www.woos.cn/bbs/thread.php?fid=34
  1. Maillist       : [phoenix-development@googlegroups.com](http://groups.google.com/group/phoenix-development)

# READ FIRST #

Read _NOTICE_ in the end of this page first, then you need to install following packages manually:
  1. MSYS Base System : http://msys-cn.googlecode.com/files/MSYS-Update.rar

The packaging system is based on _mpkg_, the tool is written in bash shell and is still at a initial stage, not including abilities such as solving dependency issues. Though, all the packages has been compiled with statical link to reduce dependency problem.

_mpkg_ Usage:
```
MSYS Package Management System Version 1.1
Copyright (C) 2008, Huaxin Tang
Copyright (C) 2008, Hefei University of Technology, China

USAGE:    mpkg COMMANDS

COMMANDS:
  --install PACKAGE     Install a package to local repository
  --remove  PACKAGE     Delete a package from local repository
  --add     PACKAGE     For demo.mpkg: mpkg --add demo
  --help                Print this message and exit
  --info                Get current full package list

Report bugs to website: http://code.google.com/p/msys-cn/issues

```

> Essential development suit:
```
mpkg --install insight
mpkg --install SDL
mpkg --install zlib
```

> FLTK1/2 GUI Development Toolkit:
```
mpkg --install fltk
mpkg --install fltk2
```

> Virtual machine packages:
```
mpkg --install bochs
mpkg --install qemu
```

> TclTk:
```
mpkg --install tcltk
```

> Automatic documentation tool:
```
mpkg --install doxygen
mpkg --install graphviz
```

> HOWTO for creating your own package distributions:
```
mpkg --install demo
```

# NOTICE #

  1. Your username, installation path must not contain white space, unicode character, or else, the installation fails, or will not working for building specific packages.
  1. The MingW compiler as provided is a TDM version, it contains only a partial POSIX system calls, lake of some essential calls such as fork, but with a full version of Windows API.
  1. If you need a Linux cross compiler, or embedded system development toolchain, please watch our distros or compile it yourself, if you want to share with others, contact us.

# SCREENSHOTS #

## Displaying versions of tools ##
> http://msys-cn.googlecode.com/files/overview-1.JPG

## Available package lists ##
> ![http://msys-cn.googlecode.com/files/overview-2.jpeg](http://msys-cn.googlecode.com/files/overview-2.jpeg)

## When entering function name, press Ctrl+x Ctrl+o to autocomplete ##
> ![http://msys-cn.googlecode.com/files/overview-3.jpg](http://msys-cn.googlecode.com/files/overview-3.jpg)

## After pressing F12 updating the tags database, input '.' '->' to intellisense ##
> ![http://msys-cn.googlecode.com/files/overview-4.jpg](http://msys-cn.googlecode.com/files/overview-4.jpg)

## Visual Windows form design ##
> ![http://msys-cn.googlecode.com/files/overview-5.jpg](http://msys-cn.googlecode.com/files/overview-5.jpg)

Last Updated: 2008.12.15