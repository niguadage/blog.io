---
layout: post
title: VxWorks培训总结
date: 2016-7-24
categories: blog
tags: [学习]
description: 顶着桑拿天学了几天，写个总结记录一下
---


#VxWorks培训
---
# 1 嵌入式实时操作系统介绍
实时操作系统：实时操作系统并不是要求快，是需要保证时间的可预料。

###VxWorks内核
多任务调度（采用基于优先级抢占方式，同时支持同优先级任务间的分时间片调度）

任务间的同步

进程间通信机制 

中断处理

定时器和内存管理机制

###嵌入式实时操作系统的选择
1. 高可靠性

2. 裁剪性（可裁剪到几百k）

3. 易用性 

4. 技术支持 （风河）

5. 支持的处理器类型（PPC，X86，MIPS，ARM，6.9开始支持64位X86）

6. 源代码（可以得到源代码）

7. 工具（workbench）

8. 标准兼容（兼容POSIX的PSE51/52，部分支持53和54）    

# 2  VxWorks

##  开发方式 
 * 上位机上开发，通过串口或者网线加载到Target上运行。
 * 使用wtregd这个进程下载到下位机中，5.x和6.x的不兼容，所以运行tornado或者workbench时需要先结束这个进程

###Target OS组成


###Application
---
###kernel和components


kernel不可拆分

*kernel对硬件不做假设，这一层隔离了应用和底层硬件。*

compones：WDB、TCP/IP、FileSystem、 I/O System

---
###硬件层

各种驱动

BSP

---

## 镜像文件

* 引导文件：bootrom
* 系统镜像：vxworks
* 系统库、第三方库：*.a（静态库）
* 应用程序：*.o、 *.out

##系统重启

    Ctr+X 热重启
*Vx没有软件关机，只能重新上电，并且没有电源管理，所以功耗方面和其他的系统差很多。*

## 帮助获取

* 去所在的docs文件夹查可能会快一些，直接搜慢的不行。。。*
* 

## 工程管理

* 工程常用的VIP工程，生成系统镜像
* DKM工程，生成应用程序，可以编译成库（驱动等）
* RTP工程（类似win下的应用程序，支持多进程）


##Host Shell 介绍

### Host Shell是一个运行在上位机上的Shell
* 可以同时运行多个
* 有C解释器
 * 函数调用
 * 变量声明
 * 四则运算

---

### 常用命令

    devs            查看设备
    iosFdShow       查看哪些IO设备被打开
    h               history，查看历史命令
    lkup            lookup（lkup “show”表示产看和show相关的API）
    d/m             读/写内存
    repeat          repeat（times，func，para），重复执行
    pwd             查看当前路径
    ld              加载程序ld(1, 0, a.out)
    taskSpown/sp    建立新任务（sp是taskSpown前四个参数使用默认，可以直接sp API）
    printLogo       打印Logo
    cd              change dir， cd “c:”
    ls/ll           list （和linux 一样）
    mkdir           新建文件夹
    ><              重定向（ld < file / i > file）
    
*所有命令直接运行都是对server上的shell，要操作target中，需要在命令前面加@*
    
### host shell的其他一些注意事项

host shell和target中的shell 是有区别的，是运行在win下的一个shell，可以使用target console，  
target console是将target中的IO强制重定向到shell中。  
host shell 中支持cmd和GDB。  
支持VI的命令。方向是hjkl，以及插入，删除，替换，追加等。

## 多任务

VX中的任务类似于线程。是最小的调度单元，由TCB和Stack构成。TCB存储系统相关控制信息，
CPU相关控制信息和任务的上下文。  
Stack用于存储局部变量和参数。  

###任务状态
* 就绪态
  * 执行态
* 阻塞态
* 延时态
* 挂起态
* 任务状态可以组合

###调度策略

* 基于优先级抢占（很好的保证实时性）
 * 不同工作优先级不同，CPU区别对待
 * 抢占基于优先级，相同时按先来先执行
 * ready队列中优先级最高的可以获得资源
 * 有kernel调用和终端发生时，重新进行调度


###时间片轮转（默认关闭的）

* 优先级相同可以相互抢占（基于时间片）
* 不同优先级任务可以发生抢占，高抢低

    kernelTimeSlice(ticks)   //打开时间片轮转
    
###禁止抢占
* taskLock()和taskUnlock()来关闭和使能调度器
* 可以防止任务切换，但是对中断不起作用


##创建任务

    int taskSpown(name, priority, options, stacksize, entryPt,arg1,...arg10)
* name 任务名称（通常t开头，可以同名）
* priority 0-255

 * taskPriorityGet(tid,&prio)
 * taskPrioritySet(tid,prio)
    
* stack 堆栈大小
* option 
 * VX_FP_TASK 支持浮点操作
* 返回值是taskID
* entryPt 函数入口

##删除任务
* taskDelete(ID)
 * 删除指定任务
 * 释放TCB和stack  
 *删除任务存在危险，慎用*

##任务重启、挂起、恢复、延时    
     taskRestart(ID)
     taskSuspend(ID)
     taskResume(ID)
     taskDelay(tick)  
##查看任务信息  
    i             类似任务管理器
    ti(tid)       查看stack 、option、CPU寄存器
    show          可以查看任何内核对象  
##错误状态  
* 高16位显示module，低16位标识error number
* printErrno可以显示具体信息


##任务通信  
* 共享内存
* 信号量
* 消息队列
* 管道

###信号量  
* 二进制信号量
* 互斥信号量
* 计数信号量


#### 二进制信号量

    SEM_ID semBCreate(int opt, STATE)
    //opt排队机制，SEM_Q_PRIORITY基于优先级，SEM_Q_FIFO基于fifo
    返回值为信号量ID或者NULL

#### 申请和释放
    semGive（SEM_ID）；
    semTake(sem_id,timeout)
    //timeout：WAIT_FOREVER、NO_WAIT、Ticks  
*take之后，信号量无效*  
*其他两类信号量类似，但是要注意死锁和优先级倒置的问题*  
 _中断中不能申请信号量_







