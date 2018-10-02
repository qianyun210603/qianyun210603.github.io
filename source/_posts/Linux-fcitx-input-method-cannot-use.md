---
title: Linux Mint某些程序无法使用fcitx的输入法
date: 2018-07-29 12:59:32
categories:
- Linux
---
#### 问题
　　安装了GoldenDict结果发现无法在搜索框里输入中文和日文。
  
#### 解决
　　安装Qt5版本的fcitx前端：
  ```
  sudo apt install fcitx-frontend-qt5
  ```
  
#### 原因
　　Linux下这点比较麻烦，输入法的UI框架有时需要和应用程序的UI框架一致才能启动。一般输入法都会有gtk+ 2/3，Qt 4/5等不同的前端。如果对于某个桌面应用输入法无法使用，需要先确定该应用是基于哪个UI框架的，再检查输入法该UI框架的前端有没有安装。如果没有安装，一般安装后即可解决问题。





