---
layout: "post"
title: "Ubuntu 22.04安装Cuda Toolkit"
date: "2022-11-03 11:54"
categories:
  - Linux
  - Cuda
---

Nvidia现在要求安装CuDNN和NICC都得登录Nvidia Developer网站，国内巨慢。药丸。

#### CUDA
```bash
sudo -i # 切换root权限
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.0-1_all.deb
dpkg -i cuda-keyring_1.0-1_all.deb
apt update && apt install cuda -y
```

#### NICC
```bash
apt install libnccl2=2.15.5-1+cuda11.8 libnccl-dev=2.15.5-1+cuda11.8
```

#### cuDNN
```bash
dpkg -i cudnn-local-repo-ubuntu2204-8.6.0.163_1.0-1_amd64.deb
cp /var/cudnn-local-repo-ubuntu2204-8.6.0.163/cudnn-local-FAED14DD-keyring.gpg /usr/share/keyrings/
apt install libcudnn8 libcudnn8-dev libcudnn8-samples
```
###### 测试cuDNN
mnistCUDNN测试需要FreeImage。
```bash
apt install libfreeimage3 libfreeimage-dev
```
将sample拷贝到有写权限的目录，并正确设置权限。
```bash
cp /usr/src/cudnn_samples_v8/ . -R
```
进入每个目录`make`生成可执行文件，运行。如都可以运行，显示`Test passed!`，则安装成功。
