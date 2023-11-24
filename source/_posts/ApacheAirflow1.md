---
title: 在Anaconda环境下安装Apache Airflow以进行定时任务管理：安装篇
date: 2023-07-15T15:30:21+08:00
toc: true
tags:
categories:
- 应用软件
- Airflow
---

本文将介绍如何在Anaconda环境下安装和配置Apache Airflow，用于管理和调度定时任务。以下是主要步骤：

1. 在Anaconda的base环境或者自定义的环境下，通过pip安装`typing_extensions`和`apache-airflow`包。
    ```bash
    pip install typing_extensions
    pip install apache-airflow
    ```

2. 创建一个目录来存放Airflow的数据，并将这个目录的路径设置为环境变量`AIRFLOW_HOME`。
    ```bash
    mkdir <where you want to put airflow data in>
    export AIRFLOW_HOME=<where you want to put airflow data in>
    ```

3. 运行`airflow scheduler`命令，然后退出。这将在`$AIRFLOW_HOME`目录下创建基础的文件。
    ```bash
    airflow scheduler
    ```

4. 运行`airflow db migrate`命令来初始化数据库，然后创建一个用户。
    ```bash
    airflow db migrate
    airflow users create --username airflow --role Admin -f xxx -l xxx -e xxx@xxx.com
    ```

5. 运行`airflow scheduler`和`airflow webserver`命令来分别启动定时器后台和网页服务。你可以选择使用`nohup`命令来使这些服务在后台运行。
    ```bash
    nohup airflow scheduler > scheduler.log 2>&1 &
    nohup airflow webserver > scheduler.log 2>&1 &
    ```

6. 通过浏览器访问`http://localhost:8080`来管理定时任务。Airflow已经提供了多个例子，你可以参考这些例子来配置自己的定时任务。
