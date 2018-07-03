---
title: 重命名Visual Studio项目
date: 2018-07-02 21:02:08
categories:
- Programming
- Visual Studio
---

因为Visual Stuido的主文件名和项目名以及一些依赖文件都是相互关联的，因此单单简单粗暴地重命名项目和源代码会导致编译错误。以下给出一个完美重命名VS项目的步骤检查表：

1. 在Visual Studio重命名项目；
2. 重命名相关的源文件和头文件，并更正`#include`；
3. 重命名`.sln`, .`vcpoj`和`.vcxpoj`文件；
4. 将3中提到的三个文件（均为文本文件，可以用文本编辑器打开）中的旧名称替换为新名称。