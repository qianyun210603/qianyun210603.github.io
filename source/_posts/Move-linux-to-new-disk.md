---
title: 无损迁移Linux系统到新磁盘
date: 2018-07-01 21:02:10
categories: Linux
---
笔者的电脑原有一块SSD上装了Windows 10和Linux的双系统，其中Linux分配了大约300GB空间，但随着系统膨胀，发现空间逐步不够使用了，因此又买了一块1TB的SSD，打算将Linux整体迁移到1TB盘上，将原SSD上Linux部分都划给Windows 10。在此记录下迁移步骤，以供参考。
（此步骤基于Linux Mint，其他发行版应该大同小异。）

#### 迁移步骤
1. 硬件安装；
2. 刻录Linux安装U盘并以之启动电脑；
3. 用`fdisk`命令分区，首先分2GBswap分区，其他作为迁移后的文件系统分区：
    1. 用`fdisk -l`命令检查当前磁盘挂载和分区情况，找出并记录下原SSD上Linux的swap分区和文件系统分区，以及新SSD的挂载点，假设分别是`/dev/sda2`, `/dev/sda3`,和`/dev/sdd`；
    2. 用`fdisk /dev/sdd`给新磁盘分为swap分区和文件系统分区：
        ```bash
        Command (m for help): n # create partition of 2GB for swap area
        Partition number (1-128, default 1): 
        First sector (1-10000000, default 2048): 
        Last sector, +sectors or +size{K,M,G,T,P} (34-2047, default 10000000): +2G
        
        Created a new partition 1 of type 'Linux filesystem' and of size 2 GiB.
        
        Command (m for help): n # create partition with remaining space for main filesystem
        Partition number (2-128, default 2): 
        First sector (600001-10000000): 
        Last sector, +sectors or +size{K,M,G,T,P} (600002-10000000, default 10000000): 
        
        Created a new partition 3 of type 'Linux filesystem' and of size 600 GiB.
        ```
    3. 再次`fdisk -l`查看新磁盘的分区，假设分别是`/dev/sdd1`和`/dev/sdd2`；
    4. 格式化新分区：
        ```bash
        mkfs.ext4 /dev/sdd1
        mkfs.ext4 /dev/sdd2
        ```
    
3. 用`dd`命令复制旧SSD上Linux各分区到对应的新SSD分区上：
    ```bash
    dd if=/dev/sda2 of=/dev/sdd1
    dd if=/dev/sda3 of=/dev/sdd2
    ```
    取决于文件分区大小，复制可能需要数个小时。此外，如果之前`/boot`是单独挂载的，在步骤3也需要创建对应的分区并拷贝内容。
    
4. 更新磁盘信息：
    ```bash
    e2fsck -f /dev/sdd1
    resize2fs /dev/sdd1
    ```
5. 使用`gparted`更新磁盘的UUID。完成后对应更新`/etc/fstab`中关于Linux系统的分区UUID：
    * 使用`blkid /dev/sdd1`获得新磁盘上分区的UUID；
    * 挂载新磁盘上的分区并更新位于`/<mount dir>/etc/fstab`为新UUID；
    * 重启进入Windows 10，进入`管理工具->磁盘管理`，删除原SSD上的Linux分区；

6. 用Linux安装盘再次启动系统，用boot-repair修复grub；
    1. 安装并打开boot-repair
        ```bash
        sudo add-apt-repository ppa:yannubuntu/boot-repair
        sudo apt-get update
        sudo apt-get install -y boot-repair
        boot-repair
        ```
    2. advanced mode选项卡中：
        * "GRUB location"选项卡, 选中"Separate /boot/efi partition"，并选择Linux所在的分区。
        * "GRUB options"选项卡，选中"Purge GRUB before reinstalling it"。
    
7. 重启，此时应该可以进入迁移后的Linux了。