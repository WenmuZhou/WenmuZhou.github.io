---
title: ubuntu16.04 + cuda8.0 + cudnn6.0 + opencv3.2安装caffe
date: 2017/06/02 12:59
comments: true
categories: 
- 深度学习
tags: 
- caffe
- ubuntu
---

# step1 安装NVIDIA显卡驱动
因为我是重装了系统所以之前没有安过nvidia驱动，如果之前有装得话，可以自行删除，先 通过快捷键Ctrl+Alt+T打开终端
~~~c++
sudo apt-get remove --purge nvidia-*
~~~
下面开始安装 首先添加官方源
~~~c++
sudo add-apt-repository ppa:graphics-drivers/ppa
~~~
然后刷新软件库并安装(首先先去[NVIDIA官网](http://www.nvidia.com/Download/index.aspx?lang=en-us)查询自己适合的驱动)
~~~c++
sudo apt-get update
sudo apt-get install nvidia-375 nvidia-settings nvidia-prime
~~~
上面的nvidia-375根据你查询的结果自行更改 
Ps:这方法适用于Ubuntu16.04其他版本可能有问题 安装完成之后重启电脑 然后在命令行输入
~~~c++
nvidia-smi
~~~
出现显卡信息说明安装成功

# step2 安裝CUDA8.0
先下载[cuda8.0](https://developer.nvidia.com/cuda-downloads)
然后cd进下载目录执行
~~~c++
sudo sh cuda_8.0.61_375.26_linux.run --override
~~~
看到这么一步时，选择n
~~~c++
Install NVIDIA Accelerated Graphics Driver for Linux-x86_64 *?
(y)es/(n)o/(q)uit: n
~~~

# step3 修改~/.bashrc
~~~c++
sudo gedit ~/.bashrc
~~~
在最后加入
~~~c++
export PATH=/usr/local/cuda-8.0/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-8.0/lib64:$LD_LIBRARY_PATH
~~~
# step4 安裝cuDNN 6.0
先下载[cuDNN 6.0](https://developer.nvidia.com/rdp/cudnn-download)
然后执行
~~~c++
sudo tar xvf cudnn-8.0-linux-x64-v6.0.tgz
sudo cp cuda/include/cudnn.h /usr/local/cuda/include
sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*
~~~
# step5 安装依赖及opencv 3.2.0
~~~c++
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install build-essential cmake git pkg-config libprotobuf-dev libleveldb-dev libsnappy-dev libhdf5-serial-dev protobuf-compiler libatlas-base-dev libgflags-dev libgoogle-glog-dev liblmdb-dev python-pip python-dev python-numpy python-scipy
sudo apt-get install --no-install-recommends libboost-all-dev
pip install --upgrade pip
sudo pip install Cython

cd ~/
wget https://raw.githubusercontent.com/milq/milq/master/scripts/bash/install-opencv.sh
bash install-opencv.sh#这一步我执行了两次才成功
~~~
# step6 安装Caffe
下载caffe并进入目录
~~~c++
git clone https://github.com/BVLC/caffe.git
cd caffe
~~~
复制配置文件
~~~c++
cp Makefile.config.example Makefile.config
~~~
修改配置文件
~~~c++
sudo gedit Makefile.config #打开Makefile.config文件 根据个人情况修改文件：
~~~

a.若使用cudnn，则将
~~~c++
#USE_CUDNN := 1
~~~

修改成： 
~~~c++
USE_CUDNN := 1
~~~
b.若使用的opencv版本是3的，则
将
~~~c++
#OPENCV_VERSION := 3 
~~~
修改为：
~~~c++ 
OPENCV_VERSION := 3
~~~
c.若要使用python来编写layer，则
将       
~~~c++
#WITH_PYTHON_LAYER := 1  
~~~
修改为
~~~c++
 WITH_PYTHON_LAYER := 1 
~~~
d.重要的一项 :
将 
~~~c++
# Whatever else you find you need goes here. 下面的
~~~
~~~c++
INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include
LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib 
~~~
修改为：
~~~c++
INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include /usr/include/hdf5/serial
LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib /usr/lib/x86_64-linux-gnu /usr/lib/x86_64-linux-gnu/hdf5/serial       
~~~
这是因为ubuntu16.04的文件包含位置发生了变化，尤其是需要用到的hdf5的位置，所以需要更改这一路径.

# step7 编译caffe
~~~c++
make all
make pycaffe
make test
make runtest
~~~
在任意位置import caffe
~~~c++
sudo gedit ~/.bashrc   
export PYTHONPATH=~/caffe/python:$PYTHONPATH  
#上述语句中 “～” 号表示caffe 所在的根目录。
~~~
关闭文件，在终端输入下面命令，使环境变量生效
~~~c++
source ~/.bashrc  
~~~

# 编译python3版本
移除下面语句前面的注释
~~~c++
# Uncomment to use Python 3 (default is Python 2)
PYTHON_LIBRARIES := boost_python3 python3.5m
PYTHON_INCLUDE := /usr/include/python3.5m \
                 /usr/lib/python3.5/dist-packages/numpy/core/include
~~~
可能会遇到如下问题
~~~c++
CXX .build_release/src/caffe/proto/caffe.pb.cc
PROTOC (python) src/caffe/proto/caffe.proto
LD -o .build_release/lib/libcaffe.so.1.0.0-rc3
CXX/LD -o python/caffe/_caffe.so python/caffe/_caffe.cpp
/usr/bin/ld: cannot find -lboost_python3
collect2: error: ld returned 1 exit status
make: *** [python/caffe/_caffe.so] 错误 1
~~~
这时候，检查是否有如下文件：
 !!! 这里的py35根据你的python版本来，我的是python3.5，这里就是py35
~~~c++
ls /usr/lib/x86_64-linux-gnu/libboost_python-py35.so
~~~
如果有，说明我们的系统中已经有了这种库文件，只是文件名不同。接下来执行下面语句
~~~c++
cd /usr/lib/x86_64-linux-gnu/
sudo ln -s libboost_python-py35.so libboost_python3.so
~~~
重新编译即可。
finish

# reference
http://blog.csdn.net/Tang_DH/article/details/52556636
http://bleuren.me/106/install-caffe/