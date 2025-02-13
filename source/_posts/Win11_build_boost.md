---
title: Windows 11编译Boost (v1.86.0)库并加入系统路径
date: 2024-12-07T10:33:22+08:00
toc: true
tags: 
categories: 
- Programming
- C++
---

1. 去官网下载源码压缩包并解压；
2. 打开Developer Command Prompt for VS 2022，切换到解压后的目录；
3. 运行以下命令：
   ```cmd
   mkdir installed
   bootstrap.bat
   b2 install variant=release link=shared,static threading=multi address-model=64 --prefix=installed
   ```
   - `installed`为希望安装编译好的boost头文件、库文件的目录，这里是相对路径，也可以是绝对路径
4. 搜索栏中搜索`Environment Variable`，打开Windows 11的环境变量配置，新增`BOOST_ROOT`指向3中的安装路径。

## 参考资料
> - [boost - Invocation](https://www.boost.org/build/doc/html/bbv2/overview/invocation.html)
