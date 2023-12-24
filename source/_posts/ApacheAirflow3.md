---
title: 单个非root用户Anaconda下安装Apache Airflow用于定时任务：使用Postgresql
date: 2023-07-15T15:30:21+08:00
toc: true
tags: 
categories: 
- 应用软件
- Airflow
---

## 准备工作
首先参考：
- 《在Anaconda环境下安装Apache Airflow以进行定时任务管理：安装篇》安装`airflow`
- 《Ubuntu安装Postgresql并设置自己的数据文件位置》配置好数据库
- 安装`SQLAlchemy`和`psycopg2`
   ```
   pip install SQLAlchemy
   pip install psycopg2
   ```
- 停止Airflow相关服务
    ```bash
    sudo systemctl stop airflow-scheduler.service
    sudo systemctl stop airflow-webserver.service
    ```

## 创建Airflow相关的数据库和数据库用户
- 切换到Postgresql的管理用户，运行`psql`进入Postgresql的命令行
    ```
    sudo su postgres
    psql
    ```
- 在Postgresql的命令行运行以下命令（注意最后的分号要带上）创建相应的DB和用户。
    ```sql
    CREATE DATABASE airflow_db;
    CREATE USER airflow_user WITH PASSWORD 'airflow_pass';
    GRANT ALL PRIVILEGES ON DATABASE airflow_db TO airflow_user;
    -- PostgreSQL 15+ requires additional privileges:
    \connect airflow_db;
    GRANT ALL ON SCHEMA public TO airflow_user;
    ```
- 编辑Postgresql的`pg_hba.conf`文件赋予`airflow_user`访问权限
    - 默认位于`/etc/postgresql/16/main/pg_hba.conf`
    - 添加一行
      ```
      host    all             all             127.0.0.1/32            password
      ```
      允许所有本地用户通过`localhost`访问。这个限制有些宽，可以具体限制到用户访问ip等。


## 修改Airflow配置并迁移到新数据库
首先配置环境变量：
```bash
export AIRFLOW_HOME=/data/airflow
export AIRFLOW_CONFIG=/data/airflow/airflow.cfg
```
(注意这里一定要手工`export`，试验过`source /etc/sysconfig/airflow`，尽管`echo $AIRFLOW_HOME`输出的是正确的目录，还是不能被`airflow`识别，不知道是不是因为用的Anaconda环境。)

打开`airflow.cfg`文件
```
sudo vi $AIRFLOW_CONFIG
```
将`sql_alchemy_conn`设为`Postgresql`的链接
```
sql_alchemy_conn = postgresql+psycopg2://airflow_user:airflow_pass@localhost/airflow_db
```
初始化
```bash
airflow db migrate
```
重新启Airflow的服务
```bash
sudo systemctl start airflow-scheduler.service
sudo systemctl start airflow-webserver.service
```
检查状态
```bash
sudo systemctl status airflow-scheduler.service
sudo systemctl status airflow-webserver.service
```
