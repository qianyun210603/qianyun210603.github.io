---
layout: "post"
title: "Ubuntu系统挂载Windows共享文件夹通过credential文件认证"
date: "2022-11-29 15:55"
categories:
 - Linux
---

### 问题

要把Windows的共享文件夹挂载到Ubuntu系统的电脑里面，使用命令
```
sudo mount -t cifs //<ip or host>/<source folder> <destiny folder> -o username='xxx'',password='xxx',vers=2.0
```
可以，但是使用密码文件
```
sudo mount -t cifs //<ip or host>/<source folder> <destiny folder> -o credentials=/xxx/.smbcredentials,vers=2.0
```
报如下错误
```
mount: <destiny folder>: bad option; for several filesystems (e.g. nfs, cifs) you might need a /sbin/mount.<type> helper program.
```

### 解决
安装`cifs-util`包。
```
sudo apt install cifs-util
```

安装后重新挂载，成功~
