---
title: Ubuntu安装Postgresql并设置自己的数据文件位置
tags:
date: 2023-12-24 17:31:10
categories: 
- 数据库
- Postgresql
---


## 安装
可以直接从官方`apt`源安装
```bash
sudo sh -c 'echo "deb https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt update
sudo apt install postgresql
```

## 设置数据文件位置
#### 准备工作——创建新位置
创建如下新目录，并且GRANT全部权限，数据文件将保存于此：
```
/home/db/postgre/data
```

#### 步骤一：移动数据文件
切换到默认PostgreSQL用户(一般为`postgres`)
```bash
sudo su postgres
```
进入交互：
```
psql
```

查看当前默认的数据目录位置：
```
postgres=# SHOW data_directory;
```
输出
```
Output
       data_directory
-----------------------------
 /var/lib/postgresql/16/main
(1 row)
```
退出交互：
```
postgres=# \q
```
停止服务：
```bash
sudo systemctl stop postgresql
sudo systemctl status postgresql # 查看状态，确认已停止
```
```
○ postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Sun 2023-12-24 14:41:08 CST; 4s ago
    Process: 183242 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 183242 (code=exited, status=0/SUCCESS)
        CPU: 1ms
```
看到`inactive (dead)`确认已经停止。

拷贝数据库目录到新位置，如果没有rsync，可以使用apt安装下，参数说明：
  - -a 参数保存权限和其他目录属性
  - -v 会显示详细过程

注意：
 - 如果在目录最后加上／，会把目录下面的内容拷贝过去，
 - 如果最后没有／，会拷贝这个目录及下面的内容。

```bash
sudo rsync -av /var/lib/postgresql/ /home/db/postgre/data/
```

上面的拷贝结束后，把当前数据目录改名成备份，等最后确认新位置没有问题后再删除：
```bash
$ sudo mv /var/lib/postgresql/16/main /var/lib/postgresql/16/main.bak
```

#### 步骤二：指向数据文件新位置
编辑配置文件：
```bash
sudo vi /etc/postgresql/16/main/postgresql.conf
```
找到有data_directory的一行，修改后面的目录路径为新位置：
```
data_directory = '/home/db/postgre/data/16/main'
```
保存，退出。

#### 步骤三：重新启动数据库并验证数据文件位置
重新启动服务：
```bash
sudo systemctl start postgresql
```
查看服务状态：
```bash
sudo systemctl status postgresql
```
可以看到输出中有active (exited)字样，说明服务启动成功了：
```
Output
● postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sun 2023-12-24 14:41:18 CST; 44s ago
    Process: 217149 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 217149 (code=exited, status=0/SUCCESS)
        CPU: 1ms
```
看到`active (exited)`确认已经启动。

注：这里说明下`(exited)`。`postgresql.service`只是一个启动的wrapper，实际运行的服务是` postgresql@<version>-main.service`。

#### 步骤四：清理数据
如果上述验证过程没有问题，可以删除原有的数据目录了：
```bash
sudo rm -rf /var/lib/postgresql/16/main.bak
```

## 其他常用设置

#### 允许从非本机访问
编辑`postgresql.conf`，将`listen_addresses = 'localhost'`改成`listen_addresses = '*'`以允许所有ip访问，或者填写所需的ip。
