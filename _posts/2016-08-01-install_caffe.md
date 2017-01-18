---
layout: post
title: Ubuntu 16.04 + caffe + openCV 3.1 配置
category: tech
comments: false
---

接上文([装机日志，配置一台基础的机器学习主机](https://jmydurant.github.io/blog/tech/2016/08/01/build_env.html))
，我们在此基础上安装caffe。

方法基本参照caffe官方的[wiki](https://github.com/BVLC/caffe/wiki)，可以直接按照官方方法进行安装
，当然也有许多需要注意的地方。

## gcc 与 g++ 的版本切换

这个东西先弄明白比较好，毕竟之后编译过程那么多，万一gcc与需要编译的程序不兼容就需要切换一下。我平时都用的 5.X的版本，
必要时，需要切回4.9版本。

- (1). 首先，我们查看下系统里面有多少gcc版本：

``` bash
ls /usr/bin/gcc*
```

- (2). 假设4.9没有，我们需要安装：

``` bash
sudo apt-get install gcc-4.9 gcc-4.9-multilib g++-4.9 g++-4.9-multilib
```

- (3). 之后输入如下命令：

``` bash
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.9 40 
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-5 50
```

其中，最后的数字代表优先级，数字越大优先级越高

- (4). 最后进行切换指令：

``` bash
sudo update-alternatives --config gcc
```

此时会出现如下所示选项：

```
  选择       路径            优先级  状态
------------------------------------------------------------
* 0            /usr/bin/gcc-5     50        自动模式
  1            /usr/bin/gcc-4.9   40        手动模式
  2            /usr/bin/gcc-5     50        手动模式

```

这样我们就能够自由选择版本了，利用`gcc -v`查看版本就知道切换是否成功了。
**需要注意的是，我们需要对g++也完整的做一遍。**

## 安装依赖项

还是老办法，首先更新源

``` bash
sudo apt-get update
sudo apt-get upgrade
```

然后开始安装各个细小的依赖项(参照wiki，我就照搬下来了)

``` bash
sudo apt-get install -y build-essential cmake git pkg-config
sudo apt-get install -y libprotobuf-dev libleveldb-dev libsnappy-dev libhdf5-serial-dev
sudo apt-get install -y protobuf-compiler libatlas-base-dev 
sudo apt-get install -y --no-install-recommends libboost-all-dev
sudo apt-get install -y libgflags-dev libgoogle-glog-dev liblmdb-dev
```

安装python相关项(python 2.7)：

``` bash
sudo apt-get install -y python-pip
sudo apt-get install -y python-dev
sudo apt-get install -y python-numpy python-scipy
```

## 安装openCV 3.1

对于openCV 2.X 版本，可以用如下命令(**本人并没有实验过**)：

``` bash
sudo apt-get install -y libopencv-dev
```

对于 openCV 3.X 版本，需要参考如下方法：

- (1). 安装依赖项：

``` bash
sudo apt-get install --assume-yes libopencv-dev build-essential cmake git libgtk2.0-dev pkg-config python-dev python-numpy libdc1394-22 libdc1394-22-dev libjpeg-dev libpng12-dev libtiff5-dev libjasper-dev libavcodec-dev libavformat-dev libswscale-dev libxine2-dev libgstreamer0.10-dev libgstreamer-plugins-base0.10-dev libv4l-dev libtbb-dev libqt4-dev libfaac-dev libmp3lame-dev libopencore-amrnb-dev libopencore-amrwb-dev libtheora-dev libvorbis-dev libxvidcore-dev x264 v4l-utils unzip
```

- (2). 下载[openCV 3.1](http://opencv.org/downloads.html)，解压至`$HOME`下面，文件夹重命名为`opencv`
- (3). 在安装之前，先解决一个兼容问题，由于现在openCV 3.1 与 CUDA 8.0 不是完全兼容，所以安装时会报错，
说`graphcuts.cpp`内有很多变量没有声明，我们需要找到`$HOME/opencv/modules/cudalegacy/src`下的`graphcuts.cpp`进行如下修改
，找到其中某一行如下：

``` c++
#if !defined (HAVE_CUDA) || defined (CUDA_DISABLER)
```

修改为：

``` c++
#if !defined (HAVE_CUDA) || defined (CUDA_DISABLER) || (CUDART_VERSION >= 8000)
```

- (4). 利用如下命令进行编译

``` bash
cd $HOME/opencv
mkdir build
cd build/
cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local -D WITH_TBB=ON -D WITH_V4L=ON -D WITH_QT=ON -D WITH_OPENGL=ON ..
make　-j8
```

- (5). 安装

``` bash
sudo make install
sudo /bin/bash -c 'echo "/usr/local/lib" > /etc/ld.so.conf.d/opencv.conf'
sudo ldconfig
sudo apt-get update
```

这样，openCV的安装就告一段落了。

## 安装CuDNN 5.0

这是一个能快速计算Deep Neural Network的包，最好进行安装。
下载好之后解压到`$HOME`下面，文件夹名字应该就叫cuda。
然后我们只需简单的复制文件即可：

``` bash
sudo cp cuda/include/cudnn.h /usr/local/cuda/include/
sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64/
sudo chmod a+r /usr/local/cuda/include/cudnn.h
sudo chmod a+r /usr/local/cuda/lib64/libcudnn*
```

## 安装caffe

如果我们在用`apt-get`命令时安装 `protobuf` ，有可能在编译caffe时产生错误。所以需要用编译安装的方式来安装它。
首先需要卸载它，然后下载[protobuf-2.5.0](https://github.com/google/protobuf/tree/v2.5.0)。
解压之后，进入文件夹，进行如下所示的安装：

``` bash
./autogen.sh
./configure
make
make check
make install
```

完成了这个之后，可以正式安装caffe了。

- (1). 下载[caffe](https://github.com/BVLC/caffe)，并且解压到`$HOME`命名为`caffe`
- (2). 修改配置文件：

``` bash
cd caffe
cp Makefile.config.example Makefile.config
vi Makefile.config
```

在配置文件中进行修改，修改结果如下：

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
 OPENCV_VERSION := 3

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
# MATLAB_DIR := /usr/local
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

- (3). 在这个系统中，hdf5由于命名不规范，所以make的时候caffe有可能找不到它，
所以我们还要做一下软链接：(具体so文件版本要看`find`命令的结果，**不要直接复制**)

``` bash
find . -type f -exec sed -i -e 's^"hdf5.h"^"hdf5/serial/hdf5.h"^g' -e 's^"hdf5_hl.h"^"hdf5/serial/hdf5_hl.h"^g' '{}' \;
cd /usr/lib/x86_64-linux-gnu
sudo ln -s libhdf5_serial.so.8.0.2 libhdf5.so
sudo ln -s libhdf5_serial_hl.so.8.0.2 libhdf5_hl.so
```

- (4). 如果编译caffe时gcc 版本为5.4以及以上，需要修改如下文件：

``` bash
sudo vi /usr/local/cuda/include/host_config.h 
```

找到如下内容，并将其注释：

``` c++
#if __GNUC__ > 5 || (__GNUC__ == 5 && __GNUC_MINOR__ > 3)

#error -- unsupported GNU version! gcc versions later than 5.3 are not supported!

#endif /* __GNUC__ > 5 || (__GNUC__ == 5 && __GNUC_MINOR__ > 1) */
```

- (5). 大胆的编译吧！！

``` bash
make all -j8
make test -j8
make runtest -j8
```

如果caffe的python接口没有编译，我们还可以执行`make pycaffe`。

好啦，安装到此为止，feel free to enjoy it！！！