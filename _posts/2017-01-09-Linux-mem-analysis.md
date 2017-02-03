---
layout: post
title:  "Linux 内存占用分析"
date: 2017-01-09
categories: Linux,Java
tags: Java,Linux
published: true
---
* 目录
{:toc}

### free查看Linux系统内存占用情况

```bash
free -m
             total       used       free     shared    buffers     cached
Mem:         15949      15562        386          0        152       1195
-/+ buffers/cache:      14214       1734
Swap:         4094          0       4094
```

free指令结果解释
![image]({{site.url}}/assets/2017/01/free_command.jpg)

### jmap查看系统Java进程内存占用情况

```plain
jmap -heap 3974
Attaching to process ID 3974, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.45-b02

using thread-local object allocation.
Parallel GC with 4 thread(s)

Heap Configuration:
  MinHeapFreeRatio         = 0
  MaxHeapFreeRatio         = 100
  MaxHeapSize              = 4181721088 (3988.0MB)
  NewSize                  = 87031808 (83.0MB)
  MaxNewSize               = 1393557504 (1329.0MB)
  OldSize                  = 175112192 (167.0MB)
  NewRatio                 = 2
  SurvivorRatio            = 8
  MetaspaceSize            = 21807104 (20.796875MB)
  CompressedClassSpaceSize = 1073741824 (1024.0MB)
  MaxMetaspaceSize         = 17592186044415 MB
  G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
  capacity = 31981568 (30.5MB)
  used     = 6378992 (6.0834808349609375MB)
  free     = 25602576 (24.416519165039062MB)
  19.945838803150615% used
From Space:
  capacity = 524288 (0.5MB)
  used     = 517528 (0.49355316162109375MB)
  free     = 6760 (0.00644683837890625MB)
  98.71063232421875% used
To Space:
  capacity = 524288 (0.5MB)
  used     = 0 (0.0MB)
  free     = 524288 (0.5MB)
  0.0% used
PS Old Generation
  capacity = 175112192 (167.0MB)
  used     = 11148392 (10.631935119628906MB)
  free     = 163963800 (156.3680648803711MB)
  6.366428215346651% used

7416 interned Strings occupying 627728 bytes.
```

查看Java进程的内存各区占用情况

```plain
jstat -gc 3974
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
512.0  512.0   0.0   192.0  31232.0   7554.7   171008.0   10887.1   20864.0 20187.0 2432.0 2289.0   1845    3.040   0      0.000    3.040
```


```plain
-gc Option
Garbage-collected heap statistics
Column	Description
S0C	Current survivor space 0 capacity (KB).
S1C	Current survivor space 1 capacity (KB).
S0U	Survivor space 0 utilization (KB).
S1U	Survivor space 1 utilization (KB).
EC	Current eden space capacity (KB).
EU	Eden space utilization (KB).
OC	Current old space capacity (KB).
OU	Old space utilization (KB).
PC	Current permanent space capacity (KB).
PU	Permanent space utilization (KB).
YGC	Number of young generation GC Events.
YGCT	Young generation garbage collection time.
FGC	Number of full GC events.
FGCT	Full garbage collection time.
GCT	Total garbage collection time.
```

![image]({{site.url}}/assets/2017/01/ES-mem.png)

### 查看系统缓存占用大小

```bash
cat /proc/slabinfo |awk '{if($3*$4/1024/1024 > 100){print $1,$3*$4/1024/1024} }'
dentry 3776.57
```

![image]({{site.url}}/assets/2017/01/ES-mem-dentry.png)

```bash
cat /proc/meminfo
MemTotal:       16332272 kB
MemFree:          487012 kB
Buffers:          155428 kB
Cached:          1170668 kB
SwapCached:            0 kB
Active:          9334704 kB
Inactive:        2109700 kB
Active(anon):    8552520 kB
Inactive(anon):  1566548 kB
Active(file):     782184 kB
Inactive(file):   543152 kB
Unevictable:           0 kB
Mlocked:               0 kB
SwapTotal:       4192924 kB
SwapFree:        4192924 kB
Dirty:             31600 kB
Writeback:             0 kB
AnonPages:      10118764 kB
Mapped:            69284 kB
Shmem:               320 kB
Slab:            4254256 kB
SReclaimable:    4225984 kB
SUnreclaim:        28272 kB
KernelStack:        2648 kB
PageTables:        28624 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:    12359060 kB
Committed_AS:   10378928 kB
VmallocTotal:   34359738367 kB
VmallocUsed:       41304 kB
VmallocChunk:   34359694216 kB
HardwareCorrupted:     0 kB
AnonHugePages:   9912320 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
DirectMap4k:        8184 kB
DirectMap2M:    16769024 kB
```

查询Linux实际物理内存占用

```bash
#/bin/bash
for PROC in `ls  /proc/|grep "^[0-9]"`
do
  if [ -f /proc/$PROC/statm ]; then
      TEP=`cat /proc/$PROC/statm | awk '{print ($2)}'`
      RSS=`expr $RSS + $TEP`
  fi
done
RSS=`expr $RSS \* 4`
echo $RSS"KB"

10259564KB
```

查询slabinfo大小

> 简单的说内核为了高性能每个需要重复使用的对象都会有个池，这个slab池会cache大量常用的对象，所以会消耗大量的内存。

![image]({{site.url}}/assets/2017/01/SLABTOP.png)

```bash
echo `cat /proc/slabinfo |awk 'BEGIN{sum=0;}{sum=sum+$3*$4;}END{print sum/1024/1024}'` MB
3948.44 MB
904.256 MB
```
查询Pagetables大小

```bash
echo `grep PageTables /proc/meminfo | awk '{print $2}'` KB
29124 KB
```
> 内存的去向主要有3个：1. 进程消耗。 2. slab消耗 3.pagetable消耗。


### Elasticsearch服务占用内存过大原因分析

![image]({{site.url}}/assets/2017/01/es-stat.png)

```plain
8651  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/4/index/_jqkw_Lucene41_0.tim", 0x7f0b566eb100) = -1 ENOENT (No such file or directory)
8651  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/4/index/_jqkw_Lucene41_0.tip", 0x7f0b566eb100) = -1 ENOENT (No such file or directory)
8651  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/4/index/_jqkw.fnm", 0x7f0b566eb2e0) = -1 ENOENT (No such file or directory)
8651  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/4/index/_jqkw.cfs", 0x7f0b566eb120) = -1 ENOENT (No such file or directory)
8651  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/4/index/_jqkw.cfe", 0x7f0b566eb170) = -1 ENOENT (No such file or directory)
8651  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/4/index/_jqkw.si", 0x7f0b566eb2d0) = -1 ENOENT (No such file or directory)
8651  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/4/index/_jqkw_1.del", 0x7f0b566eb3b0) = -1 ENOENT (No such file or directory)
8759  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/3/index/_jqhz.fdx", 0x7f0b5134e3c0) = -1 ENOENT (No such file or directory)
8759  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/3/index/_jqhz.fdt", 0x7f0b5134e3c0) = -1 ENOENT (No such file or directory)
8805  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/3/index/_jqhz.nvd", 0x7f0b941f1350) = -1 ENOENT (No such file or directory)
8805  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/3/index/_jqhz.nvm", 0x7f0b941f1350) = -1 ENOENT (No such file or directory)
8805  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/3/index/_jqhz_Lucene410_0.dvd", 0x7f0b941f1200) = -1 ENOENT (No such file or directory)
8805  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/3/index/_jqhz_Lucene410_0.dvm", 0x7f0b941f1200) = -1 ENOENT (No such file or directory)
8805  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/3/index/_jqhz_Lucene41_0.doc", 0x7f0b941f10e0) = -1 ENOENT (No such file or directory)
8805  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/3/index/_jqhz_Lucene41_0.pos", 0x7f0b941f10e0) = -1 ENOENT (No such file or directory)
8805  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/3/index/_jqhz_Lucene41_0.tim", 0x7f0b941f10d0) = -1 ENOENT (No such file or directory)
8805  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/3/index/_jqhz_Lucene41_0.tip", 0x7f0b941f10d0) = -1 ENOENT (No such file or directory)
8805  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/3/index/_jqhz.fnm", 0x7f0b941f12b0) = -1 ENOENT (No such file or directory)
8805  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/3/index/_jqhz.cfs", 0x7f0b941f10f0) = -1 ENOENT (No such file or directory)
8805  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/3/index/_jqhz.cfe", 0x7f0b941f1140) = -1 ENOENT (No such file or directory)
8805  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/3/index/_jqhz.si", 0x7f0b941f12a0) = -1 ENOENT (No such file or directory)
8805  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/3/index/_jqhz_1.del", 0x7f0b941f1380) = -1 ENOENT (No such file or directory)
8766  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/4/index/_jqkx.fdx", 0x7f0b4f1ad3c0) = -1 ENOENT (No such file or directory)
8766  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/4/index/_jqkx.fdt", 0x7f0b4f1ad3c0) = -1 ENOENT (No such file or directory)
8651  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/4/index/_jqkx.nvd", 0x7f0b566eb380) = -1 ENOENT (No such file or directory)
8651  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/4/index/_jqkx.nvm", 0x7f0b566eb380) = -1 ENOENT (No such file or directory)
8651  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/4/index/_jqkx_Lucene410_0.dvd", 0x7f0b566eb230) = -1 ENOENT (No such file or directory)
8651  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/4/index/_jqkx_Lucene410_0.dvm", 0x7f0b566eb230) = -1 ENOENT (No such file or directory)
8651  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/4/index/_jqkx_Lucene41_0.doc", 0x7f0b566eb110) = -1 ENOENT (No such file or directory)
8651  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/4/index/_jqkx_Lucene41_0.pos", 0x7f0b566eb110) = -1 ENOENT (No such file or directory)
8651  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/4/index/_jqkx_Lucene41_0.tim", 0x7f0b566eb100) = -1 ENOENT (No such file or directory)
8651  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/4/index/_jqkx_Lucene41_0.tip", 0x7f0b566eb100) = -1 ENOENT (No such file or directory)
8651  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/4/index/_jqkx.fnm", 0x7f0b566eb2e0) = -1 ENOENT (No such file or directory)
8651  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/4/index/_jqkx.cfs", 0x7f0b566eb120) = -1 ENOENT (No such file or directory)
8651  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/4/index/_jqkx.cfe", 0x7f0b566eb170) = -1 ENOENT (No such file or directory)
8651  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/4/index/_jqkx.si", 0x7f0b566eb2d0) = -1 ENOENT (No such file or directory)
8651  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/4/index/_jqkx_1.del", 0x7f0b566eb3b0) = -1 ENOENT (No such file or directory)
8723  lstat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/_state/global-21897.st.tmp", 0x7f0b947f3510) = -1 ENOENT (No such file or directory)
8759  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/0/index/_jv5t.fdx", 0x7f0b5134e3c0) = -1 ENOENT (No such file or directory)
8759  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/0/index/_jv5t.fdt", 0x7f0b5134e3c0) = -1 ENOENT (No such file or directory)
8723  lstat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/_state/global-21898.st.tmp", 0x7f0b947f3510) = -1 ENOENT (No such file or directory)
8805  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/0/index/_jv5t.nvd", 0x7f0b941f1350) = -1 ENOENT (No such file or directory)
8805  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/0/index/_jv5t.nvm", 0x7f0b941f1350) = -1 ENOENT (No such file or directory)
8805  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/0/index/_jv5t_Lucene410_0.dvd", 0x7f0b941f1200) = -1 ENOENT (No such file or directory)
8805  stat("/data/elasticsearch/xxx-production-us-west-1/nodes/0/indices/playground_profile/0/index/_jv5t_Lucene410_0.dvm", 0x7f0b941f1200) = -1 ENOENT (No such file or directory)
```
> 当应用程序发起stat系统调用时，就会创建对应的dentry_cache项。如果每次stat的文件都是不存在的文件，那么总是会有大量新的dentry_cache项被创建


### 参考

- [memory-usage-of-the-machine-with-es-is-continuously-increasing](https://discuss.elastic.co/t/memory-usage-of-the-machine-with-es-is-continuously-increasing/23537/13)

- [Linux Used内存到底哪里去了？](http://blog.yufeng.info/archives/2456)
