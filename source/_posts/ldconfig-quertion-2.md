---
title: CUDA及cuDNN函数库调用问题
date: 2017-05-29 00:57:10
tags:
---
##### CUDA及cuDNN函数库调用遇到错误
```
libcublas.so.8.0: cannot open shared object file: No such file or directory
```

>解决方案：

找到CUDA的安装路径，CUDA8.0一般在`/usr/local/cuda/lib64/`目录下。
根据自己安装的版本找到对应的路径。

执行`sudo vim /etc/ld.so.conf.d/cuda_lib.conf`创建cuda_lib.conf文件。

在文件中添加路径`/usr/local/cuda/lib64/`

然后执行`sudo ldconfig`

`LD_LIBRARY_PATH`是编译时bash shell可以使用的环境变量，但是程序在运行时，不一定会读取到这个变数，所以就要设定`/etc/ld.so.conf`.

