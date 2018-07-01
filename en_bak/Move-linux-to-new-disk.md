---
title: Move linux to new disk
date: 2018-06-26 21:02:10
tags:
---
#### Task
My desktop previously have one SSD and one HDD where SDD contains dual system Windows 10 and Linux Mint 18.3. HDD serves as data disk for Windows 10. Originally linux partition was assigned only ~300GB space and now it is not enough for usage. Therefore another 1TB SSD was purchased and I hope to move the Linux system completely to the new SSD. The space of old linux will be freed for windows.

#### Step
1. Hardware installation
2. Boot system with a live CD
3. Use fdisk to partition the new disk, first a 2GB swap area and remaining space is filesystem for moved system. 
    1. Use `fdisk -l` to check the current disk status, find out where the linux filesystem and swap area locates and the new disk. Remember the partition points of them. For example, let's assume they are `/dev/sda2`, `/dev/sda3`, and `/dev/sdd` respectively.
    2. Use `fdisk /dev/sdd` to partition the new disk
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
    3. `fdisk -l` again to check partitions of new disk. Say filesystime partition is `/dev/sdd1` and swap area is `/dev/sdd2`
    4. Format the partitions
        ```bash
        mkfs.ext4 /dev/sdd1
        mkfs.ext4 /dev/sdd2
        ```
    
3. Copy partitions of old linux to new disk with `dd` command.
    ```bash
    dd if=/dev/sda2 of=/dev/sdd1
    dd if=/dev/sda3 of=/dev/sdd2
    ```
    For filesystem, it can take hours depending on size. If `/boot` was separately mounted when previous linux mint was installed, that partition should also be created and copied.

4. Update disk information
    ```bash
    e2fsck -f /dev/sdd1
    resize2fs /dev/sdd1
    ```
5. Modify uuid with `gparted`. Once new uuid generated, update in the `/etc/fstab` of the copied linux system:
    * check new uuid with `blkid /dev/sdd1`
    * mount new disk and update the uuid in `/<mount dir>/etc/fstab` to new uuid
    * reboot and enter Windows 10. Go to Disk Management, delete the old linux partitions.

6. Reboot again with liveCD. Use boot-repair to fix grub
    1. Install and open boot-repair
        ```bash
        sudo add-apt-repository ppa:yannubuntu/boot-repair
        sudo apt-get update
        sudo apt-get install -y boot-repair
        boot-repair
        ```
    2. Go to advanced mode, 
        * "GRUB location" tab, check "Separate /boot/efi partition" checkbox, and select previous the disk previous linux was on.
        * "GRUB options" tab, check "Purge GRUB before reinstalling it"
    
7. Reboot
