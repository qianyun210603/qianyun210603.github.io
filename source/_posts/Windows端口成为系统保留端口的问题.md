---
title: Windows端口成为系统保留端口的问题
date: 2024-10-20T21:15:20+08:00
toc: true
tags: 
categories: 
- 其他
---

## 问题背景
因为要使用Redis，所以装了WSL，但启动后发现SS打不开了，提示xxx端口被系统保留。

参考文后所附文章：
- Windows 中有一个「TCP 动态端口范围」，处在这个范围内的端口，有时候会被一些服务占用。在 Windows Vista（或 Windows Server 2008）之前，动态端口范围是 1025 到 5000；在 Windows Vista（或 Windows Server 2008）之后，新的默认起始端口为 49152，新的默认结束端口为 65535。
- 如果安装了 Hyper-V，那么 Hyper-V 会为容器宿主网络服务（Windows Container Host Networking Service）随机保留一些端口号使用。

## 验证问题
##### 查看当前被占用端口
```cmd
netstat -aon | findstr <端口号>
```
返回空，说明当前并没有程序占用目标端口。

##### 查看当前被Window保留的端口
```cmd
netsh int ipv4 show excludedportrange protocol=tcp
```
返回
```
Protocol tcp Port Exclusion Ranges

Start Port    End Port
----------    --------
     xxx1       xxx2     *
     xxx3       xxx4

* - Administered port exclusions.
```
发现目标端口确实在被保留的范围内。这是因为使用WSL需要Hyper-V服务导致的。

###### 设置系统可保留的端口范围，避开常用端口
```cmd
netsh int ipv4 set dynamic tcp start=xxxxx num=yyyyy
netsh int ipv6 set dynamic tcp start=xxxxx num=yyyyy
```
这两个命令分别设定了ipv4和v6系统可保留的端口范围：从xxxxx开始的yyyyy个。

然后还需要重启`winnat`以释放原保留端口。
```cmd
net stop winnat
net start winnat
```

## 参考资料
> - [解决 Windows 10 端口被 Hyper-V 随机保留（占用）的问题](https://zhuanlan.zhihu.com/p/474392069)