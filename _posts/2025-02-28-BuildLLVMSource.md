---
title: llvm源码ubuntu+cmake编译
date: 2025-02-28
tags:
  - Git
  - "#llvm"
categories: Compilers
description: LLVM的源码在Ubuntu系统的编译和一些踩坑记录，内存和核心很重要，WSL不一定够用；耗时很长；
---

[文章CSDN链接-LLVM源码编译](https://blog.csdn.net/Topsort/article/details/145928049?fromshare=blogdetail&sharetype=blogdetail&sharerId=145928049&sharerefer=PC&sharesource=Topsort&sharefrom=from_link)

# 源码编译

之前找到的教程大部分是预编译的代码解压安装
源码编译参考文献：
- [build llvm from source code](https://getting-started-with-llvm-core-libraries-zh-cn.readthedocs.io/zh-cn/latest/ch01.html#id6)
- [LLVM系列第一章：编译LLVM源码_llvm源码编译-CSDN博客](https://blog.csdn.net/Zhanglin_Wu/article/details/124942823)
- [使用 CMake 构建 LLVM — LLVM 21.0.0git 文档 --- Building LLVM with CMake — LLVM 21.0.0git documentation](https://llvm.org/docs/CMake.html)
- [WSL2 Ubuntu22.04编译安装LLVM_wsl llvm-CSDN博客](https://blog.csdn.net/qq_41596730/article/details/143191877)

> 源码的位置不要找错了，是这个仓库

源码所属仓库 [llvm/llvm-project.](https://github.com/llvm/llvm-project/tree/main)

## 准备环境

Ubuntu
最好可以准备ninja

```bash
sudo apt-get install build-essential zlib1g-dev python
```

### 仅为自己安装

笔者最终使用的是服务器集群进行编译，没有root权限，所以只能安装到自己的用户目录`~`

```bash
apt download <package>
# 会下载一个deb文件

dpkg -x <package>.deb ~/path/to/install

vim ~/.bashrc
```

进入vim编辑

```bash
export PATH = $PATH:~/path/to/install/usr/bin # 这种安装会在指定目录下创建usr/bin目录
```

编辑后刷新环境变量
```bash
source ~/.bashrc
```

## 下载源码
真正的源码是`llvm-project`仓库的内容 -- [llvm/llvm-project.](https://github.com/llvm/llvm-project/tree/main)

直接克隆github的太慢且多次断联

尝试了缓冲区、超时时间、禁用HTTP/2、使用代理、浅层克隆、单分支克隆均无果

选择**国内镜像**进行克隆

```bash
git clone https://mirrors.tuna.tsinghua.edu.cn/git/llvm-project.git --depth 1 --branch main

git fetch
```

如果担心下载的内容不够，可以不使用浅克隆与单分支

> 时间会比较长

```bash
git clone https://mirrors.tuna.tsinghua.edu.cn/git/llvm-project.git
```

## cmake构建

```bash
cd llvm-project
mkdir build 
cd build

cmake -G Ninja -DCMAKE_BUILD_TYPE=Debug -DCMAKE_INSTALL_PREFIX="/mnt/d/Work/llvm/llvm-install" -DLLVM_ENABLE_PROJECTS=clang ../llvm
```

遇到报错

```bash
CMake Error at cmake/modules/GetHostTriple.cmake:54 (message):
  Failed to execute /mnt/d/Work/llvm/llvm-project/llvm/cmake/config.guess
Call Stack (most recent call first):
  cmake/config-ix.cmake:483 (get_host_triple)
  CMakeLists.txt:987 (include)
```

~~查找类似错误 [Windows 用 CMake 构建 LLVM 时出现 cmake/config-ix.cmake：401 （get_host_triple） 错误 - Stack Overflow ](https://stackoverflow.com/questions/66087509/cmake-config-ix-cmake401-get-host-triple-error-when-trying-to-build-llvm-with)~~

~~修改后的构建命令

```bash
# 命令无用
cmake -G Ninja -DCMAKE_BUILD_TYPE=Debug -DCMAKE_INSTALL_PREFIX="/mnt/d/Work/llvm/llvm-install" -DLLVM_ENABLE_PROJECTS=clang -DLLVM_HOST_TRIPLE=x86_64 ../llvm
```
~~
该方案不起效

第二篇参考[Ubuntu20.04编译OLLVM踩坑_llvm config.guess-CSDN博客](https://blog.csdn.net/weixin_44700621/article/details/122658900)
主要的原因是在config.guess文件中存在一些DOS行结尾(DOS line endings)，需要将其转换为Unix行结尾。

> 笔者出错猜测是因为没有在WSL中配置git，所以是在主机Windows进行git拉取，可能导致了一些编码问题

```bash
sudo apt-get install dos2unix
dos2unix path/to/config.guess
```

构建成功 `Configure Done`

## 编译

```bash
ninja

cmake --build .
```

都遇到了WSL直接崩溃死机，退出到windows的情况

```bash
free -h # WSL 分配了8GB内存， Swap分区 2GB

nproc # 能看到有16个核心
```

> 一开始选用WSL进行编译，分配的内存8GB，SWAP分区2GB，CPU16核心，
> 一进行编译就WSL崩溃
> 服务器配置是2048GB内存，192核心

转移到服务器上继续运行
编译用时12小时以上

## 安装与Path

```bash
cmake --build . --target install

# 或者
ninja install
```

安装的目录是前面设置的`Prefix_Path`
安装好后，该目录下有`bin`等四个文件夹

```bash
vim ~/.bashrc
```

修改文件

```bash
export PATH = $PATH:/path/to/llvm/install/bin
```

然后刷新环境变量

```bash
source ~/.bashrc
```

验证命令`llvm-as --version`

```bash
(base) root@panda:~$ llvm-as --version
Ubuntu LLVM version 14.0.0
  
  Optimized build.
  Default target: x86_64-pc-linux-gnu
  Host CPU: ivybridge
```

# 错误源码
一开始找到的源码是不对的，实际上应该不是源码，而是预编译的代码，用于安装的

##  ~~下载源码-out~~

[LLVM18.src.tar.xz](https://github.com/llvm/llvm-project/releases/download/llvmorg-18.1.8/llvm-18.1.8.src.tar.xz)
[clang18.src.tar.xz](https://github.com/llvm/llvm-project/releases/download/llvmorg-18.1.8/clang-18.1.8.src.tar.xz)

tar.gz和tar.xz的解压命令不一致

```bash
tar -xJvf *.tar.xz

tar -xzvf *.tar.gz
```

解压后，组织一下目录

```json
llvm
|--	llvm-src
	|-- tools
	    |-- clang 
|-- llvm-build
|-- llvm-install
```

准备编译

```bash
cd llvm-build
cmake /mnt/d/Work/llvm/llvm-src -G Ninja -DCMAKE_BUILD_TYPE="Debug" -DCMAKE_INSTALL_PREFIX="../llvm-install"
```
> 该源码有问题，缺少很多`.cmake`文件，应该不是这一份源码
