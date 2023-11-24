---
title: 在Anaconda环境下安装Apache Airflow以进行定时任务管理：设置为系统服务
date: 2023-11-23T16:57:03+08:00
toc: true
tags: 
categories: 
- 应用软件
- Airflow
---

1. 下载系统服务配置模板
    ```
    wget https://raw.githubusercontent.com/apache/airflow/master/scripts/systemd/airflow
    wget https://raw.githubusercontent.com/apache/airflow/master/scripts/systemd/airflow-webserver.service
    wget https://raw.githubusercontent.com/apache/airflow/master/scripts/systemd/airflow-scheduler.service
    ```

2. 第一个文件配置了在启动系统服务时候需要设置的环境变量，即`AIRFLOW_HOME`，`AIRFLOW_CONFIG`，设置成和安装篇里面一样的值即可。而后将其拷贝到适当位置，推荐为`/etc/sysconfig/airflow`，并将所有权修改到运行Airflow的用户下。

3. 然后配置`airflow-scheduler.service`如下：

    `airflow-scheduler.service`：
    ```
    [Unit]
    Description=Airflow scheduler daemon
    After=network.target postgresql.service mysql.service redis.service rabbitmq-server.service
    Wants=postgresql.service mysql.service redis.service rabbitmq-server.service

    [Service]
    EnvironmentFile=<2中的环境变量配置文件>
    User=<airflow user>
    Group=<airflow user group>
    Type=simple
    ExecStart=<path to airflow>/airflow scheduler'
    Restart=always
    RestartSec=5s

    [Install]
    WantedBy=multi-user.target
    ```

4. 再配置`airflow-webserver.service`如下
    `airflow-webserver.service`：
    ```
    [Unit]
    Description=Airflow webserver daemon
    After=network.target postgresql.service mysql.service redis.service rabbitmq-server.service
    Wants=postgresql.service mysql.service redis.service rabbitmq-server.service

    [Service]
    EnvironmentFile=<2中的环境变量配置文件>
    User=<airflow user>
    Group=<airflow user group>
    Type=simple
    ExecStart=<path to airflow>/airflow webserver --pid /run/airflow/webserver.pid
    Restart=on-failure
    RestartSec=5s
    PrivateTmp=true

    [Install]
    WantedBy=multi-user.target
    ```
    这里我们发现需要一个`pid`文件
    ```
    mkdir /run/airflow
    chown <airflow user>:<airflow user group> /run/airflow
    ```

5. 然后启动服务
    ```
    sudo systemctl daemon-reload  # 让更改生效
    sudo systemctl start airflow-scheduler.service
    sudo systemctl start airflow-webserver.service
    ```


6. 但这个时候发现很奇怪的问题，`airflow-scheduler`服务总是时断时续，报错“找不到airflow”。
研究后发现，虽然我们通过指定`<path to airflow>/airflow`保证了`webserver`和`scheduler`本身是通过Anaconda环境运行的，但是`scheduler`在运行任务时候，是运行`airflow task run xxx ...`，这个时候由于`systemctl`并不会自动激活conda环境，所以就无法找到`airflow`这个命令。

    所以需要在`ExecStart`里面加入激活环境的相关命令，`airflow-scheduler.service`须变更如下：
    ```
    ExecStart=/bin/bash -c 'source /home/booksword/anaconda3/etc/profile.d/conda.sh && conda activate && airflow scheduler'
    ```
    `airflow-scheduler.service`变更如下：
    ```
    ExecStart=/bin/bash -c 'source /home/booksword/anaconda3/etc/profile.d/conda.sh && conda activate && airflow webserver --pid /run/airflow/webserver.pid'
    ```