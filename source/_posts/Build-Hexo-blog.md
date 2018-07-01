---
title: Build Hexo blog
date: 2018-06-26 21:02:14
tags:
---
Local

### Install Hexo
##### Install node.js
```bash
curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
apt install nodejs -y
```

##### Install Hexo
```bash
npm install -g hexo-cli
```

`cd` to the folder where we want to store the blog, execute 
```bash
hexo init myblog
```
This will create folder `myblog` and generate necessary files in it.
When initialization is done, install two plugins `hexo-deployer-git` and `hexo-serverfor` automatic git deployment and local server for preview, respectively.
```bash
cd myblog
npm install hexo-deployer-git --save
npm install hexo-server
```

Generate RSA key pair
```
ssh-keygen -b 2048 -t rsa
```


Server

Install nginx according the instruction on [official site](http://nginx.org/en/linux_packages.html#stable).



```
adduser git
su git
cd ~
mkdir .ssh
vim .ssh/authorized_keys

```
Copy the public key generated before in to `.ssh/authorized_keys`.

build bare git repository and add hook to automaticlly copy any pushed file to the root folder of the blog site:
```
git init --bare blog.git
vi blog.git/hooks/post-receive
chmod +x blog.git/hooks/post-receive
```

The content of `post-receive` is
```
#!/bin/sh
git --work-tree=/var/www/html --git-dir=$HOME/blog.git checkout -f
```

Back to local:
Setup `_config.xml`:
```
deploy:
   type: git
   repo: git@<server>:<path to bare repository>
   branch: master 
```