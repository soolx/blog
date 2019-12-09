---
title: MySQL root密码重置（MariaDB适用）
date: 2017-05-19 16:34:00
tags:
---
> 停止mysqld 
```
service mysqld stop
```
或者
```
killall -TERM mysqld
```
> 以不检查权限的方式启动MySQL
```
mysqld --skip-grant-tables &
```
如果命令未找到，先找到mysqld位置`whereis mysqld`返回mysqld路径，使用你自己的路径替换"Your Path"执行`"Your Path"/mysqld --skip-grant-tables &`

> 然后用空密码方式使用root用户登录MySQL
```
mysql -u root -p
```
不输入密码直接Enter进入
> 修改root用户的密码
用你想要设置的密码替换NewPASSWORD
```
mysql> update mysql.user set password=PASSWORD('NewPASSWORD') where User='root'; 
mysql> flush privileges; 
mysql> quit 
```
> 重新启动MySQL
```
killall -TERM mysqld  
service mysqld restart
```