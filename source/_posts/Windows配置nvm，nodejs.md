---
title: Windows配置nvm，nodejs
date: 2024-11-02T11:43:04+08:00
toc: true
tags: 
categories: 
- Programming
---

1. 下载`nvm`安装包：[下载地址](https://github.com/coreybutler/nvm-windows/releases)；
2. 运行并按步骤提示完成安装，过程中可以指定安装目录；
3. 修改配置文件：切到`nvm`安装目录，打开`settings.txt`
   ```txt
    root: F:\App\nvm
    path: C:\Program Files\nodejs
    node_mirror: https://npmmirror.com/mirrors/node/
    npm_mirror: https://npmmirror.com/mirrors/npm/
   ```
   - `root`：`nvm`安装的node的存储目录，各版本的`nodejs`会被安装到`root\版本号`下面，要和环境变量`NVM_HOME`一致；
   - `path`：目录链接，当运行`nvm use xxx`时，会将这个链接链接到对应的`root\版本号`上。要和环境变量`NVM_SYMLINK`一致，然后`%NVM_SYMLINK%`要被添加到`Path`里面；
   - `node_mirror`，`npm_mirror`：node和npm的镜像源，如果官网过慢或无法访问，可以设置这两个。

4. 安装`nodejs`:
   ```
   nvm install latest
   ```
   `latest`可以换成指定的版本号。注意如果第3步中的`path`设置的目录需要管理员权限的话，这一步也需要用管理员权限的cmd或Terminal或Powershell运行。
5. 使用`nodejs`
   ```
   nvm use xxx
   ```
   `xxx`为版本号。如果第3步中的`path`设置的目录需要管理员权限的话，这一步会有UAC提示授权。


## 参考资料
> - [nvm Windows 安装](https://nvm.p6p.net/install/windows.html)
> - [Where does npm install packages? Find the install path](https://sebhastian.com/where-does-npm-install-packages/)
> - [下载和安装 Node.js 和 npm](https://nodejs.cn/npm/getting-started/configuring-your-local-environment/downloading-and-installing-node-js-and-npm/)
