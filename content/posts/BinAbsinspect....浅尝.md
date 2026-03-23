---
date : '2025-01-12T14:41:31+11:00'
title : 'BinAbsinspector、bap等浅尝'
---
ghidra、BinAbsinspector、bapc ew_checker and radare

以上为应实习工作内容便利，笔者最近尝试了几款主流二进制工具，记录过程以及实战对比

### 安装ghidra以及BinAbsinspector环境搭建

笔者最近了解了科恩实验室在去年推出的静态二进制分析工具BinAbsinspector，这篇文章记录下环境搭建过程

首先ghidra是开源的，所以组里想让我用下看效果怎么样以及之后是否有API方便接入，但是先总结：ghidra这玩意不管是win还是linux好像只有gui，BinAbsinspector是有提供命令行

![image-20231222170409463](C:/images/image-20231222170409463.png)

接下来是ubuntu配置过程，win差不太多，后面笔者会说明

看下官方文档

![image-20231222141026854](C:/Users/19450/Desktop/resume/some-project/ticpsh111/diary/*C:/Users/19450/Desktop/diary_now/image-20231222141026854.png)

### 首先下载ghidra

[Releases · NationalSecurityAgency/ghidra · GitHub](https://github.com/NationalSecurityAgency/ghidra/releases)

直接下载安装压缩包

但是最好下载10.1.2版本（后面会解释）

### 安装z3

根据官方的建议，我们去[Releases · Z3Prover/z3 (github.com)](https://github.com/Z3Prover/z3/releases)下载对用的zip压缩文件

（这里我没有找到amd64架构的，不过x86差别不大也能用）

下载压缩后我们把动态链接文件全都cpoy到lib文件夹中

```python
`z3-${version}-glibc-${version}/bin/*.so``/usr/local/lib/`
```

这里我的：

```python
(test-ghidra) heshang@ubuntu:~/desktop$ sudo cp ./z3-4.12.4-x64-glibc-2.31/bin/*.so /usr/local/lib/
```

但是这个方法没有用，使用BinAbs时会提醒报错

我们使用make构建z3

们到z3的github官方找到链接

![image-20231222155911794](/images/image-20231222155911794.png)

```python
git clone https://github.com/Z3Prover/z3.git
cd z3
sudo python scripts/mk_make.py
cd build
sudo make

sudo make install
```

下载BinAbsinsprctor

![image-20231222150858658](/images/image-20231222150858658.png)

官方只提供了一个版本，注意最好使用ghidra_10.1.2，因为不管是压缩包名字就是10.1.2，而且我用别的版本操作下来不行

明明z3已经好了，无语

![image-20231222165335320](/images/image-20231222165335320.png)

### BinABS插件导入

![image-20231222171149739](/images/image-20231222171149739.png)

由上到下我们分别new Project创建工程

install extension导入插件，把BinAbs前面打对号

![image-20231222171252671](/images/image-20231222171252671.png)

import file导入二进制文件

双击文件进入分析之后

![image-20231222171500498](/images/image-20231222171500498.png)

在Script Manager中双击我们的插件便可以分析了

![image-20231222171539315](/images/image-20231222171539315.png)

但是ubuntu因为z3有问题，所以报错

![image-20231222165221982](/images/image-20231222165221982.png)

笔者累了，关于ubuntu使用宣布失败

我们尝试windows

主要区别就是z3的环境配置上，直接把z3的/bin目录添加到系统环境变量中

![image-20231222171718957](/images/image-20231222171718957.png)



也就是z3里面/bin文件夹里面的pyhon路径

![image-20231222171946946](/images/image-20231222171946946.png)

按照上述的操作我们可以看到成功输出两个危险可能

我们的pwn程序是一个整数溢出

![image-20231222172058010](/images/image-20231222172058010.png)

![image-20231222172040483](/images/image-20231222172040483.png)

所以BInAbs提示gets溢出危险函数以及cat flag危险字符串



### BAP

卡内基梅隆大学二进制分析平台

![image-20231225104522498](/images/image-20231225104522498.png)

我们是ubuntu，下载tgz压缩包

![image-20231225104644693](/images/image-20231225104644693.png)

这个工具的使用是基于我之前的test-angr环境（免得出什么问题）

### 安装BAP [python](https://so.csdn.net/so/search?q=python&spm=1001.2101.3001.7020) bindings



安装OPAM

```
sudo apt-get install opam
```

安装BAP

opam init --comp=4.03    # install the compiler
opam repo add bap git://github.com/BinaryAnalysisPlatform/opam-repository
eval `opam config env`               # activate opam environment
opam depext --install bap            # install bap

![image-20231225140845638](/images/image-20231225140845638.png)

网上几乎找不到复现，自己根据gitub以及help运行有问题

放弃

### cwe_checker

也是基于ghidra

利用Ghidra反汇编出的PCode（和BinAbsinspector一样）

我们直接用提供的docker

docker build -t cwe_checker 

```python
docker run --rm -v /home/heshang/desktop/instantcam:/instantcam  fkiecad/cwe_checker /instantcam
```

分析我们的一个即时摄像头控制器二进制文件

![image-20231226095031402](/images/image-20231226095031402.png)

这个产出对比BinAbsInspector好很多看起来

下面是BinAbsinspector的结果

![image-20231226095453185](/images/image-20231226095453185.png)



### radare2



```text
 git clone https://github.com/radareorg/radare2.git
$ cd radare2
$ sudo ./sys/user.sh
```

这个要多试几次，我当时就是好几次才成功（可能会网络问题连接失败）

![image-20231225163857732](/images/image-20231225163857732.png)

完成，这里告诉我们被下载到了/root/bin

![image-20231225163752368](/images/image-20231225163752368.png)

我们看下版本号，成功

分析我们的二进制文件

![image-20231225164730939](/images/image-20231225164730939.png)

但是我们修改path还是不能直接用r2

![image-20231225165428130](/images/image-20231225165428130.png)

系统会提示apt install radare2，之后就可以正常操作了

![image-20231225165343444](/images/image-20231225165343444.png)

radare2下可以安装插件r2ghidra，你没看错，就是之前我们用过的ghidra，我在想r2ghidra中还能不能安装插件，这样我就可以把BinAbsInspector也放进去

![image-20231226155345493](/images/image-20231226155345493.png)

我们根据官网安装

```
r2pm update
r2pm -ci r2ghidra
```

但是网络问题很难受，但是不断尝试后可以跑出来

![image-20231225174653071](/images/image-20231225174653071.png)

报错，下面显示使用默认g++会接受，配置c++环境，还是不行

![image-20231225181812691](/images/image-20231225181812691.png)

![image-20231225181803844](/images/image-20231225181803844.png)

github上有一个报错相同的，但是没有解决方法

![image-20231225174346816](/images/image-20231225174346816.png)





![image-20231226160542411](/images/image-20231226160542411.png)

还是不行

修改源码后没有用，因为是github远程代码，并不能修改

我们换kali

安装radare2

![image-20231227095957766](/images/image-20231227095957766.png)

下载我们的插件r2ghidra

![image-20231227102721610](/images/image-20231227102721610.png)

报错

![image-20231227102653643](/images/image-20231227102653643.png)

makefile文件的问题好像，我们换deb包

![image-20231227112037473](/images/image-20231227112037473.png)

安装工具gdebi

```shell
sudo apt-get install gdebi-core
```

安装r2ghidra

```shell
sudo gdebi r2ghidra_5.8.8_amd64.deb 
```

我们进入radare2，输入pdg调用我们的r2

![image-20231227110310457](/images/image-20231227110310457.png)

可以看到成功

![image-20231227110848728](/images/image-20231227110848728.png)

但是官网没有找到如何在r2ghidra上添加插件（感觉应该是不行的，可能需要后续开发c++）







![image-20231226160806834](/images/image-20231226160806834.png)

![image-20231226163249680](/images/image-20231226163249680.png)



### 工具对比

#### BinAbsInspector

可移植性：需要借助ghidra的Pcode，而ghidra是只有gui，所以移植性差

用户友好：执行分析插件，反馈可能安全问题cwe

复现：linux没有成功，windows下可以，docker

![image-20231226112342263](/images/image-20231226112342263.png)

![image-20231226110012566](/images/image-20231226110012566.png)

#### cwe_checker

可移植性：需要借助ghidra的Pcode，不需要借助ghidra生成的工程文件，所以支持纯命令行

用户友好：执行分析插件，反馈可能安全问题cwe，实际使用（instancam）下来比BinAbsInspector好

复现：只复现linux下docker

![image-20231226112358689](/images/image-20231226112358689.png)

![image-20231226105222488](/images/image-20231226105222488.png)



#### radare2

是一个“类 Unix 系统上的逆向工程框架和命令行工具集”，我的理解是类似于加强后的gdb

可移植性：应该无

用户友好：适合有基础的人分析

复现：可以使用

![image-20231226105953052](/images/image-20231226105953052.png)

##### 关于radare2下的一个插件ghidra

我在想能不能在插件ghidra中安装插件BinAbsInspector，这样就可以纯命令行分析文件cwe

radare2下可以安装插件ghidra，但是github仓库源码问题，复现失败

![image-20231226105942038](/images/image-20231226105942038.png)

#### bap

网上资料少，成功下载但是命令行有问题搞不懂

![image-20231226105823465](/images/image-20231226105823465.png)

关于BInAbs  docker错误

![image-20231226131814882](/images/image-20231226131814882.png)

### HCML_S32K312_BSW_GHS_Project_V5(1).elf

#### cew_checker

![image-20240102145514530](/images/image-20240102145514530.png)

#### BinAbsInspector

![image-20240102145523000](/images/image-20240102145523000.png)

这个函数没有具体入口，所以工具无法分析

### instantcam

#### cew_checker

![image-20240102145701771](/images/image-20240102145701771.png)

cwe比较多，包括了下面BinAbs分析出来的结果

#### BinAbsInspector

![image-20240102145651440](/images/image-20240102145651440.png)

