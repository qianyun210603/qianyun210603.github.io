---
title: Windows本地编译pandas源码
date: 2024-10-26T18:03:02+08:00
toc: true
tags: 
categories: 
- Programming
- Python
---

1. 从pandas的GitHub上克隆源码；
2. 使用源码目录里面的`environment.yml`建立一个新env；（否则会破坏现有环境，因为pandas编译要设很多临时环境变量）;
    ```cmd
    conda env create --file environment.yml
    ```

    这会自动创建名为`pandas-dev`的环境并安装所需依赖，但还差一个`meson`，切换到`pandas-dev`，使用如下命令安装：
    ```cmd
    conda activate pandas-dev
    conda install meson-python
    ```
3. 切换到克隆下来的源码目录，运行
    ```cmd
    # Build and install pandas
    # By default, this will print verbose output
    # showing the "rebuild" taking place on import (see section below for explanation)
    # If you do not want to see this, omit everything after --no-build-isolation
    python -m pip install -ve . --no-build-isolation -Ceditable-verbose=true
    ```
    这一步踩了一些坑，这里说下解决方法：
    - 找不到`cl.exe`。
      这是因为Visual Studio或者Build Tools for Visual Studio 2022的安装目录没有被加入系统目录。找到安装目录，然后导引到`VC\Tools\MSVC\14.xxxx\bin\Hostx86\x86\`，`cl.exe`就在这个目录里面，将其加入系统的`Path`环境变量。其中14.xxxx是版本号，`Hostx86\x86\`也可以是`Hostx64\x64\`，看想用32位编译器还是64位编译器。
    - 找不到`MSVCRT.lib`（或其他系统`lib`文件）。这里要检查第2步中`activate`的输出，一般是未能找到所需的MSVC编译器版本，这时第2步里面设置环境变量就会显示找不到`vcvars64.bat`。这里面需要注意pandas需要用VS2019的Build Toolset (v142)，所以如果之前没有安装需要用Visual Studio Installer补上。
     ![alt text](1.png)
     注意一定要用`Desktop development with C++`里面的`MSVC v142 - VS2019 C++ x64/x86 build tools`。原因是`pandas`给的conda脚本是根据VS的Component ID找VS安装目录的。只有这个Build tools符合脚本要求。如果安装的是`Individual components`里面的v142，那还是找不到，ID不一样。

## 参考资料
> - [Creating a development environment](https://pandas.pydata.org/docs/development/contributing_environment.html)
> - [Visual Studio Build Tools component directory](https://learn.microsoft.com/en-us/visualstudio/install/workload-component-id-vs-build-tools?view=vs-2022)
