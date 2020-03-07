+++
title = "进程信息跟踪：top-jps-jstack"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "US"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/0c784a89c427)

万事多用Man 

### Top

[Top命令含义](https://blog.csdn.net/liubin1991liubin/article/details/79702640)

[What are us, sy, ni, id, wa, hi, si and st (for CPU usage)?](https://unix.stackexchange.com/questions/18918/linux-top-command-what-are-us-sy-ni-id-wa-hi-si-and-st-for-cpu-usage)

>us: user cpu time (or) % CPU time spent in user space
sy: system cpu time (or) % CPU time spent in kernel space
ni: user nice cpu time (or) % CPU time spent on low priority processes
id: idle cpu time (or) % CPU time spent idle
wa: io wait cpu time (or) % CPU time spent in wait (on disk)
hi: hardware irq (or) % CPU time spent servicing/handling hardware interrupts
si: software irq (or) % CPU time spent servicing/handling software interrupts
st: steal time - - % CPU time in involuntary wait by virtual cpu while hypervisor is servicing another processor (or) % CPU time stolen from a virtual machine

```
top - 14:54:28 up 13 days, 13 min,  0 users,  load average: 19.41, 13.56, 9.74
Tasks:   5 total,   1 running,   4 sleeping,   0 stopped,   0 zombie
%Cpu(s): 46.2 us,  5.0 sy,  0.4 ni, 47.9 id,  0.0 wa,  0.0 hi,  0.5 si,  0.0 st
KiB Mem : 15625000 total,  2836444 free,  8893532 used,  3895024 buff/cache
KiB Swap:        0 total,        0 free,        0 used.  2836444 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                                                                                                       
25618 root      20   0 8828860 4.112g  19816 S 803.0 27.6  23:20.23 java                                                                                                                                                          
                                                                                                                                                                                                                                  
                                                            
    1 root      20   0 10.955g 4.169g  13880 S   0.3 28.0 369:30.07 java                                                                                                                                                          
                                                                                                                                                                                                                                  
                                                            
20942 root      20   0   13380   1872   1436 S   0.0  0.0   0:00.02 sh                                                                                                                                                            
```

`top - 14:54:28 up 13 days, 13 min,  0 users,  load average: 19.41, 13.56, 9.74`
>第一块：当前系统时间
第二块：系统已经运行了13天13分
第三块：当前有0个用户登录系统(docker中启动的会有这种情况？)
第四块：load average后面的三个数分别是1分钟、5分钟、15分钟的负载情况。load average数据是每隔5秒钟检查一次活跃的进程数，然后按特定算法计算出的数值。如果这个数除以逻辑CPU的数量，结果高于5的时候就表明系统在超负荷运转

`Tasks:   5 total,   1 running,   4 sleeping,   0 stopped,   0 zombie`
>第一块：系统现在有5个进程
第二块：有一个进程处于运行状态
第三块：4个进程处于休眠状态
第四块：0个进程处于stopped状态
第五块：0个进程处于zombie状态

`%Cpu(s): 46.2 us,  5.0 sy,  0.4 ni, 47.9 id,  0.0 wa,  0.0 hi,  0.5 si,  0.0 st`
>46.2 us — 用户空间占用CPU的百分比。user space
5 sy — 内核空间占用CPU的百分比。 system
0.4 ni — 改变过优先级的进程占用CPU的百分比
47.9 id — 空闲CPU百分比
0.0 wa — IO等待占用CPU的百分比
0.0 hi — 硬中断（Hardware IRQ）占用CPU的百分比
0.5 si — 软中断（Software Interrupts）占用CPU的百分比
用户空间就是用户进程所在的内存区域，相对的，系统空间就是操作系统占据的内存区域。用户进程和系统进程的所有数据都在内存中。

`KiB Mem : 15625000 total,  2836444 free,  8893532 used,  3895024 buff/cache`
>15625000 total — 物理内存总量（16GB）
8893532 used — 使用中的内存总量（8.9GB）
2836444 free — 空闲内存总量（2.8G）
3895024 buffers — 缓存的内存量 （3.6G）

`KiB Swap:        0 total,        0 free,        0 used.  2836444 avail Mem `
>Swap分区：swap（类似于虚拟内存）交换分区的used，如果这个数值在不断的变化，说明内核在不断进行内存和swap的数据交换，这是可以判定服务器的内存不够用了。

看起来docker启动的没有分配任何的swap分区

`PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND   `

PID — 进程id
USER — 进程所有者
PR — 进程优先级
NI — nice值。负值表示高优先级，正值表示低优先级
VIRT — 进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
RES — 进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
SHR — 共享内存大小，单位kb
S — 进程状态。D=不可中断的睡眠状态 R=运行 S=睡眠 T=跟踪/停止 Z=僵尸进程
%CPU — 上次更新到现在的CPU时间占用百分比
%MEM — 进程使用的物理内存百分比
TIME+ — 进程使用的CPU时间总计，单位1/100秒
COMMAND — 进程名称（命令名/命令行）

`top -Hp pid` 列出进程pid所有的线程信息 类似于 `ps -T -p pid`

`ps -mp pid -o THREAD,tid,time` 
通过%CPU和 TIME，判断占用的线程TID 

-m Show threads after processes.
-p Select by PID
-o format. 自定义输出格式，例如 THREAD,tid,time

---

### JPS

jps - Lists the instrumented Java Virtual Machines (JVMs) on the target system. This command is experimental and unsupported.

### Jstack

`jstack pid > pid.txt ` 查看具体的信息

