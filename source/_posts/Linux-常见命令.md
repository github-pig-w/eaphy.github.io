---
title: Linux 常见命令
tags:
  - linux
categories:
  - linux
date: 2019-06-28 01:20:27
---

# **uptime**
精简版显示系统负载均衡。`load average: 0.00, 0.09, 0.10`， 分别代表1分钟、5分钟、15分钟的负载均衡。

```bash
[root@localhost ~]# uptime
 13:54:10 up 9 min,  2 users,  load average: 0.00, 0.09, 0.10
```
<!-- more -->


# **top**
查看整机系统性能。

```bash
[root@localhost ~]# top
top - 13:54:04 up 9 min,  2 users,  load average: 0.00, 0.09, 0.10
Tasks: 448 total,   2 running, 446 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  3866948 total,  2971808 free,   483016 used,   412124 buff/cache
KiB Swap:  2097148 total,  2097148 free,        0 used.  3144672 avail Mem 

   PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                    
  2500 mysql     20   0 1182556 188432  10056 S   0.6  4.9   0:01.52 mysqld                                     
     1 root      20   0  189244   4232   2408 S   0.0  0.1   0:02.52 systemd                                    
     2 root      20   0       0      0      0 S   0.0  0.0   0:00.03 kthreadd                                   
     3 root      20   0       0      0      0 S   0.0  0.0   0:00.01 ksoftirqd/0                                
     5 root       0 -20       0      0      0 S   0.0  0.0   0:00.00 kworker/0:0H                               
     6 root      20   0       0      0      0 S   0.0  0.0   0:00.05 kworker/u256:0                             
     7 root      rt   0       0      0      0 S   0.0  0.0   0:00.07 migration/0                                
     8 root      20   0       0      0      0 S   0.0  0.0   0:00.00 rcu_bh                                     
     9 root      20   0       0      0      0 S   0.0  0.0   0:00.00 rcuob/0                                    
    10 root      20   0       0      0      0 S   0.0  0.0   0:00.00 rcuob/1                                    
    11 root      20   0       0      0      0 S   0.0  0.0   0:00.00 rcuob/2                                    
    12 root      20   0       0      0      0 S   0.0  0.0   0:00.00 rcuob/3                                    
    13 root      20   0       0      0      0 S   0.0  0.0   0:00.00 rcuob/4                                    
    14 root      20   0       0      0      0 S   0.0  0.0   0:00.00 rcuob/5                                    
    15 root      20   0       0      0      0 S   0.0  0.0   0:00.00 rcuob/6                                    
    16 root      20   0       0      0      0 S   0.0  0.0   0:00.00 rcuob/7                                    
    17 root      20   0       0      0      0 S   0.0  0.0   0:00.00 rcuob/8     
```

输入top 回车后，按数字 1 键可以显示多核cpu各核的使用情况。按字母 q 键退出 top

```bash
top - 13:57:30 up 12 min,  2 users,  load average: 0.00, 0.04, 0.08
Tasks: 448 total,   2 running, 446 sleeping,   0 stopped,   0 zombie
%Cpu0  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu2  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu3  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu4  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu5  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu6  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu7  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  3866948 total,  2972008 free,   482760 used,   412180 buff/cache
KiB Swap:  2097148 total,  2097148 free,        0 used.  3144884 avail Mem 


   PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                    
  2500 mysql     20   0 1182556 188432  10056 S   0.6  4.9   0:01.52 mysqld                                     
     1 root      20   0  189244   4232   2408 S   0.0  0.1   0:02.52 systemd                                    
     2 root      20   0       0      0      0 S   0.0  0.0   0:00.03 kthreadd                                   
     3 root      20   0       0      0      0 S   0.0  0.0   0:00.01 ksoftirqd/0                                
     5 root       0 -20       0      0      0 S   0.0  0.0   0:00.00 kworker/0:0H                               
     6 root      20   0       0      0      0 S   0.0  0.0   0:00.05 kworker/u256:0                             
     7 root      rt   0       0      0      0 S   0.0  0.0   0:00.07 migration/0                                
     8 root      20   0       0      0      0 S   0.0  0.0   0:00.00 rcu_bh                                     
     9 root      20   0       0      0      0 S   0.0  0.0   0:00.00 rcuob/0                                    
    10 root      20   0       0      0      0 S   0.0  0.0   0:00.00 rcuob/1                                    
    11 root      20   0       0      0      0 S   0.0  0.0   0:00.00 rcuob/2                                    
    12 root      20   0       0      0      0 S   0.0  0.0   0:00.00 rcuob/3                                    
    13 root      20   0       0      0      0 S   0.0  0.0   0:00.00 rcuob/4                                    
    14 root      20   0       0      0      0 S   0.0  0.0   0:00.00 rcuob/5                                    
    15 root      20   0       0      0      0 S   0.0  0.0   0:00.00 rcuob/6                                    
    16 root      20   0       0      0      0 S   0.0  0.0   0:00.00 rcuob/7                                    
    17 root      20   0       0      0      0 S   0.0  0.0   0:00.00 rcuob/8  
```



# free
查看系统内存。 -m 以兆为单位显示，-g 以G为单位显示
```bash
[root@localhost ~]# free
              total        used        free      shared  buff/cache   available
Mem:        3866948      482152     2972304        9328      412492     3145432
Swap:       2097148           0     2097148
[root@localhost ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           3776         471        2902           9         402        3071
Swap:          2047           0        2047
[root@localhost ~]# free -g
              total        used        free      shared  buff/cache   available
Mem:              3           0           2           0           0           2
Swap:             1           0    
```
# df
文件系统的磁盘空间占用情况 。  -h 表示以常用单位显示 。

```bash
[root@localhost ~]# df
文件系统                   1K-块    已用    可用 已用% 挂载点
/dev/mapper/centos-root 18307072 9416900 8890172   52% /
devtmpfs                 1918188       0 1918188    0% /dev
tmpfs                    1933472      84 1933388    1% /dev/shm
tmpfs                    1933472    9224 1924248    1% /run
tmpfs                    1933472       0 1933472    0% /sys/fs/cgroup
/dev/sda1                 508588  160364  348224   32% /boot
tmpfs                     386696       0  386696    0% /run/user/0
tmpfs                     386696      16  386680    1% /run/user/42
[root@localhost ~]# df -h
文件系统                 容量  已用  可用 已用% 挂载点
/dev/mapper/centos-root   18G  9.0G  8.5G   52% /
devtmpfs                 1.9G     0  1.9G    0% /dev
tmpfs                    1.9G   84K  1.9G    1% /dev/shm
tmpfs                    1.9G  9.1M  1.9G    1% /run
tmpfs                    1.9G     0  1.9G    0% /sys/fs/cgroup
/dev/sda1                497M  157M  341M   32% /boot
tmpfs                    378M     0  378M    0% /run/user/0
tmpfs                    378M   16K  378M    1% /run/user/42

```
# vmstat
显示虚拟内存状态（“Viryual Memor Statics”），它可以显示进程、内存、I/O等系统整体运行状态。

```bash
[root@localhost ~]# vmstat
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 2972176   1260 411324    0    0    25     2   23   30  0  0 99  1  0
 
[root@localhost ~]# vmstat -n 2 3    // 2 秒刷新一次，共 3 次
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 2972036   1260 411324    0    0    25     2   23   30  0  0 99  1  0
 0  0      0 2972160   1260 411324    0    0     0     0   71  125  0  0 100  0  0
 0  0      0 2972160   1260 411324    0    0     0     0   64  109  0  0 100  0  0
```
字段解释：

 - r: 运行队列中进程数量，这个值也可以判断是否需要增加CPU。（长期大于1） 
 - b: 等待IO的进程数量。 
 - swpd:使用虚拟内存大小，如果swpd的值不为0，但是SI，SO的值长期为0，这种情况不会影响系统性能。 
 - free: 空闲物理内存大小。
 - id: 空闲时间百分比

# mpstat
查看所有 cpu 核信息。

```bash
[root@localhost ~]# mpstat  // 输出为从系统启动以来的平均值。
Linux 3.10.0-327.el7.x86_64 (localhost.localdomain) 	2019年06月29日 	_x86_64_	(8 CPU)

14时23分28秒  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
14时23分28秒  all    0.02    0.00    0.14    0.38    0.00    0.00    0.00    0.00    0.00   99.45

[root@localhost ~]# mpstat -P ALL 2 3 // 查看所有cpu核信息，2 秒刷新一次，共 3 次
Linux 3.10.0-327.el7.x86_64 (localhost.localdomain) 	2019年06月29日 	_x86_64_	(8 CPU)

14时31分16秒  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
14时31分18秒  all    0.00    0.00    0.06    0.00    0.00    0.00    0.00    0.00    0.00   99.94
14时31分18秒    0    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
14时31分18秒    1    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
14时31分18秒    2    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
14时31分18秒    3    0.00    0.00    0.50    0.00    0.00    0.00    0.00    0.00    0.00   99.50
14时31分18秒    4    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
14时31分18秒    5    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
14时31分18秒    6    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
14时31分18秒    7    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00

14时31分18秒  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
14时31分20秒  all    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
14时31分20秒    0    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
14时31分20秒    1    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
14时31分20秒    2    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
14时31分20秒    3    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
14时31分20秒    4    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
14时31分20秒    5    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
14时31分20秒    6    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
14时31分20秒    7    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00

14时31分20秒  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
14时31分22秒  all    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
14时31分22秒    0    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
14时31分22秒    1    0.00    0.00    0.49    0.00    0.00    0.00    0.00    0.00    0.00   99.51
14时31分22秒    2    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
14时31分22秒    3    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
14时31分22秒    4    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
14时31分22秒    5    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
14时31分22秒    6    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
14时31分22秒    7    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00

平均时间:  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
平均时间:  all    0.00    0.00    0.02    0.00    0.00    0.00    0.00    0.00    0.00   99.98
平均时间:    0    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
平均时间:    1    0.00    0.00    0.17    0.00    0.00    0.00    0.00    0.00    0.00   99.83
平均时间:    2    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
平均时间:    3    0.00    0.00    0.17    0.00    0.00    0.00    0.00    0.00    0.00   99.83
平均时间:    4    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
平均时间:    5    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
平均时间:    6    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
平均时间:    7    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00

```
# pidstat
用于监控全部或指定进程占用系统资源的情况，如CPU，内存、设备IO、任务切换、线程等。

```bash
[root@localhost ~]# pidstat		//查看全部进程cpu使用信息
Linux 3.10.0-327.el7.x86_64 (localhost.localdomain) 	2019年06月29日 	_x86_64_	(8 CPU)

14时33分54秒   UID       PID    %usr %system  %guest    %CPU   CPU  Command
14时33分54秒     0         1    0.00    0.11    0.00    0.11     0  systemd
14时33分54秒     0         2    0.00    0.00    0.00    0.00     6  kthreadd
14时33分54秒     0         3    0.00    0.00    0.00    0.00     0  ksoftirqd/0
14时33分54秒     0         6    0.00    0.00    0.00    0.00     3  kworker/u256:0
14时33分54秒     0         7    0.00    0.00    0.00    0.00     0  migration/0
14时33分54秒     0       137    0.00    0.04    0.00    0.04     3  rcu_sched
14时33分54秒     0       138    0.00    0.01    0.00    0.01     2  rcuos/0

[root@localhost ~]# pidstat -U 1 5 -p 2977   //查看 2977 进程cpu使用信息，1 秒一次，采集 5 次
Linux 3.10.0-327.el7.x86_64 (localhost.localdomain) 	2019年06月29日 	_x86_64_	(8 CPU)

14时36分50秒     USER       PID    %usr %system  %guest    %CPU   CPU  Command
14时36分51秒      gdm      2977    0.00    0.00    0.00    0.00     7  goa-identity-se
14时36分52秒      gdm      2977    0.00    0.00    0.00    0.00     7  goa-identity-se
14时36分53秒      gdm      2977    0.00    0.00    0.00    0.00     7  goa-identity-se
14时36分54秒      gdm      2977    0.00    0.00    0.00    0.00     7  goa-identity-se
14时36分55秒      gdm      2977    0.00    0.00    0.00    0.00     7  goa-identity-se
平均时间:      gdm      2977    0.00    0.00    0.00    0.00     -  goa-identity-se

[root@localhost ~]# pidstat -r 1 5 -p 2977   //查看 2977 进程内存的统计信息，1 秒一次，采集 5 次
[root@localhost ~]# pidstat -d 1 5 -p 2977   //查看 2977 进程 IO的统计信息，1 秒一次，采集 5 次
```

# iostat
查看磁盘IO信息。
```bash
[root@localhost ~]# iostat
Linux 3.10.0-327.el7.x86_64 (localhost.localdomain) 	2019年06月29日 	_x86_64_	(8 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.02    0.00    0.11    0.26    0.00   99.61

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
sda               4.32       106.80        10.54     366872      36194
scd0              0.00         0.01         0.00         44          0
dm-0              3.67        97.95         9.93     336460      34125
dm-1              0.04         0.37         0.00       1268          0

```
# ifstat
查看网络IO信息。

```bash
[root@localhost ~]# ifstat
#kernel
Interface        RX Pkts/Rate    TX Pkts/Rate    RX Data/Rate    TX Data/Rate  
                 RX Errs/Drop    TX Errs/Drop    RX Over/Rate    TX Coll/Rate  
lo                     4 0             4 0           340 0           340 0      
                       0 0             0 0             0 0             0 0      
eno16777736         1478 0          1307 0        118781 0        945129 0      
                       0 0             0 0             0 0             0 0      
virbr0                 0 0             0 0             0 0             0 0      
                       0 0             0 0             0 0             0 0      
```

# awk
