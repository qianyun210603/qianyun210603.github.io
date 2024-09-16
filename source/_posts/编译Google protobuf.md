---
layout: "post"
title: "Windows下编译Google protobuf"
date: "2022-10-15 16:04"
categories:
  - Programming
---

### 准备工作
`Protobuf`依赖`CMake`、`Git`、`Abseil`和`zlib`(可选)。前两者都是常用的工具，略过，这里详述后两者。

##### Abseil
1. 在[官方Github的Release页面](https://github.com/abseil/abseil-cpp/releases)下载最新版源码包。
2. 解压到适当位置，比如`E:\protobuf\abseil`。
3. 打开CMD并进入该目录
   ```bash
   > cd E:\protobuf\abseil
   ```
4. 创建编译工作目录并进入
   ```bash
   > mkdir build && cd build
   ```
5. 创建CMake配置
   ```bash
   > cmake -G "Visual Studio 17 2022" -A x64 -DCMAKE_BUILD_TYPE=Release -B . -DCMAKE_INSTALL_PREFIX=./output_release -DCMAKE_CXX_STANDARD=17 ..
   ```
   其中
   - `-G "Visual Studio 17 2022"`用于指定项目文件的生成器。生成器负责为您选择的构建系统生成构建配置文件。可选有（但不限于）`Unix Makefiles`、 `NMake Makefiles`、 `Visual Studio x yyyy`、`Xcode`等。
   - `-A x64`表示编译64位，32位设为`Win32`
   - `-DCMAKE_BUILD_TYPE=Release`表示编译Release版本，Debug版本就改为Debug（这个似乎要放到靠前位置，否则会有警告表示并没有被用到）
   - `-B .`设置工作目录
   - `-DCMAKE_INSTALL_PREFIX=./output_release`是最终安装的位置
   - `-DCMAKE_CXX_STANDARD=17`表面使用C++17标准
   - `..`为源代码根目录（根`CMakeList.txt`所在目录）
6. 编译安装
   ```bash
   > cmake --build . --target install --config release -j 12
   ```
   其中
   - `--build .`即为前一步的工作目录
   - `--target install`指示CMake将项目的工件（例如，可执行文件、库、头文件）安装到配置项目时指定的安装前缀目录（`-DCMAKE_INSTALL_PREFIX`指定的目录）
   - `--config release`表面编译Release版本
   - `-j 12`表示用12线程并行编译

完成后可以在配置时候指定的`./output_release`找到头文件和编译的库文件。

##### zlib
1. 在[官网](https://www.zlib.net/)下载最新版源码包。
2. 解压到适当位置，并进入该目录
5. 创建CMake配置
   ```bash
   > cmake -G "Visual Studio 17 2022" -A x64 -B . -DCMAKE_INSTALL_PREFIX=./output_release -DCMAKE_CXX_STANDARD=17 ..
   ```
   其中
   - `-G "Visual Studio 17 2022"`用于指定项目文件的生成器。生成器负责为您选择的构建系统生成构建配置文件。可选有（但不限于）`Unix Makefiles`、 `NMake Makefiles`、 `Visual Studio x yyyy`、`Xcode`等。
   - `-A x64`表示编译64位，32位设为`Win32`
   - `-B .`设置工作目录
   - `-DCMAKE_INSTALL_PREFIX=./output_release`是最终安装的位置
   - `-DCMAKE_CXX_STANDARD=17`表面使用C++17标准
   - `..`为源代码根目录（根`CMakeList.txt`所在目录）
6. 编译安装
   ```bash
   > cmake --build . --target install --config release -j 12
   ```
   其中
   - `--build .`即为前一步的工作目录
   - `--target install`指示CMake将项目的工件（例如，可执行文件、库、头文件）安装到配置项目时指定的安装前缀目录（`-DCMAKE_INSTALL_PREFIX`指定的目录）
   - `--config release`表面编译Release版本
   - `-j 12`表示用12线程并行编译

### 编译安装
1. 在[官方Github的Release页面](https://github.com/protocolbuffers/protobuf/releases)下载最新版源码包。
2. 解压到适当位置，比如`E:\protobuf\protobuf`。
3. 切换到该目录
4. 创建CMake配置
   ```bash
   > cmake -S . -B cmake-out -DCMAKE_INSTALL_PREFIX=./output -DCMAKE_CXX_STANDARD=17 -Dprotobuf_BUILD_TESTS=OFF -DCMAKE_BUILD_TYPE=Release -Dprotobuf_ABSL_PROVIDER=package -Dprotobuf_MSVC_STATIC_RUNTIME=OFF -DCMAKE_PREFIX_PATH=E:\protobuf\abseil\build\output_release -DZLIB_ROOT=E:\protobuf\zlib\build\output_release
   ```
   其中：
   - `-S ."`指定源码解压的位置（根`CMakeList.txt`所在目录），这里是当前文件夹
   - `-B cmake-out`设置工作目录
   - `-DCMAKE_INSTALL_PREFIX=./output`是最终安装的位置
   - `-DCMAKE_CXX_STANDARD=17`表面使用C++17标准
   - `-Dprotobuf_BUILD_TESTS=OFF`关掉GTest
   - `-Dprotobuf_ABSL_PROVIDER=package`指定protobuf如何链接Abseil，`module`是Abseil已经包含了Abseil，不需要格外提供Abseil在库目录里面；`package`表示需要提供Abseil在库目录里面
   - `-Dprotobuf_MSVC_STATIC_RUNTIME=OFF`表示动态链接C++ runtime library (CRT)，设为`ON`则是静态链接。因为Abseil官方只提供了动态链接方式（不改CMakeList.txt的情况下），所以这里也只能用OFF，否则报链接错误
   - `-DCMAKE_PREFIX_PATH=..\abseil\build\output_release`准备工作中`Abseil`的安装目录
   - `-DZLIB_ROOT=E:\protobuf\zlib\build\output_release`准备工作中`zlib`的安装目录

5. 编译安装
   ```bash
   cmake --build cmake-out --config Release --target install
   ```
   这个选项含义同上面`Abseil`