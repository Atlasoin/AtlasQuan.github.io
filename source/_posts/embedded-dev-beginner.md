---
title: 嵌入式开发入门指北 | 让NUCLEO-F429ZI的小灯闪烁
date: 2020-03-022 12:06:00
tags: 
- Embedded
- Nucleo
- Open Source
---

本篇文章是自己刚刚入坑嵌入式开发的一点总结和分享，内容非常基础，只讲到如何成功在开发板上运行一个最简单的使小灯闪烁的程序为止（类似hello world）。我算是有一些computer science的基础，但是对于嵌入式开发来说是纯新手，所以这篇文章更侧重从程序员的视角来介绍如何配置环境(使用基于Linux的开源的工具链），跑通整个开发的流程，而并不特别关注具体的开发。如果你是和我一样的背景，希望这篇博客可以对你有帮助。如果你需要更具体深入的开发指导，这篇文章不适合你。

## 嵌入式开发背景介绍

嵌入式开发就是在嵌入式操作系统上开发，嵌入式系统呢，就是一个包括了处理器，内存和输入输出的系统，说白了，就是一块芯片，一个和我们常用的计算机内部的那个芯片差不多的东西。他们具体的差距就在于架构（architecture），比如我们通常的计算机芯片是x86-64的架构，而嵌入式系统一般基于arm架构的会更多一点）。综上，嵌入式开发，可以理解为为另一种类型的计算机开发程序，这种计算机和我们平常开发程序的计算机大同小异（毕竟都是冯诺依曼体系嘛，不难理解的）。

那么现在我们要开始开发了，首先我们选定一个芯片。从我另一位搞机器人的朋友得知，stm32是目前比较主流的芯片系列，其中stm32 f4系列比较适合入门（stm是一个芯片厂商的品牌名称，32代表32位，f4是它的具体型号，表示基于arm cortex-M4F的内核）。

等等，光一块芯片怎么开发，怎么把这个芯片和我们的计算机怼到一起？要自己焊接什么东西吗？别急，最初的学习阶段不需要担心电路方面的问题，我们只需要一个开发板就可以了。简单理解，开发板就是厂商预先给芯片连接了很多外设（比如小灯）的板子，方便我们开发和调试。等我们把代码都在开发板上调试好了，真正生产产品的时候，我们可以直接把程序下载到最小系统板（minimum system board）上。最小系统板自带最基础的外围电路，如下图所示，这种是我们个人制作一些小东西的选择，如果是工厂量化生产产品的话会使用裸片（bare chip），然后工厂再自己焊接必要的电路。
![最小系统板](https://ae01.alicdn.com/kf/HTB1LqVzXvfsK1RjSszbq6AqBXXaI/STM32F103C8T6-ARM-STM32-Minimum-System-Development-Board-Module-for-arduino-DIY-KIT.jpg)
![裸片](https://1.bp.blogspot.com/-7RTG4Rbd9uQ/Vu19R4FSwzI/AAAAAAAAAiY/aTK-9z6qeawoHmKyP7fuI6vzX8BRXEERA/s1600/stm32f103rbt6_bare.jpg)

我选择的开发板是[NUCLEO-F429ZI](https://www.st.com/en/evaluation-tools/nucleo-f429zi.html)。Nucleo是stm官方的开发板，价格不算很贵，而且板子自带st-link（一个调试器，后文还会提到），但外设比较少。

![NUCLEO-F429ZI开发板](https://www.st.com/bin/ecommerce/api/image.PF262637.en.feature-description-include-personalized-no-cpn-large.jpg)

## 开发之前，我们先思考一下这个过程

这个开发的大体的流程呢，就是在我们自己的计算机上编译好程序，然后把这个程序写入芯片的内存中即可。

之前也说了，我们平常写的程序是在自己的x86-64架构的计算机上运行，现在我们需要写能在arm架构运行的程序怎么办呢？这就是交叉编译了，怎么写代码没有什么特殊的，语法还是那些语法啊（你问用什么语言写？C/C++）只不过我们需要一些**交叉编译的工具**，使我们的代码可以在arm架构上运行。（这里先理清概念，具体的工具和指令后文再提）（关于语言，我个人的理解是其实也没有人规定你一定要用什么语言写，但是要交叉编译的话C和C++应该是工具链最完善最方便的语言了）。

我们继续思考，在那个和我们的架构不一样的arm架构的计算机上，我们怎么知道如何读取输入输出呢？平常我们写``#include <stdio.h>``，那现在我们include什么？此外，特别要注意的是，我们平常的写的代码需要运行的平台都是有操作系统的，操作系统会帮助我们初始化一些寄存器和重要的数据结构之类的，但是stm32 f4芯片没有啊，那我们该怎么办呢？幸运的是这些任务都有现成的库函数，我们也不太需要干什么（yeah！），只需要调用一下**firmware library**就好了！

接下来假设我们已经写好了程序，并且我们已经有了开发板，这个开发板上已经有足够支撑我们基础学习的外设了。那么首先，我们要把这个开发板和我们的计算机怼到一起，这需要一根**TypeA-MicroB**的线就行。
![TypeA to MircoB](https://cdn.worldcordsets.com/products/usb-20-type-a-male-to-micro-b-male-usb-2-a-micro-b_large.jpg)

然后我们需要把编译好的程序下载到我们的芯片里，完成这一步我们需要**stlink**的帮助。stlink是针对stm8和stm32系列芯片的调试器（其他的芯片厂商有他们自己对应的其他调试器）。我们的开发板上已经自带了stlink的硬件，所以硬件上我们不需要配置，但是软件上需要相应的工具。

## 开发过程

基础知识我们已经准备好了，现在我们开始进行实际操作。最开始也提到了，我的开发环境是Linux系统，选择的开发工具和使用的库也都是开源的。这个一方面还是因为（执着的）开源信仰，另一方面也是因为开源的这一套工具链真的很简洁好用，你不需要注册一堆账号，下载一堆非常heavy的工具，而且平台依赖度小，作为开发者也更好的掌握期间到底哪个环节如果出了问题。可能潜在的难点就是需要对操作系统和程序编译运行的原理稍有了解，但是这一定是投资一次收获多次的努力，而且它也真的没有特别难的：）

### 工具

综上所分析，硬件上我准备的有：
- Ubuntu 18.04 系统的计算机
- NUCLEO-F429ZI开发板
- TypeA-MicroB的线

软件上需要的准备有：
- 可能需要预备一些环境
  ```
  sudo apt install build-essential
  sudo apt-get install libusb-1.0.0-dev
  ```
- 交叉编译工具
  ```
  sudo apt-get install gcc-arm-none-eabi gdb-multiarch
  ```
  - 注：似乎Ubuntu 18.04之前的版本需要的是``gdb-arm-none-eabi``，Ubuntu 18.04合并到了``gdb-multiarch``。[1]
- firmware library
  - [LibOpenCM](https://github.com/libopencm3/libopencm3/): 支持相当多的芯片型号（具体见他们的README），STMF4系列也在支持范围内。
  - 安装：
  ```
  git clone https://github.com/libopencm3/libopencm3/
  cd libopencm3
  make
  ```
- [stlink](https://github.com/texane/stlink)
  - 目前支持STLINKv1, v2, v2-1
  - 安装：（需要预先安装cmake，c compiler和libusb，详见[此](https://github.com/texane/stlink/blob/master/doc/compiling.md)）
  ```
  git clone https://github.com/texane/stlink
  cd stlink
  make release
  cd build/Release
  sudo make install
  ```

### 开发

正好libopencm3库附带了一个简单的[使小灯闪烁的实例教程](https://github.com/libopencm3/libopencm3-miniblink)，我就直接拿这个来学习了。

首先把开发板和通过TypeA-MicroB线和计算机连接起来。
然后执行以下命令：
```
git clone https://github.com/libopencm3/libopencm3-miniblink.git
cd libopencm3-miniblink
make
st-flash write bin/stm32/nucleo-f429zi.bin 0x8000000
```
- 0x800 0000是flash memory的开始映射位置，可以从官方文档[stm32f439zi型号的数据表](http://www.st.com/resource/en/datasheet/stm32f429zi.pdf)看到（在第5节memory mapping的图中）。其实所有stm32系列的芯片的flash memory都是从0x800 0000开始的。

{% asset_img memory-mapping.jpg %}

- 最后一步可能出现的错误是Couldn't find any ST-Link/V2 devices，尝试把开发板连接到电脑的USB 2.0的端口，而不是3.0的端口可解决问题。（小tip：可以通过端口的颜色辨认是2.0还是3.0）
![USB 2.0 和 3.0 的区别](https://i.ytimg.com/vi/p7qP_qtaisA/maxresdefault.jpg)

That's it! 这时候你应该可以看到小灯Led2在闪烁啦。

## 总结

这是一篇从程序员的角度出发的，以小灯闪烁的代码为例，侧重讲解如何在Linux上使用开源工具链配置嵌入式开发的环境的一篇入门教程。下一篇我们会进一步分析这篇博客用到的小灯闪烁的代码。

## 引用
[1] https://askubuntu.com/questions/1031103/how-can-i-install-gdb-arm-none-eabi-on-ubuntu-18-04-bionic-beaver

