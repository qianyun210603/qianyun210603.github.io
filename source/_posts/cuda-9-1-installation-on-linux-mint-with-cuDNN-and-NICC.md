---
title: Linux Mint安装Cuda 9.1以及cuDNN、NICC
date: 2018-07-02 21:02:10
tags: Linux
categories:
- Programming
- Cuda
---
#### CUDA

```bash
# execute following command as root user
sudo -i
dpkg -i cuda-repo-ubuntu1604_9.1.85-1_amd64.deb
apt-key adv --fetch-keys http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/7fa2af80.pub
apt update
apt install cuda
chmod a+x /usr/local/cuda-9.1/bin/cuda-install-samples-9.1.sh
# install cuda sample dependencies
apt install libgl-dev libglu-dev libx11-dev libegl1-mesa-dev libgles2-mesa-dev libmpich-dev
# back to normal user
exit
# install cuda examples
cuda-install-samples-9.1.sh ./
# force input macro value as linux mint is not officially supported
GLPATH="/usr/lib/x86_64-linux-gnu" GLLINK="-L/usr/lib/x86_64-linux-gnu" DFLT_PATH="/usr/lib" EGLLIB="/usr/lib/x86_64-linux-gnu" GLESLIB="/usr/lib/x86_64-linux-gnu" make -j6
```

#### cuDNN
首先从https://developer.nvidia.com/rdp/cudnn-download
* cuDNN v7.0.5 Runtime Library for Ubuntu16.04 (Deb)
* cuDNN v7.0.5 Developer Library for Ubuntu16.04 (Deb)
* cuDNN v7.0.5 Code Samples and User Guide for Ubuntu16.04 (Deb)


接下来，安装
```bash
dpkg -i libcudnn7_7.0.5.15-1+cuda9.1_amd64.deb
dpkg -i libcudnn7-dev_7.0.5.15-1+cuda9.1_amd64.deb
dpkg -i libcudnn7-doc_7.0.5.15-1+cuda9.1_amd64.deb
```

最后，运行官方测试用例以验证安装：

复制`/usr/src/cudnn_samples_v7/`到有写权限的目录下
```bash
cd cudnn_samples_v7/mnistCUDNN/
make
```

#### NICC
从https://developer.nvidia.com/nccl/nccl-download下载并 `dpkg` 安装。
```bash
dpkg -i nccl-repo-ubuntu1604-2.1.15-ga-cuda9.1_1-1_amd64.deb
```
