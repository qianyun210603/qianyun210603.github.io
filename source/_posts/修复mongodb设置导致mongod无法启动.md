---
layout: "post"
title: "修复MongoDB设置导致mongod无法启动"
date: "2022-10-15 16:04"
categories:
  - Programming
  - MongoDB
---

## 缘起

为了解决使用Arctic[^fn1] MongoDB的日志里出现

```
SASL SCRAM-SHA-1 authentication failed for myAdminUser on admin from client 192.168.3.100:9560 ; UserNotFound: Could not find user "myAdminUser" for db "admin"
```
错误的问题，参考了数个连接（包括StackOverflow[^fn2]），都提示要将`authSchema.currentVersion`从`5`改成`3`。照做以后就杯具了，`mongod`服务完全打不开了，提示错误
```
This server is using MONGODB-CR, an authentication mechanism which has been removed from MongoDB 4.0. In order to upgrade the auth schema, first downgrade MongoDB binaries to version 3.6 and then run the authSchemaUpgrade command. See http://dochub.mongodb.org/core/3.0-upgrade-to-scram-sha-1
```
原因是我的MongoDB server是4.2版本，`authSchema.currentVersion=3`代表MONGODB-CR的认证方式，4.0以上的Mongo Server已经不再支持了。

<font color=red>
*感觉MongoDB这种设计简直愚蠢反人类到家了，这种能导致启动崩溃的设置居然是写在数据库里而不是独立的配置文件里，直接导致万一设错了服务就完全起不来，服务器起不来又无法更正配置，直接死循环了。*

*所以，尽量不要更改数据库里的配置，一定需要改的话，改之前一定要备份。*
</font>
## 解决

#### 失败尝试一

按照日志里的提示，将MongoDB服务的版本降低到3.6，同样无法启动，提示WeirdTiger格式不兼容。顿时感觉天昏地暗，前一个备份已经两周多了，完全不想重新恢复这两周的所有数据。

#### 灵机一动的尝试二

想到我按照的修改既然是存在数据库里的，那么必然改了某个数据库的磁盘文件。那么这个文件的修改日期一定是所有数据库数据文件中最新的。于是赶紧`cd`去数据库文件在磁盘里的目录，`ls -lnt`找到最后修改的文件是这个`collection-0-5610079802494816593.wt`。

用vim打卡，`:%!xxd`切换到二进制模式，搜索`authSchema`，果然找到了
```
00005070: 6400 0b00 0000 6175 7468 5363 6865 6d61  d.....authSchema
00005080: 0010 6375 7272 656e 7456 6572 7369 6f6e  ..currentVersion
00005090: 0003 0000 0000 0000 0000 0000 0000 0000  ................
```
赶紧把`currentVersion`后边的0003改成0005。再次重启`mongod`。发现日志还是报错

```
wiredTiger Error: collection-*.wt does not appear to be a WiredTiger file
```
并提示可以加`--repair`参数尝试修复，于是执行
```bash
mongod --noauth --dbpath /mongodb/data/ --repair
```
成功结束后再次启动mongod服务
```
systemctl start mongod
```

成功~~

[^fn1]: https://github.com/man-group/arctic
[^fn2]: https://stackoverflow.com/questions/29006887/mongodb-cr-authentication-failed
