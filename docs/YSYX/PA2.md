- [RTL实现单周期处理器](#rtl实现单周期处理器)
- [程序，运行时环境与AM](#程序运行时环境与am)
- [基础设施](#基础设施)
  - [bug诊断的利器-trace](#bug诊断的利器-trace)
    - [ftrace](#ftrace)
  - [AM作为基础设施](#am作为基础设施)
  - [Differential Testing](#differential-testing)
  - [一键回归测试](#一键回归测试)

# RTL实现单周期处理器
之前的计组实验课有写过五级流水线处理器，所以这个单周期处理器实现起来还算简单，相比于之前写五级流水线处理器，这次我对译码操作理解更加深入。

不足的是一生一芯讲义建议尽量不要用行为建模，但是我自己在写的时候就是想用case和if语句等。

最后完成仿真部分让我对verilator的使用更加熟悉，也学会了DPI-C机制，在RTL代码中调用C++文件中实现的函数，但是要注意的是，在C++文件中声明函数时要加上`extern "C"`

*** 
# 程序，运行时环境与AM
PA2.2中makefile文件特别值得学习，看懂makefile就能够理解nemu，am-kernels，abstract-machine之间的关系，根据我目前的理解，am-kernels提供一些c文件，也就是客户程序，nemu是模拟的处理器，abstract-machine所做的就是提供客户程序在nemu中运行起来的环境，比如库文件等等。

初次看abstract-machine的Makefile文件时，我只看懂了一些诸如把.c文件编译成.o文件的规则。当我想尝试无脑直接输入`make`时,命令行告诉我没有指定ARCH。于是我再输入`make ARCH=riscv32-nemu`，命令行又告诉我没有SRCS，在这一点我卡了很久，尝试了直接输入`make ARCH=riscv32-nemu SRCS=dummy`,发现没什么用。最后发现是要去am-kernels文件夹下指定客户程序，比如我在`~/ysyx-workbench/am-kern- [RTL实现单周期处理器](#rtl实现单周期处理器)
- [RTL实现单周期处理器](#rtl实现单周期处理器)
- [程序，运行时环境与AM](#程序运行时环境与am)
- [基础设施](#基础设施)
  - [bug诊断的利器-trace](#bug诊断的利器-trace)
    - [ftrace](#ftrace)
  - [AM作为基础设施](#am作为基础设施)
  - [Differential Testing](#differential-testing)
  - [一键回归测试](#一键回归测试)
```
# Building hello-image [riscv32-nemu]
+ CC hello.c
# Building am-archive [riscv32-nemu]
+ CC src/platform/nemu/trm.c
+ AR -> build/am-riscv32-nemu.a
# Building klib-archive [riscv32-nemu]
+ LD -> build/hello-riscv32-nemu.elf
# Creating image [riscv32-nemu]
+ OBJCOPY -> build/hello-riscv32-nemu.bin
```
我根据输出的结果又重新看了一遍abstract-machine目录下的Makefile文件，得出以下理解：
- 首先编译am-kernels中的客户程序
- 编译am和klib中的.c文件并打包成.a文件
- 链接客户程序编译出的.o文件和打包的库文件.a

在这个过程中，我发现Makefile文件类似于函数，供其他Makefile文件调用。
```
-include $(AM_HOME)/scripts/$(ARCH).mk
@$(MAKE) -s -C $(AM_HOME)/$* archive
```
Makefile中的这两句特别巧妙，实现了Makefile文件的交互
- include可以包含其它makefile文件的一些变量
- $(MAKE) -C意为在其它makefile文件中执行make指令

最后生成了二进制文件后，我又遇到了一个问题，该怎么执行呢。
```
~/ysyx-workbench/am-kernels/tests/cpu-tests$ make ARCH=riscv32-nemu ALL=dummy run
```
最后的run命令会调用abstract-machine/scripts/platform/nemu.mk里的run规则

# 基础设施
## bug诊断的利器-trace
### ftrace
实现ftrace有几个难点：
*** 
难点1：如何采用menuconfig来设置是否进行ftrace

这里我回顾了之前PA1讲menuconfig的内容，然后根据`nemu/Kconfig`里的内容依葫芦画瓢增加了ftrace的一个配置选项，最后在`abstract-machine/Makefile`中为`CFLAGS_TRACE`增加ftrace相关的宏
***
难点2：如何让nemu获取ftrace所需要的elf文件以及相应的txt输出文件

首先在parse_args()函数中增加相应的选项，然后在`abstract-machine/scripts/platform/nemu.mk`文件下给`NEMU_FLAGS`增加相应的命令
***
难点3：ftrace记录的信息如何获取

首先要在`nemu/inst.c`执行`jal`和`jalr`的时候添加相应的操作，然后解析elf文件获取符号表（[参考了网上大佬的](https://blog.csdn.net/qq_57973134/article/details/135536952)），采用结构体symbol，成员变量有`name`表示函数名，`addr`表示函数地址，`size`表示函数所占用存储空间大小。通过对比符号表中函数的地址和当前指令跳转的地址来确定是调用或者返回哪个函数。

elf文件的解析以及函数名匹配都写在了`nemu/utils/trace.c`中

## AM作为基础设施
在native上运行AM，可以验证AM中Klib程序是否正确
```
make ALL=string ARCH=native run
```
当我执行上述命令时有很多报错，我也卡了很久。
1. 在string.c中自己实现了一个strnlen的库函数，但是编译运行的时候会报错与<string.h>中的strnlen函数冲突。所以我只能删了这个函数，但是其它自定义的库函数为什么不会冲突，我暂时还没有理清楚
2. malloc函数的实现有问题，后来参考了网上别人写的，发现他写的时候有注意地址对齐

## Differential Testing
这个比较简单，读懂代码，然后只需要实现ISA-dependent的`isa_difftest_checkregs()`函数

## 一键回归测试
