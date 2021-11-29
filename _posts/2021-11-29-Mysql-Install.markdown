---
layout:     post
title:      "「原创」MySQL安装（简易版）"
subtitle:   "React versus Angular 2: There Will Be Blood"
date:       2021-11-29 12:00:00
author:     "NI"
header-img: "img/post-bg-re-vs-ng2.jpg"
header-mask: 0.3
catalog:    true
tags:
  - MySQL
  - 原创
---


#### 1.检查是否已经安装mysql相关软件

````
rpm -qa | grep mysql
rpm -qa | grep mariadb
````

删除相应的包

````
rpm -e --nodeps ...
````

#### 2.解压安装包

````
tar -xvf ...
````

####3.安装依赖包

````
rm -f /var/run/yum/pid
yum install -y glibc.i686
yum install -y libaio
````

#### 4.安装rpm

> common libs client server

````
rpm -ivh ...
````

#### 5.数据库初始化 & 查看日志获得密码
````
mysqld --initialize --user=mysql
cat /var/log/mysqld.log
````

#### 6.启动mysqld服务
````
systemctl start mysqld.service
````

#### 7.登录mysql并修改密码
````
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';
````

#### 8.管理mysql

重启/关闭/启动/状态

````
systemctl start/stop/restart/status
````

开机自启动 :

````
systemctl enable mysqld.serivce
systemctl disable mysqld.serivce
````

重新安装：删除以下目录后重新安装：

````
rm -rf /var/lib/mysql
````

#### 9.远程连接到mysql服务器

````
GRANT ALL PRIVILEGES ON *.* TO 'study'@'%' IDENTIFIED BY 'study' WITH GRANT OPTION;
FLUSH PRIVILEGES;
````

重启mysqld

#### 10.开放端口号

````
firewall-cmd --zone=public --add-port=3306/tcp --permanent
````

重启防火墙

````
systemctl restart firewalld.service
````