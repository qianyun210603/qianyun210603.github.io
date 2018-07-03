---
title: cuda 9.1 installation on linux mint with cuDNN and NICC
date: 2018-06-26 21:02:10
tags:
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
First download
* cuDNN v7.0.5 Runtime Library for Ubuntu16.04 (Deb)
* cuDNN v7.0.5 Developer Library for Ubuntu16.04 (Deb)
* cuDNN v7.0.5 Code Samples and User Guide for Ubuntu16.04 (Deb)    

from https://developer.nvidia.com/rdp/cudnn-download

Then, install

```bash
dpkg -i libcudnn7_7.0.5.15-1+cuda9.1_amd64.deb
dpkg -i libcudnn7-dev_7.0.5.15-1+cuda9.1_amd64.deb
dpkg -i libcudnn7-doc_7.0.5.15-1+cuda9.1_amd64.deb
```

Finally, verify

Copy /usr/src/cudnn_samples_v7/ to any place that can write
```bash
cd cudnn_samples_v7/mnistCUDNN/
make
```

#### NICC
First download from https://developer.nvidia.com/nccl/nccl-download
```bash
dpkg -i nccl-repo-ubuntu1604-2.1.15-ga-cuda9.1_1-1_amd64.deb
```
