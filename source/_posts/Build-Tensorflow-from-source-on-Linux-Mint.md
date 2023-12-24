---
title: Build Tensorflow from source on Linux Mint
date: 2018-06-26 21:02:09
tags: Linux
categories:
  - AI
  - 框架
---
1. Install Bazel (a )
    1. Install JDK 8
        Install JDK 8 by using:
        ```bash
        apt install openjdk-8-jdk
        ```
    2. Add Bazel distribution URI as a package source (one time setup)
        ```bash
        echo "deb [arch=amd64] http://storage.googleapis.com/bazel-apt stable jdk1.8" | tee /etc/apt/sources.list.d/bazel.list
        curl https://bazel.build/bazel-release.pub.gpg | sudo apt-key add -
        ```
        If you want to install the testing version of Bazel, replace stable with testing.
    3. Install and update Bazel
        ```bash
        apt update
        apt install bazel
        ```
2. Install CUDA and cuDNN (Details ignored, refer to )
3. Clone the TensorFlow repository
    ```bash
    git clone --recurse-submodules https://github.com/tensorflow/tensorflow
    ```
    `--recurse-submodules` is used to clone the dependency library <font color=#ff0000>protobuf</font>.
4. Install other dependency
    ```bash
    apt-get install python3-numpy swig python3-dev libgrpc-dev
    ```

5. Configuration
    ```bash
    You have bazel 0.10.1 installed.
    Please specify the location of python. [Default is /usr/bin/python]: /usr/local/anaconda3/bin/python


    Found possible Python library paths:
      /usr/local/anaconda3/lib/python3.6/site-packages
    Please input the desired Python library path to use.  Default is [/usr/local/anaconda3/lib/python3.6/site-packages]

    Do you wish to build TensorFlow with jemalloc as malloc support? [Y/n]: y
    jemalloc as malloc support will be enabled for TensorFlow.

    Do you wish to build TensorFlow with Google Cloud Platform support? [Y/n]: y
    Google Cloud Platform support will be enabled for TensorFlow.

    Do you wish to build TensorFlow with Hadoop File System support? [Y/n]: n
    No Hadoop File System support will be enabled for TensorFlow.

    Do you wish to build TensorFlow with Amazon S3 File System support? [Y/n]: n
    No Amazon S3 File System support will be enabled for TensorFlow.

    Do you wish to build TensorFlow with Apache Kafka Platform support? [y/N]: n
    No Apache Kafka Platform support will be enabled for TensorFlow.

    Do you wish to build TensorFlow with XLA JIT support? [y/N]: y
    XLA JIT support will be enabled for TensorFlow.

    Do you wish to build TensorFlow with GDR support? [y/N]: n
    No GDR support will be enabled for TensorFlow.

    Do you wish to build TensorFlow with VERBS support? [y/N]: n
    No VERBS support will be enabled for TensorFlow.

    Do you wish to build TensorFlow with OpenCL SYCL support? [y/N]: n
    No OpenCL SYCL support will be enabled for TensorFlow.

    Do you wish to build TensorFlow with CUDA support? [y/N]: y
    CUDA support will be enabled for TensorFlow.

    Please specify the CUDA SDK version you want to use, e.g. 7.0. [Leave empty to default to CUDA 9.0]: 9.1


    Please specify the location where CUDA 9.1 toolkit is installed. Refer to README.md for more details. [Default is /usr/local/cuda]:


    Please specify the cuDNN version you want to use. [Leave empty to default to cuDNN 7.0]: 7.0.5


    Please specify the location where cuDNN 7 library is installed. Refer to README.md for more details. [Default is /usr/local/cuda]: /usr


    Do you wish to build TensorFlow with TensorRT support? [y/N]: n
    No TensorRT support will be enabled for TensorFlow.

    Please specify a list of comma-separated Cuda compute capabilities you want to build with.
    You can find the compute capability of your device at: https://developer.nvidia.com/cuda-gpus.
    Please note that each additional compute capability significantly increases your build time and binary size. [Default is: 3.5,5.2]6.1


    Do you want to use clang as CUDA compiler? [y/N]: n
    nvcc will be used as CUDA compiler.

    Please specify which gcc should be used by nvcc as the host compiler. [Default is /usr/bin/gcc]:


    Do you wish to build TensorFlow with MPI support? [y/N]: y
    MPI support will be enabled for TensorFlow.

    Please specify the MPI toolkit folder. [Default is ]: /usr


    Please specify optimization flags to use during compilation when bazel option "--config=opt" is specified [Default is -march=native]:


    Would you like to interactively configure ./WORKSPACE for Android builds? [y/N]: n
    Not configuring the WORKSPACE for Android builds.

    Preconfigured Bazel build configs. You can use any of the below by adding "--config=<>" to your build command. See tools/bazel.rc for more details.
    	--config=mkl         	# Build with MKL support.
    	--config=monolithic  	# Config for mostly static monolithic build.
    Configuration finished

    ```

6. Build
    ```bash
    bazel build -c opt --config=cuda //tensorflow/cc:tutorials_example_trainer
    ```
7. Test run
    ```bash
    bazel-bin/tensorflow/cc/tutorials_example_trainer --use_gpu
    ```

8. Build pip package
    * Create an env under Anaconda specified for tensorflow (assuming that Anaconda is installed)
        ```bash
        /usr/local/anaconda3/bin/conda create -n tensorflow pip python=3.6.4 anaconda
        ```
    * Activate the env
        ```bash
        source /usr/local/anaconda3/bin/activate tensorflow
        ```
    * Build under the env
        ```bash
        # To resolve the error caused by location of mpi header files in system is not consistent with tensorflow
        rm -rf third_party/mpi/mpi*.h
        ln -s /usr/include/mpich/mpi.h third_party/mpi/mpi.h
        ln -s /usr/include/mpich/mpicxx.h third_party/mpi/mpicxx.h
        ln -s /usr/include/mpich/mpio.h third_party/mpi/mpio.h

        # build
        bazel build --config=opt --config=cuda //tensorflow/tools/pip_package:build_pip_package

        # generate wheel in /tmp/tensorflow_pkg
        bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg
        ```

9. Install under Anaconda env
    ```bash
    pip install /tmp/tensorflow_pkg/tensorflow-1.6.0rc1-cp36-cp36m-linux_x86_64.whl
    ```

10. Test installation
    ```bash
    # enter tensorflow env
    source /usr/local/anaconda3/bin/activate tensorflow

    # enter python
    python
    ```
    In python interactive
    ```python
    Python 3.6.4 |Anaconda, Inc.| (default, Jan 16 2018, 18:10:19)
    [GCC 7.2.0] on linux
    Type "help", "copyright", "credits" or "license" for more information.
    >>> import tensorflow as tf
    >>> tf.__version__
    '1.6.0-rc1'
    >>>
    ```


##### Reference
1. Bazel installation guide: https://docs.bazel.build/versions/master/install-ubuntu.html
2. Tensorflow: https://tensorflow.google.cn/install/install_sources
