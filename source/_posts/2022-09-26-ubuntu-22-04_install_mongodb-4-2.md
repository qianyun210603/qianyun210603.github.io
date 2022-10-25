---
title: "Ubuntu 22.04安装MongoDB 4.2"
date: "2022-09-26 09:52"
tags:
categories:
- Linux
---
问题在于Ubuntu 22.04开始移除了libssl1.1的支持。所以需要先安装libssl1.1

```bash
wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.0g-2ubuntu4_amd64.deb
sudo dpkg -i libssl1.1_1.1.
```

然后将18.04 boinic的source加入source list
```bash
wget -qO - https://www.mongodb.org/static/pgp/server-4.2.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.2.list
```

更新apt并安装
```bash
apt update
apt install -y mongodb-org
```

##### 参考
1. <https://askubuntu.com/questions/1403619/mongodb-install-fails-on-ubuntu-22-04-depends-on-libssl1-1-but-it-is-not-insta>
2. <https://www.mongodb.com/community/forums/t/installing-mongodb-over-ubuntu-22-04/159931/39>
