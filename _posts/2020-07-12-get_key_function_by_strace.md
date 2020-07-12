---
layout: post
title:  "获取业务关键函数之STRACE"
date:   2020-07-12 20:48:06 +0800
tags: 开发
---

## 概述与使用场景

在研究开源代码或接手项目时，可能面临一些无文档的一些代码，先要快速上手，如何破解? 通过

> STRACE工具

快速了解调用堆栈以及关键业务函数，是一种低成本的方式。

方法大致流程如下：

- 编译strace工具
- 运行应用 
- 分析日志

## 使用步骤

### 编译依赖库和工具

{% highlight shell %}

sudo yum install -y autoconf automake libunwind libunwind-devel gcc

git clone https://github.com/strace/strace.git

cd strace
./bootstrap
./configure --with-libunwind
make
sudo make install

{% endhighlight %}

### 运行用例

以pg为例，通过以下命令启动:

{% highlight shell %}

strace -k -o STRACE -ff -t $PG_CTL start -D $DATA

{% endhighlight %}

NOTE: 注意`STRACE`表示日志前缀

NOTE: `-ff`表示追踪完整堆栈

### 结果分析

{% highlight shell %}

02:50:26 pread64(18, "\0\0\0"..., 8192, 73728) = 8192
 > /usr/lib64/libpthread-2.17.so(__pread_nocancel+0xa) [0xef63]
 > /usr/local/pgsql/bin/postgres(FileRead+0x94) [0x31b374]
 > /usr/local/pgsql/bin/postgres(mdread+0x40) [0x33d330]
 
 ... ... 
 
 > /usr/local/pgsql/bin/postgres(main+0x443) [0x7f003]
 > /usr/lib64/libc-2.17.so(__libc_start_main+0xf5) [0x22505]
 > /usr/local/pgsql/bin/postgres(_start+0x29) [0x7f06a]
{% endhighlight %}

在启动命令运行的目录下，会生成以STRACE为前缀，包含PID的文件，文件内容如上，通过以上日志的跟踪和分析，有较大的概率找到关键入口。

## 存在局限

- 要求所关注业务涉及系统调用，如：IO、网络、线程等

- 二进制编译过程没有进行过 `strip`, 即：函数以及符号表没有被压缩
