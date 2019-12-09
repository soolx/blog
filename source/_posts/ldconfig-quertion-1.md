---
title: Ubuntu的ldconfig详解(解决*.so不是符号连接)
date: 2017-05-29 00:42:50
tags:
---
自己在CUDA cudnn这一块出现错误

`/sbin/ldconfig.real: /usr/local/cuda/lib64/libcudnn.so.5 不是符号连接`
>解决方案：

在终端输入：
`sudo ldconfig -v`
找到这一行错误：`libcudnn.so.5 -> libcudnn.so.5.1.10`
根据找到的路径执行下面的命令，例如在终端输入：
```
sudo ln -sf /usr/local/cuda-8.0/lib64/libcudnn.so.5.1.10 /usr/local/cuda-8.0/lib64/libcudnn.so.5
sudo ldconfig
```
问题就解决了。

其他*.so不是符号连接的错误可以参照该解决方案。