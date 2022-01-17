---
layout:     post
title:      "「原创」Linux指令 之 df"
subtitle:   "df详解与应用"
date:       2021-11-29 12:00:00
author:     "NI"
header-img: "img/post-bg-re-vs-ng2.jpg"
header-mask: 0.3
catalog:    true
tags:
  - Linux
  - df
  - 监控
---

#### 概述

df Disk Free

> linux中df命令的功能是用来检查linux服务器的文件系统的磁盘空间占用情况。可以利用该命令来获取硬盘被占用了多少空间，目前还剩下多少空间等信息。

#### 命令格式

<font color='pink'>df  [选项] [文件]</font>

#### 命令功能

> 显示指定磁盘文件的可用空间。如果没有文件名被指定，则所有当前被挂载的文件系统的可用空间将被显示。默认情况下，磁盘空间将以1KB为单位进行显示，除非环境变量POSIXLY_CORRECT被指定，那样将以512字节为单位进行显示。

#### 命令选项

+ -a或—all：包含全部的文件系统；
+ —block-size=<区块大小>：以指定的区块大小来显示区块数目；
+ -h或—human-readable：以可读性较高的方式来显示信息；
+ -H或—si：与-h参数相同，但在计算时是以1000 Bytes为换算单位而非1024 Bytes；
+ -i或—inodes：显示inode的信息；
+ -k或—kilobytes：指定区块大小为1024字节；
+ -l或—local：仅显示本地端的文件系统；
+ -m或—megabytes：指定区块大小为1048576字节；
+ —no-sync：在取得磁盘使用信息前，不要执行sync指令，此为预设值；
+ -P或—portability：使用POSIX的输出格式；
+ —sync：在取得磁盘使用信息前，先执行sync指令；
+ -t<文件系统类型>或—type=<文件系统类型>：仅显示指定文件系统类型的磁盘信息；
+ -T或—print-type：显示文件系统的类型；
+ -x<文件系统类型>或—exclude-type=<文件系统类型>：不要显示指定文件系统类型的磁盘信息；
+ —help：显示帮助；
+ —version：显示版本信息。

#### 查看系统磁盘设备，默认是KB为单位

**df**

![avatar](/img/linux/df.png)

> linux中df命令的输出清单的第1列是代表文件系统对应的设备文件的路径名（一般是硬盘上的分区）；第2列给出分区包含的数据块（1024字节）的数目；第3，4列分别表示已用的和可用的数据块数目。用户也许会感到奇怪的是，第3，4列块数之和不等于第2列中的块数。这是因为缺省的每个分区都留了少量空间供系统管理员使用。即使遇到普通用户空间已满的情况，管理员仍能登录和留有解决问题所需的工作空间。清单中Use% 列表示普通用户空间使用的百分比，即使这一数字达到100％，分区仍然留有系统管理员使用的空间。最后，Mounted on列表示文件系统的挂载点。

#### 以inode模式来显示磁盘使用情况

**df -i**

![avatar](/img/linux/df-i.png)

#### 显示指定类型磁盘

**df -t ext4**

![avatar](/img/linux/df-t.png)

#### 打印除ext4外所有的文件系统

**df -x ext4**

![avatar](/img/linux/df-x.png)

#### 列出个文件系统的i节点使用情况

**df -ia**

![avatar](/img/linux/df-ia.png)

#### 列出文件系统类型

**df -T**

![avatar](/img/linux/df-bigt.png)

#### 以更简读的方式显示目前磁盘空间和使用情况

**df -h**

![avatar](/img/linux/df-h.png)

**df -H**

![avatar](/img/linux/df-bigh.png)

**df -lh**

![avatar](/img/linux/df-lh.png)

**df -k**

![avatar](/img/linux/df-k.png)


#### 补充
+ -h更具目前磁盘空间和使用情况 以更易读的方式显示
+ -H根上面的-h参数相同,不过在根式化的时候,采用1000而不是1024进行容量转换
+ -k以单位显示磁盘的使用情况
+ -l显示本地的分区的磁盘空间使用率,如果服务器nfs了远程服务器的磁盘,那么在df上加上-l后系统显示的是过滤nsf驱动器后的结果
+ -i显示inode的使用情况。linux采用了类似指针的方式管理磁盘空间影射.这也是一个比较关键应用














