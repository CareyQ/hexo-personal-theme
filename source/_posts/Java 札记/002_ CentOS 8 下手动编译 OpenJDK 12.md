---
title: CentOS 8 下手动编译 OpenJDK 12
categories: Java 札记
tag:
    - Java
    - Linux
    - JDK
date: 2021-04-20
url: java-002
---

## 前言

最近开始硬啃《深入理解 Java 虚拟机》，书中第一章最后的内容就是【自己编译 JDK】，就想亲自动手试试。

自己水平原因，网上大佬们发的贴子我看不懂，只有自己摸索折腾，踩了很多坑之后我写了这篇文章。

## 软件版本

- 操作系统：CentOS 8.3.2011-x86_64
- openjdk version "11.0.9.1" 2020-11-04 LTS
- gcc (GCC) 8.3.1 20191121 (Red Hat 8.3.1-5)
- g++ (GCC) 8.3.1 20191121 (Red Hat 8.3.1-5)
- autoconf (GNU Autoconf) 2.69
- GNU Make 4.2.1
- freetype：22.1.16
- CUPS：2.2.6

## 配置环境

### Bootstrap JDK

根据原书和 `doc/building.html` 文档中说明，要编译版本号为 N 的 JDK，需要提前准备一个至少版本号为 N-1 的 JDK 环境，这个 JDK 官方称之为 `Bootstrap JDK`。因为 JDK 是由很多个部分构成，一部分用了 C、C++ 编写，一部分用了 Java 编写，所以在编译 JDK 的时候需要一个前置的 JDK。

> 鸡生蛋，蛋生鸡的问题...

```bash
sudo yum install java-11-openjdk-devel
```

### GCC 和 make

```bash
sudo dnf group install "Development Tools"
```

### 依赖

在编译的过程中需要其他第三方库的依赖，可根据书中表 1-1 进行安装，也可参考下面的命令进行安装：

```bash
sudo yum install libXtst-devel libXt-devel libXrender-devel libXrandr-devel libXi-devel

sudo yum install cups-devel

sudo yum install fontconfig-devel

sudo yum install alsa-lib-devel

sudo yum install freetype-devel

sudo yum install libffi-devel

sudo yum install autoconf
```

## 开始编译

### 获取源码

OpenJDK 12 仓库地址：https://hg.openjdk.java.net/jdk/jdk12/
进入后点击左侧 `zip` 即可下载当前版本打包好的源码。


- 路径中最好不要包含空格和中文
- 文件夹名不建议太长，文件夹层次不建议太深
- 通过虚拟机编译的，不要直接在共享文件夹内操作，应该拷贝至虚拟机磁盘后，再进行操作


### 依赖检查

```bash
bash configure # 后面可携带参数
```

  注意 

在 GCC 8.0 后，加入了 `stringop truncation` 的验证警告，出现警告就会导致编译不通过，在这里需要禁止这个策略，再进行编译

```bash
bash configure --disable-warnings-as-errors
```

![](https://cdn.jsdelivr.net/gh/CareyQ/careyqx@master/article/25xndkoidvsw.png)

检查依赖后结果如上，通常此步出现错误，一般是因为某个软件环境（依赖）没有安装，或者是安装了版本不匹配导致的，控制台通常会对此类信息进行打印。

### 执行编译

```bash
make images
```

此过程很是耗时，需耐心等待，电脑可能起飞。

![](https://cdn.jsdelivr.net/gh/CareyQ/careyqx@master/article/4imzbimjjlo0.png)

## 验证成果

编译完成后，当前目录下有一个 `build` 文件夹，里面 `linux-x86_64/jdk` 文件夹就是最终编译结果（不一定是 linux-x86_64），进入该文件夹中的 `bin` 目录，执行 `./java -version` 可得到结果。

```bash
$ ./java -version
openjdk version "12-internal" 2019-03-19
OpenJDK Runtime Environment (fastdebug build 12-internal+0-adhoc.careyq.jdk12)
OpenJDK 64-Bit Server VM (fastdebug build 12-internal+0-adhoc.careyq.jdk12, mixed mode, sharing)
```

之后 `jdk` 文件夹就可以直接用在 IDEA 中使用了。

> 在多次执行 `make` 命令时，应该先使用 `make clean` 和 `make dist-clean` 命令清除目录

