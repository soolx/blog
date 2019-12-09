---
title: Python环境依赖文件的创建及使用
date: 2017-05-15 09:15:42
tags:
---
> 在真实环境或虚拟环境中使用pip生成
```
(venv) $ pip freeze >requirements.txt
```
安装或升级包后，最好更新这个文件。

> 当需要创建这个环境的副本时，在相应环境运行以下命令：
```
(venv) $ pip install -r requirements.txt
```