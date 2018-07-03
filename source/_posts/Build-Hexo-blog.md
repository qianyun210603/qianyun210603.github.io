---
title: Hexo静态博客系列之一——在VPS上搭建
date: 2018-07-03 21:02:14
categories:
- Hexo
---

### 本地部分
1. 安装node.js
```bash
curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
apt install nodejs -y
```

2. 安装Hexo
```bash
npm install -g hexo-cli
```

3. 初始化博客：
	1. 使用
        ```bash
        hexo init myblog
        ```
        创建`myblog`目录并在其中生成博客需要的基本文件。
    2. 初始化结束后，安装用于git部署的插件`hexo-deployer-git`和用于预览的本地server`hexo-serverfor`。
        ```bash
        cd myblog
        npm install hexo-deployer-git --save
        npm install hexo-server
        ```

4. 生成RSA密钥对用于与VPS之间的git验证
    ```bash
    ssh-keygen -b 2048 -t rsa
    ```


### 服务器部分

1. 安装Web Server服务，这里用nginx。参照[官网](http://nginx.org/en/linux_packages.html#stable)上的说明。

2. 安装git：`apt install git`

3. 为git单独建立一个账户，在`$HOME`下建立`.ssh`目录并在其中建立`authorized_keys`文件，将之前生成的密钥对中的公钥粘贴到其中：
```bash
adduser git
su git
cd ~
mkdir .ssh
vim .ssh/authorized_keys # paste public key
```
4. 建立git裸仓库（bare repository）并设置一个钩子使得push进入裸仓库的文件自动复制到网站根目录下：
```bash
git init --bare blog.git
vi blog.git/hooks/post-receive
chmod +x blog.git/hooks/post-receive
```
`post-receive`文件的内容为：
```bash
#!/bin/sh
git --work-tree=/var/www/html --git-dir=$HOME/blog.git checkout -f
```

### 返回本地部分

1. 撰写第一篇博客
	1. 在之前生成的myblog目录下运行
	```bash
    hexo new “helloworld”
    ```
    生成新文章模板，位于`/myblog/source/_posts/helloworld.md`。
    2. 在模板中填入内容
    3. 生成HTML
    ```bash
    hexo generate
    ```

2. 预览：`hexo server`
3. 部署：
在`_config.xml`文件中配置部署信息：
```
deploy:
   type: git
   repo: git@<server>:<path to bare repository>
   branch: master
```
之后
```bash
hexo deploy
```
即可。
