---
layout: post
title: 一些关于caffe的小坑
category: tech
comments: false
---

## 起因

宇亮说，需要弄一个关于caffe的新手提示。故我先在这弄一个草稿放在这里。

之前写了一个关于caffe安装的，[在这里](https://jmydurant.github.io/blog/tech/2016/08/01/install_caffe.html)。不幸的是，他有点过时了。
对于最新的caffe版本，安装已经友好多了，所以在这里更新一下吧。

## 机器环境

这里实验室环境是两个TiTian Pascal，CUDA 8.0，Cudnn5.1

Ubuntu版本是16，如果要编译老版本的caffe，之后会给一些建议。

## 配置依赖项

之前安装NVIDIA显卡驱动使用的apt更新，这里但是在运行opengl的模拟器的时候会出错，所以我们这次我们安装run文件，并且不安装opengl相关的文件。
命令如下:

``` bash
# 安装370版本的驱动
sudo sh NVIDIA-Linux-x86_64-375.20.run --no-opengl-files
```

安装CUDA的时候同理

``` bash
sudo sh cuda_8.0.44_linux.run --no-opengl-libs
```

安装完需要把cuda的库目录export到动态库的PATH上，即需要修改```~/.bashrc```

``` bash
sudo vi ~/.bashrc
```

``` bash
# 在 .bashrc 中添加
export CUDA_HOME=/usr/local/cuda-8.0
export LD_LIBRARY_PATH=${CUDA_HOME}/lib64:$LD_LIBRARY_PATH
```

CuDnn 下载源代码，然后直接复制到相关位置

``` bash
sudo cp cuda/include/cudnn.h /usr/local/cuda/include/
sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64/
sudo chmod a+r /usr/local/cuda/include/cudnn.h
sudo chmod a+r /usr/local/cuda/lib64/libcudnn*
```

**当然，如果机器已经安装好这些驱动，请跳过这些步骤！！！不要重复安装驱动！！！（我们实验室的小伙伴这些步骤都不用做了！！）**

然后是安装相关软件依赖，boost，lmdb，glog，blas啥的

``` bash
# 先更新一下源，交大的源挺不错的
sudo apt-get update
sudo apt-get upgrade

# 相关库
sudo apt-get install -y build-essential cmake git pkg-config
sudo apt-get install -y libprotobuf-dev libleveldb-dev libsnappy-dev libhdf5-serial-dev
sudo apt-get install -y protobuf-compiler libatlas-base-dev 
sudo apt-get install -y --no-install-recommends libboost-all-dev
sudo apt-get install -y libgflags-dev libgoogle-glog-dev liblmdb-dev

# python 2.7相关项
sudo apt-get install -y python-pip
sudo apt-get install -y python-dev
sudo apt-get install -y python-numpy python-scipy

# openCV 2.X 如果是 3.X 的话，请参考之前的安装过程。
sudo apt-get install -y libopencv-dev
```

## 编辑makefile的配置文件

- 1. 首先，去github上面clone最新的caffe，放到目标文件夹，比如说```~/dev```

``` bash
cd ~/dev

git clone https://github.com/BVLC/caffe.git
```

- 2. 编辑配置文件

```
cd caffe
cp Makefile.config.example Makefile.config
vi Makefile.config
```

- 3. 修改具体的项，为了编译matcaffe，这里我们也输入了matlab的位置。

``` yaml
## Refer to http://caffe.berkeleyvision.org/installation.html
# Contributions simplifying and improving our build system are welcome!

# cuDNN acceleration switch (uncomment to build with cuDNN).
 USE_CUDNN := 1

# CPU-only switch (uncomment to build without GPU support).
# CPU_ONLY := 1

# uncomment to disable IO dependencies and corresponding data layers
# USE_OPENCV := 0
# USE_LEVELDB := 0
# USE_LMDB := 0

# uncomment to allow MDB_NOLOCK when reading LMDB files (only if necessary)
#	You should not set this flag if you will be reading LMDBs with any
#	possibility of simultaneous read and write
# ALLOW_LMDB_NOLOCK := 1

# Uncomment if you're using OpenCV 3
# OPENCV_VERSION := 3

# To customize your choice of compiler, uncomment and set the following.
# N.B. the default for Linux is g++ and the default for OSX is clang++
# CUSTOM_CXX := g++

# CUDA directory contains bin/ and lib/ directories that we need.
CUDA_DIR := /usr/local/cuda
# On Ubuntu 14.04, if cuda tools are installed via
# "sudo apt-get install nvidia-cuda-toolkit" then use this instead:
# CUDA_DIR := /usr

# CUDA architecture setting: going with all of them.
# For CUDA < 6.0, comment the *_50 lines for compatibility.
CUDA_ARCH := -gencode arch=compute_20,code=sm_20 \
		-gencode arch=compute_20,code=sm_21 \
		-gencode arch=compute_30,code=sm_30 \
		-gencode arch=compute_35,code=sm_35 \
		-gencode arch=compute_50,code=sm_50 \
		-gencode arch=compute_50,code=compute_50

# BLAS choice:
# atlas for ATLAS (default)
# mkl for MKL
# open for OpenBlas
BLAS := atlas
# Custom (MKL/ATLAS/OpenBLAS) include and lib directories.
# Leave commented to accept the defaults for your choice of BLAS
# (which should work)!
# BLAS_INCLUDE := /path/to/your/blas
# BLAS_LIB := /path/to/your/blas

# Homebrew puts openblas in a directory that is not on the standard search path
# BLAS_INCLUDE := $(shell brew --prefix openblas)/include
# BLAS_LIB := $(shell brew --prefix openblas)/lib

# This is required only if you will compile the matlab interface.
# MATLAB directory should contain the mex binary in /bin.
 MATLAB_DIR := /home/XXX/MATLAB/R2015b
# MATLAB_DIR := /Applications/MATLAB_R2012b.app

# NOTE: this is required only if you will compile the python interface.
# We need to be able to find Python.h and numpy/arrayobject.h.
PYTHON_INCLUDE := /usr/include/python2.7 \
		/usr/lib/python2.7/dist-packages/numpy/core/include
# Anaconda Python distribution is quite popular. Include path:
# Verify anaconda location, sometimes it's in root.
# ANACONDA_HOME := $(HOME)/anaconda
# PYTHON_INCLUDE := $(ANACONDA_HOME)/include \
		# $(ANACONDA_HOME)/include/python2.7 \
		# $(ANACONDA_HOME)/lib/python2.7/site-packages/numpy/core/include \

# Uncomment to use Python 3 (default is Python 2)
# PYTHON_LIBRARIES := boost_python3 python3.5m
# PYTHON_INCLUDE := /usr/include/python3.5m \
#                 /usr/lib/python3.5/dist-packages/numpy/core/include

# We need to be able to find libpythonX.X.so or .dylib.
PYTHON_LIB := /usr/lib
# PYTHON_LIB := $(ANACONDA_HOME)/lib

# Homebrew installs numpy in a non standard path (keg only)
# PYTHON_INCLUDE += $(dir $(shell python -c 'import numpy.core; print(numpy.core.__file__)'))/include
# PYTHON_LIB += $(shell brew --prefix numpy)/lib

# Uncomment to support layers written in Python (will link against Python libs)
# WITH_PYTHON_LAYER := 1

# Whatever else you find you need goes here.
INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include /usr/include/hdf5/serial
LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib /usr/lib/x86_64-linux-gnu /usr/lib/x86_64-linux-gnu/hdf5/serial

# If Homebrew is installed at a non standard location (for example your home directory) and you use it for general dependencies
# INCLUDE_DIRS += $(shell brew --prefix)/include
# LIBRARY_DIRS += $(shell brew --prefix)/lib

# Uncomment to use `pkg-config` to specify OpenCV library paths.
# (Usually not necessary -- OpenCV libraries are normally installed in one of the above $LIBRARY_DIRS.)
# USE_PKG_CONFIG := 1

# N.B. both build and distribute dirs are cleared on `make clean`
BUILD_DIR := build
DISTRIBUTE_DIR := distribute

# Uncomment for debugging. Does not work on OSX due to https://github.com/BVLC/caffe/issues/171
# DEBUG := 1

# The ID of the GPU that 'make runtest' will use to run unit tests.
TEST_GPUID := 0

# enable pretty build (comment to see full commands)
Q ?= @

```

这里需要注意的是，如果不需要matlab的话，是可以不用编辑的。
并且```INCLUDE_DIRS```和```LIBRARY_DIRS```需要修改一下，其中hdf5的地址根据自己的操作系统进行修改。

如果需要Cudnn和openCV3，那么就去掉相关注释。

## 编译 caffe

之前由于caffe和CUDA 8.0出现不兼容的问题，需要修改一些地方。但是现在完全不需要了，直接编译即可！

``` bash
#编译caffe主体
make all -j20
make test -j20
make runtest -j20
``` 

如果需要编译pycaffe或者matcaffe,则

``` bash
# pycaffe
make pycaffe

# matcaffe
make matcaffe
```

如果没有出现错误，那么恭喜你，编译成功！！

## 杂项

- 1. Q: 实验室小朋友很多人需要使用Faster RCNN。但是其中caffe版本过老，应该怎么解决呢？

>> A: 可以参考[这里](https://github.com/rbgirshick/py-faster-rcnn/issues/237)。

>> 我来解释一下这里安装的大概过程，clone了Faster RCNN之后，进入caffe目录，merge最新的caffe(**注意，以后遇到caffe过老的问题时，用这个方法可能会有用，但是请慎用**)。

>> 因为merge了caffe之后要解决caffe的兼容问题，所以修改了文件的一处位置。这里仍然不能使用CuDnn

如果懒得看，那么就做如下操作：

``` bash
cd caffe-fast-rcnn  
git remote add caffe https://github.com/BVLC/caffe.git  
git fetch caffe  
git merge caffe/master
```

然后在```include/caffe/layers/python_layer.hpp```中删除下面这句话.

``` c++
self_.attr("phase") = static_cast<int>(this->phase_);
```

- 2. Q: matcaffe警告说g++版本不符合，怎么办？

>> A: 辣鸡Matlab，编译的时候用g++ 5.0之后的版本是没有啥大的问题的。Matlab的libstc++有些过时，如果出现问题，可以将这个动态库软连接到自己系统比较新的库就可以解决。

>> 大致方法如下:

```
# 把PATH_TO_MATLAB换成MATLAB所在位置
ln -s /usr/lib/x86_64-linux-gnu/libstdc++.so.6 /PATH_TO_MATLAB/sys/os/glnxa64/libstdc++.so.6
```

最好的方式就是用python，一劳永逸。