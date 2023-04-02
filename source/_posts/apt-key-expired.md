---
title: apt提示key expired问题
date: 2018-07-02 12:02:12
categories: Linux
---
首先找出过期的key:
```bash
$ apt-key list | grep expired
```
其中<font color=#0000ff>BE1DB1F1</font>就是key ID。

找到ID后即可用
```bash
sudo apt-key adv --recv-keys --keyserver keys.gnupg.net BE1DB1F1
```
更新key。
##### Reference
https://serverfault.com/questions/7145/what-should-i-do-when-i-got-the-keyexpired-error-message-after-an-apt-get-update/718435#718435
