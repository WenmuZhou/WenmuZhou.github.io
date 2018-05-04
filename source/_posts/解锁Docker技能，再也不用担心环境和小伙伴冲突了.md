---
title: 解锁Docker技能，再也不用担心环境和小伙伴冲突了
date: 2017/12/27 17:44
comments: true
categories: 
- 深度学习
tags: 
- docker
---

# 以基于[ubuntu](https://hub.docker.com/_/ubuntu/)镜像创建自己的镜像并打包带走为例解读docker的使用方法。

ps:docker命令和git命令极其相似
# step1 查找ubuntu镜像
在安装好docker之后，我们需要使用如下命令来查找需要的镜像
```sh
docker search 镜像关键字
```
![查找docker镜像](http://upload-images.jianshu.io/upload_images/1575688-6f627660a9c25412.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# step2 下载ubuntu镜像
查找完成之后，使用如下命令来下载需要的镜像
```sh
docker pull 镜像仓库全称
```
![下载镜像](http://upload-images.jianshu.io/upload_images/1575688-fa2bfa8b438eea45.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# step3 启动ubuntu镜像
下载完成之后，使用如下命令来下载启动的镜像
```sh
nvidia-docker run --name zj-ubuntu -i -t ubuntu
```
其中，`--name`表示使用基于`ubuntu`镜像创建一个名称为`zj-ubuntu`的临时镜像，`-i` 以交互模式运行容器，`-t` 为容器重新分配一个伪输入终端，加上`-i -t`之后，当前终端会被镜像里的终端代替
![运行镜像](http://upload-images.jianshu.io/upload_images/1575688-8fb33227caaa79c2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

此时通过下面的命令可以查询当前运行的镜像信息
```sh
docker ps -a
```
![image.png](http://upload-images.jianshu.io/upload_images/1575688-4bb61ffa9e74ef6e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

启动时，挂载本地文件夹
```sh
nvidia-docker run --name zj-ubuntu -i -t -v /data:/data ubuntu
```
`-v`表示挂载本地文件夹，后面的`/data:/data`，前一个`/data`表示宿主机文件夹，后一个`/data`表示镜像里显示的文件夹。

至此，已经完成了镜像的下载和运行，但是在运行镜像之后搭好的环境怎么保存呢？

# step4 退出容器
通过`run`命令开始的镜像，会形成一个容器，这是可以理解为，我们的机器开机了，想关机，直接输入`exit`就能够完成关机，此时使用下面命令可以获得容器id
```sh
docker ps -a
```
获得容器id之后，可以使用下面的命令开机并再次进入。
```sh
docker start 容器id或容器名  #开机
docker attach 容器id或容器名 #登录
```

# step5 保存ubuntu镜像
在对镜像做完更改之后，使用如下命令来保存更改
```sh
docker commit -a "zj" -m "456" 93934b35e497 zj-ubuntu1:1.0
```
`-a`表示作者信息，`-m`表示此次更改的提交信息，`93934b35e497 `为镜像id，`zj-ubuntu1`是镜像保存的仓库名，`1.0`是镜像的tag信息，可以理解为版本号。

# step6 提交zj-ubuntu镜像
之前我们已经保存了我们对于`ubuntu`镜像的更改，并以此创建了一个新的镜像`zj-ubuntu1`(docker不会再原有的镜像上做更，每次commit都会创建一个新的)。

接下来，我们将创建好的镜像提交到https://hub.docker.com上，已完成打包带走的初衷。

使用如下命令可以提交指定的docker镜像到https://hub.docker.com
```sh
docker push 镜像仓库地址
```
在提交之前，我们需要使用如下命令登录
```sh
docker login
```
随后使用如下命令提交镜像
```sh
docker push zj-ubuntu1:1.0
```
就这么提交会报错
![提交报错](http://upload-images.jianshu.io/upload_images/1575688-e7ff2111bfc677c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到，提交被拒绝了，为什么呢？其实是因为，docker在提交时找不到`zj-ubuntu`这个用户，我们需要使用下面的命令来对镜像仓库进行修改
```sh
 docker tag zj-ubuntu1:1.0 wenmu/zj-ubuntu:1.0
```
`wenmu`是我的docker用户名。
修改完成之后，使用下面的命令就可以愉快的提交了。
```sh
 docker push wenmu/zj-ubuntu
```
![提交完成](http://upload-images.jianshu.io/upload_images/1575688-bddb700a1bfeac2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

去官网检查一下
![打包带走](http://upload-images.jianshu.io/upload_images/1575688-cc8f53006b3849e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到，提交成功，至此，打包带走完成。