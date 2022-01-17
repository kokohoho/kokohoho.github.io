---
layout:     post
title:      "「原创」Linux指令 之 netstat"
subtitle:   "netstat详解与应用"
date:       2021-11-29 12:00:00
author:     "NI"
header-img: "img/post-bg-re-vs-ng2.jpg"
header-mask: 0.3
catalog:    true
tags:
  - Linux
  - netstat
  - 监控
---

### 一、概述

top命令提供运行系统的动态实时视图,执行top命令可得如下图：

![avatar](/img/linux/top.png)

top命令主要包括两个显示区域

**1.汇总区**

**2.任务区**

和一个输入行：

**3.在汇总区和任务区字段行的中间**

### 二、汇总区

**第1行** ：系统当前时间 up 系统运行时间、当前用户数、平均负载。

使用uptime指令可得到相同的结果（是否可将uptime用于系统监控？）：

```shell
top - 19:47:54 up 67 days,  8:53,  1 user,  load average: 0.04, 0.04, 0.05
```

**第2行** ： 进程总数total、运行进程数running、休眠进程数sleeping、停止进程数stopped、僵尸进程数zombie

```shell
Tasks:  80 total,   1 running,  79 sleeping,   0 stopped,   0 zombie
```

**第3行** ：不同模式下所占cpu百分比

**us,user** : 运行（未调整优先级的）用户进程的CPU时间

**sy,system** : 运行内核进程的CPU时间

**ni,niced** : 运行已调整优先级的用户进程的CPU时间

**wa,IO wait** : 用于等待IO完成的CPU时间

**hi** : 处理硬件中断的CPU时间

**si** : 处理软件中断的CPU时间

**st** : 这个虚拟机被宿主机偷去的CPU时间

```shell
%Cpu(s):  0.8 us,  0.8 sy,  0.0 ni, 98.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
```

**第4、5行：**

第4行物理内存使用：全部可用内存、已使用内存、空闲内存、缓冲内存

第5行虚拟内存使用：全部、已使用、空闲、缓冲交换空间

```shell
KiB Mem :   952084 total,    93920 free,   275052 used,   583112 buff/cache
KiB Swap:        0 total,        0 free,        0 used.   514244 avail Mem
```

### 三、任务区

**第1行：** 字段头

```shell
ID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND 
```

**第2-n行：** 内容行

```shell
ID    USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND 
1420  root      10 -10  130852  12660   6892 S   3.7  1.3   2657:04 AliYunDun              
32045 admin     20   0  162004   2248   1576 R   0.3  0.2   0:00.60 top                    
    1 root      20   0   51716   3580   2272 S   0.0  0.4  13:52.23 systemd                
    2 root      20   0       0      0      0 S   0.0  0.0   0:00.21 kthreadd 
```

>**ID** ：进程ID

>**USER** ：进程启动用户

>**PR** ：priority，进程优先级，<font color='red'>0-99显示为rt</font>（代表实时优先级），<font color='red'>100-139时PR值为</font>（静态优先级-100）

>**NI** ：nice value，<font color='red'>nice值越低，抢cpu越凶</font>

>**VIRT** ：Virtual Memory Size (KiB)，进程使用的虚拟内存总量，<font color='red'>可以理解成进程向内核申请占用的虚拟内存的总量</font>。但是，虚拟内存申请了不一定真正会用到，<font color='red'>所以虚拟内存并不是进程真正占用的内存</font>。像java进程在linux上运行时，虚拟内存就占用非常大，但是实际上进程真正用到的比较少。

>**RES** ：Resident Memory Size (KiB)，与VIRT正好相反，<font color='red'>RES是进程实打实占用的物理内存总量</font>（包括共享内存），不包括swap out的内存。所以，<font color='red'>要看进程真实占用的内存就看RES</font>

>**SHR** ：Shared Memory Size (KiB)，与其他进程共享的内存，比如链结的动态库等。<font color='red'>进程本身占用的物理内存量 = RES - SHR</font>

>**S** ：进程状态,D-不可中断睡眠态,R-运行态,S-睡眠态,T-被跟踪或已停止,Z-僵尸态

>**%CPU** ：CPU使用百分比

>**%MEM** ：内存使用百分比，RES/物理内存总量*100%

>**TIME+COMMAND** ：任务启动后到现在所使用的全部CPU时间，精确到10ms；运行进程所使用的命令


### 四、交互命令 

**h或?** ： 显示帮助

![avatar](/img/linux/top-h.png)

**q** ：   退出交互回到top界面

![avatar](/img/linux/top-q.png)

**'<ENTER>' '<SPACE>'** ：刷新显示

**1** ：监控每个逻辑CPU的状况

![avatar](/img/linux/top-1.png)

**z** ：彩色

![avatar](/img/linux/top-z.png)

**'<'与'>'** ：向前与向后

**m** ：改变内存显示样式

![avatar](/img/linux/top-m.png)

**l** ：显示所有CPU负载

**P** ：按CPU使用排序

![avatar](/img/linux/top-P.png)

**N** ：以PID大小排序

![avatar](/img/linux/top-N.png)

**R** ：对排序进行反转

![avatar](/img/linux/top-R.png)

**s** ：改变画面更新频率

![avatar](/img/linux/top-s.png)

未完，待补充......

### 五、使用场景

总方向：尝试找出你的机器正在运行什么程序，以及哪个进程耗尽了内存导致系统非常非常慢，这是 top 命令所能胜任的工作。

### 六、案例

**基本参数**：

默认情况下，top命令是3秒刷新一次

| 参数 | 说明 |
| ---- | ---- |
| -d | 指定刷新频率 |
| -p | 查看指定进程的信息 |
| -u | 查看指定用户的进程 |
| -n | 查看指定top次数的信息 |
| -b | 批次档模式，搭配 "n" 参数一起使用，可以用来将 top 的结果输出到档案内 |


**案例1** ：1秒刷新一次

>top -d 1

**案例2** ：查看指定用户nginx进程

>top -d 1 -u nginx

**案例3** ：将两次top信息写入到文件

>top -b -n 2 > top.txt

**案例4** ：查看PID为1352的信息

>top -p 1352























