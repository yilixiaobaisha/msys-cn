# 介绍 #

我们在第三章中介绍了一种通用Makefile的写法，但是那个Makefile仍然是最基础形式的，除了能按照需求完成简单的编译之外，不能自动的根据依赖关系更新目标文件。

在这里，我们所说的依赖关系支持，是指比如源代码中的 test.c 文件 include 了 test.h 文件，但是在修改代码的过程中我们很有可能只修改了 test.h 而没有修改 test.c，因而输入 make 命令之后，test.c 文件并不能自动被重新编译更新，导致我们的 Makefile 不能完美的完成它的使命。

在大多数 Linux 代码包中普遍采用了基于依赖关系的自动编译方法，当然依赖关系的具体实现方式也不同，有的采用 mkdepend 命令来实现，有的采用 gcc 的自动依赖生成来实现，在这里我们采用的是后者的方法。

同时，在使用上这个 Makefile 与第三章中所描述的没有太大的区别，唯一的区别就是多了一些代码，而这些代码不需要用户来进行维护。

为了尽量扩大这个Makefile的使用范围，笔者扩展了这个通用 Makefile 支持了更多的源代码格式：
  1. ".s" 汇编代码
  1. ".c" C 语言代码
  1. ".cpp" C++ 代码
  1. ".f" Fortran 代码
  1. ".rc" Windows 资源文件代码
  1. ".m" Objective C/C++ 源代码
  1. ".go" Google Go 源代码

# Martin版本源代码 #
```
###############################################################################
# 通用依赖关系驱动 MAKEFILE for MSYS
#
# 当前支持的代码格式:
#  *.s, *.c, *.cpp, *.f, *.rc, *.m, *.go
#
# PROVIDED WITH NO WARRANTY OF ANY KIND, AND NO COPYRIGHT RESTRICTIONS
###############################################################################

# 工具链配置（可以修改，比如 C 语言 LD=gcc，C++ 语言 LD=g++ 等等
PP=cpp
AS=as
CC=gcc
CX=g++
FC=gfortran
RC=windres
MC=windmc
8G=8g
8L=8l
LD=gcc
DB=insight

# 编译器配置（这里基本不需要修改，熟悉编译参数的用户可以酌情修改）
INCLUDE=
ASFLAGS=$(INCLUDE) -g
CCFLAGS=$(INCLUDE) -g
CXFLAGS=$(INCLUDE) -g
FCFLAGS=$(INCLUDE) -g
8GFLAGS=$(INCLUDE) -g
8LFLAGS=$(INCLUDE) -g
RCFLAGS=
LDFLAGS=

# 文件对象（创建项目只需要修改这里，下面是举例）
OBJECT=Main.o     # 填写源代码Main.cpp对应生成的.o文件名，后面可以继续添加，用 \ 换行
TARGET=App.exe    # 填写链接所有.o文件后最终生成的exe文件名
DEPEND=$(OBJECT:.o=.dep)

# 编译命令（这里提供了标准的命令，熟悉的用户可以自行添加）
all : $(TARGET)
	@$(RM) $(DEPEND)

$(TARGET) : $(OBJECT)
	$(LD) -o $@ $^ $(LDFLAGS)

debug: all
	@$(DB) $(TARGET)

run: all
	@$(TARGET)

clean:
	@$(RM) $(OBJECT) $(DEPEND) $(TARGET)

# 标准处理过程（以下内容为内部机制，用户请勿修改）
%.dep : %.s
	@$(PP) $(INCLUDE) -MM -MT $(@:.dep=.o) -o $@ $<

%.dep : %.c
	@$(PP) $(INCLUDE) -MM -MT $(@:.dep=.o) -o $@ $<

%.dep : %.m
	@$(PP) $(INCLUDE) -MM -MT $(@:.dep=.o) -o $@ $<

%.dep : %.cpp
	@$(PP) $(INCLUDE) -MM -MT $(@:.dep=.o) -o $@ $<

%.dep : %.f
	@$(PP) $(INCLUDE) -MM -MT $(@:.dep=.o) -o $@ $<

%.dep : %.rc
	@$(PP) $(INCLUDE) -MM -MT $(@:.dep=.o) -o $@ $<

%.o : %.s
	$(AS) $(ASFLAGS) -o $@ $<

%.o : %.c
	$(CC) $(CCFLAGS) -c -o $@ $<

%.o : %.m
	$(CC) $(CCFLAGS) -c -o $@ $<

%.o : %.cpp
	$(CX) $(CXFLAGS) -c -o $@ $<

%.o : %.f
	$(FC) $(FCFLAGS) -c -o $@ $<

%.o : %.rc
	$(RC) $(RCFLAGS) $< $@

%.8 : %.go
	$(8G) $(8GFLAGS) -o $@ $<

%.exe : %.8
	$(8L) $(8LFLAGS) -o $@ $<

-include $(DEPEND)
```