---
title: Linux 常用命令
date: 2018/04/12 13:45
comments: true
categories: 
- Linux
tags: 
- Linux
---

# 登录服务器
登录服务器并设置可显示服务器软件GUI
```sh
ssh -X zj@ip
```

# 压缩及解压
## 7z
### 压缩
```sh
7za a -t7z -r Mytest.7z /opt/phpMyAdmin-3.3.8.1-all-languages/*
```
参数含义：
a  代表添加文件／文件夹到压缩包
-t 是指定压缩类型，这里定为7z，可不指定，因为7za默认压缩类型就是7z。
-r 表示递归所有的子文件夹
Mytest.7z 是压缩好后的压缩包名
/opt/phpMyAdmin-3.3.8.1-all-languages/*：是目录压缩目标。

### 解压
```sh 
7za x xxx.7z -r -o./
```

## zip
### 压缩
将当前目录下的所有文件和文件夹全部压缩成myfile.zip文件,－r表示递归压缩子目录下所有文件.
```sh
zip -r myfile.zip ./*
```

### 解压
```sh
zip -r filename.zip filesdir
```
在这个例子里，filename.zip 代表你创建的文件，filesdir 代表你想放置新 zip 文件的目录。-r 选项指定你想递归地（recursively）包括所有包括在 filesdir 目录中的文件。

要抽取 zip 文件的内容，键入以下命令：
```sh
unzip filename.zip
```

你可以使用 zip 命令同时处理多个文件和目录，方法是将它们逐一列出，并用空格间隔：
```sh
zip -r filename.zip file1 file2 file3 /usr/work/school 
```
上面的命令把 file1、file2、 file3、以及 /usr/work/school 目录的内容（假设这个目录存在）压缩起来，然后放入 filename.zip 文件中。