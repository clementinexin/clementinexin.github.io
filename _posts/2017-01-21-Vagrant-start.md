---
layout: post
title:  "虚拟机 - Vagrant、Qemu、KVM、XEN"
date: 2017-01-21
categories: Linux
tags: Qemu、KVM、XEN、Vagrant
published: true
---
* 目录
{:toc}

## 一、Vargrant上手


### 1.启动Vargrant遇到的问题

```plain
Vagrant was unable to mount VirtualBox shared folders. This is usually
because the filesystem "vboxsf" is not available. This filesystem is
made available via the VirtualBox Guest Additions and kernel module.
Please verify that these guest additions are properly installed in the
guest. This is not a bug in Vagrant and is usually caused by a faulty
Vagrant box. For context, the command attempted was:

mount -t vboxsf -o uid=1000,gid=1000 vagrant /vagrant

The error output from the command was:
```

解决办法

```plain
vagrant plugin install vagrant-vbguest
Installing the 'vagrant-vbguest' plugin. This can take a few minutes...
Fetching: micromachine-2.0.0.gem (100%)
Fetching: vagrant-vbguest-0.13.0.gem (100%)
Installed the plugin 'vagrant-vbguest (0.13.0)'!
```

```plain
vagrant up
/opt/vagrant/embedded/gems/gems/vagrant-1.9.1/lib/vagrant/util/platform.rb:24: warning: Insecure world writable dir /opt in PATH, mode 040777
Bringing machine 'default' up with 'virtualbox' provider...
```

Vagrant ssh连接到虚拟机

```plain
vagrant ssh
/opt/vagrant/embedded/gems/gems/vagrant-1.9.1/lib/vagrant/util/platform.rb:24: warning: Insecure world writable dir /opt in PATH, mode 040777
Welcome to Ubuntu 15.04 (GNU/Linux 3.19.0-15-generic x86_64)

 * Documentation:  https://help.ubuntu.com/
Your Ubuntu release is not supported anymore.
For upgrade information, please visit:
http://www.ubuntu.com/releaseendoflife

New release '15.10' available.
Run 'do-release-upgrade' to upgrade to it.

vagrant@vagrant-ubuntu-trusty:~$
```


### 2.启动Windows镜像遇到的问题

```plain
The "metadata.json" file for the box 'XP' was not found.
Boxes require this file in order for Vagrant to determine the
provider it was made for. If you made the box, please add a
"metadata.json" file to it. If someone else made the box, please
notify the box creator that the box is corrupt. Documentation for
box file format can be found at the URL below:

https://www.vagrantup.com/docs/boxes/format.html
```

解决办法

```plain
vagrant box add XP IE8\ -\ WinXP.box
/opt/vagrant/embedded/gems/gems/vagrant-1.9.1/lib/vagrant/util/platform.rb:24: warning: Insecure world writable dir /opt in PATH, mode 040777
==> box: Box file was not detected as metadata. Adding it directly...
==> box: Adding box 'XP' (v0) for provider:
    box: Unpacking necessary files from: file:///Users/daijiajia/exercise/vagrant/windows/xp/IE8%20-%20WinXP.box
==> box: Successfully added box 'XP' (v0) for 'virtualbox'!
```

### 3.Vagrant command

```plain

vagrant box add base URL

vagrant init

vagrant up

vagrant box list

默认BOX文件路径
/Users/daijiajia/.vagrant.d/boxes

vagrant status

vagrant ssh 默认账户/密码 vagrant/vagrant

vagrant suspend

vagrant resume

vagrant halt

vagrant destory

```

## 二、QEMU
> QEMU 本身是一个非常强大的虚拟机，甚至在 Xen、KVM 这些虚拟机产品中都少不了 QEMU 的身影。在 QEMU 的官方文档中也提到，QEMU 可以利用 Xen、KVM 等技术来加速。为什么需要加速呢，那是因为如果单纯使用 QEMU 的时候，它自己模拟出了一个完整的个人电脑，它里面的 CPU 啊什么的都是模拟出来的，它甚至可以模拟不同架构的 CPU，比如说在使用 Intel X86 的 CPU 的电脑中模拟出一个 ARM 的电脑或 MIPS 的电脑，这样模拟出的 CPU 的运行速度肯定赶不上物理 CPU。使用加速以后呢，可以把客户操作系统的 CPU 指令直接转发到物理 CPU，自然运行效率大增。

QEMU 同时也是一个非常简单的虚拟机，给它一个硬盘镜像就可以启动一个虚拟机，如果想定制这个虚拟机的配置，比如用什么样的 CPU 啊、什么样的显卡啊、什么样的网络配置啊，指定相应的命令行参数就可以了。它支持许多格式的磁盘镜像，包括 VirtualBox 创建的磁盘镜像文件。它同时也提供一个创建和管理磁盘镜像的工具 qemu-img。QEMU 及其工具所使用的命令行参数，直接查看其文档即可。

### 1.Ubuntu16 安装QEMU
```plain
sudo apt install aptitude
sudo dpkg -L  qemu-system-x86
```
- [包管理系统指南](http://wiki.ubuntu.org.cn/%E5%8C%85%E7%AE%A1%E7%90%86%E7%B3%BB%E7%BB%9F%E6%8C%87%E5%8D%97)

### 2.创建镜像并加载
```plain
# qemu-img create -f qcow2 WinXP.img 10G
Formatting 'WinXP.img', fmt=qcow2 size=10737418240 encryption=off cluster_size=65536 lazy_refcounts=off refcount_bits=16


# qemu-system-i386 -m 2G WinXP.img -cdrom zh-hans_windows_xp_professional_with_service_pack_3_x86_cd_vl_x14-74070.iso

# ll
总用量 615804
drwxr-xr-x  2 root root      4096 1月  23 10:38 ./
drwx------ 19 root root      4096 1月  22 18:21 ../
-rw-r--r--  1 root root    393216 1月  23 10:45 WinXP.img
-rwxrwx---  1 root root 630237184 1月  23 10:00 zh-hans_windows_xp_professional_with_service_pack_3_x86_cd_vl_x14-74070.iso*
```
最终启动的结果可以如图所示：
![image](/{{site.url}}/assets/2017/02/Qemu-start-XP.png)

### 3.VirtualBox 虚拟机磁盘调整

#### 1.VBoxManage list hdds

#### 2.VBoxManage modifyhd uuid --resize size命令将其扩容，uuid是上面获得的UUID，size是扩容后的大小。以兆为单位。


```plain
VBoxManage list hdds
UUID:           3e3685e7-5750-444e-8d76-4c1e1959c482
Parent UUID:    base
State:          created
Type:           normal (base)
Location:       /Users/daijiajia/.docker/machine/machines/default/disk.vmdk
Storage format: VMDK
Capacity:       20000 MBytes
Encryption:     disabled

UUID:           77a5561b-9434-49ca-ad41-6c206a753d34
Parent UUID:    base
State:          created
Type:           normal (base)
Location:       /Users/daijiajia/VirtualBox VMs/1504_default_1485010940924_10767/box-disk2.vmdk
Storage format: VMDK
Capacity:       40960 MBytes
Encryption:     disabled

UUID:           d49a6db4-2df6-4a76-8dc6-dead4be532c0
Parent UUID:    base
State:          created
Type:           normal (base)
Location:       /Users/daijiajia/VirtualBox VMs/xp_default_1485013415710_85029/box-disk1.vmdk
Storage format: VMDK
Capacity:       130048 MBytes
Encryption:     disabled

UUID:           24fb472d-11a0-4c0e-81e5-7c92fa283ee1
Parent UUID:    base
State:          created
Type:           normal (base)
Location:       /Users/daijiajia/VirtualBox VMs/Ubuntu16.04/Ubuntu16.04.vdi
Storage format: VDI
Capacity:       8192 MBytes
Encryption:     disabled

VBoxManage modifyhd 24fb472d-11a0-4c0e-81e5-7c92fa283ee1 --resize 16384
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
```

#### 3.gpated重新分区

### 4.启动镜像
```plain
 qemu-system-i386 -m 1G -smp 2 -vga vmware W98.img
```
- [Virtualbox虚拟硬盘根分区大小调整](http://aaronmoment.cn/partition/)

## 三、KVM

QEMU 是一个强大的虚拟机软件，它可以完全以软件的形式模拟出一台完整的电脑所需的所有硬件，甚至是模拟出不同架构的硬件，在这些虚拟的硬件之上，可以安装完整的操作系统。QEMU 的运行模式如下图：
![image](/{{site.url}}/assets/2017/02/qemu_without_kvm.png)

很显然，这种完全以软件模拟硬件的形式虽然功能强大，但是性能难以满足用户的需要。模拟出的硬件的性能和物理硬件的性能相比，必然会大打折扣。为了提高虚拟机软件的性能，开发者们各显神通。其中，最常用的办法就是在主操作系统中通过内核模块开一个洞，通过这个洞将虚拟机中的操作直接映射到物理硬件上，从而提高虚拟机中运行的操作系统的性能。如下图：
![image](/{{site.url}}/assets/2017/02/qemu_with_kvm.png)

> 其中 KVM 就是这种加速模式的典型代表。在社区中，大家常把 KVM 和 Xen 相提并论，但是它们其实完全不一样。从上图可以看出，使用内核模块加速这种模式，主操作系统仍然占主导地位，内核模块只是在主操作系统中开一个洞，用来连接虚拟机和物理硬件，给虚拟机加速，但是虚拟机中的客户操作系统仍然受到很大的限制。这种模式比较适合桌面用户使用，主操作系统仍然是他们的主战场，不管是办公还是打游戏，都通过主操作系统完成，客户操作系统只是按需使用。至于 Xen，则完全使用不同的理念，比较适合企业级用户使用

### 1.Ubuntu16 安装KVM
```plain
# sudo aptitude install kvm

# sudo aptitude install qemu-kvm

# kvm
Could not access KVM kernel module: No such file or directory
failed to initialize KVM: No such file or directory

```

### 2.KVM在虚拟机中启用的问题
[How to fix error ‘Could not access KVM kernel module’ in Proxmox, Virtualizor, SolusVM, Redhat, CentOS and Ubuntu
](https://bobcares.com/blog/how-to-fix-error-could-not-access-kvm-kernel-module/)

#### 1.lscpu

```plain
# lscpu
Architecture:          x86_64
CPU 运行模式：    32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                1
On-line CPU(s) list:   0
每个核的线程数：1
每个座的核数：  1
Socket(s):             1
NUMA 节点：         1
厂商 ID：           GenuineIntel
CPU 系列：          6
型号：              70
Model name:            Intel(R) Core(TM) i7-4770HQ CPU @ 2.20GHz
步进：              1
CPU MHz：             2194.918
BogoMIPS:              4389.83
超管理器厂商：  KVM
虚拟化类型：     完全
L1d 缓存：          32K
L1i 缓存：          32K
L2 缓存：           256K
L3 缓存：           6144K
L4 缓存：           131072K
NUMA node0 CPU(s):     0
Flags:                 fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 syscall nx rdtscp lm constant_tsc rep_good nopl xtopology nonstop_tsc pni pclmulqdq monitor ssse3 cx16 sse4_1 sse4_2 movbe popcnt aes xsave avx rdrand hypervisor lahf_lm abm

```

#### 2.modprob kvm

```plain
root@dj-VirtualBox:~/os# modprobe kvm
root@dj-VirtualBox:~/os# kvm
Could not access KVM kernel module: No such file or directory
failed to initialize KVM: No such file or directory
root@dj-VirtualBox:~/os# modeprobe kvm_intel
未找到 'modeprobe' 命令，您要输入的是否是：
 命令 'modprobe' 来自于包 'kmod' (main)
modeprobe：未找到命令
root@dj-VirtualBox:~/os# modprobe kvm_intel
modprobe: ERROR: could not insert 'kvm_intel': Operation not supported
root@dj-VirtualBox:~/os# sudo modprobe kvm_intel
modprobe: ERROR: could not insert 'kvm_intel': Operation not supported
root@dj-VirtualBox:~/os# sudo modprobe kvm_amd
modprobe: ERROR: could not insert 'kvm_amd': Operation not supported
root@dj-VirtualBox:~/os# lsmod | grep kvm
kvm                   536576  0
irqbypass              16384  1 kvm

```

#### 3.dmesg

```plain
[ 1440.925082] Out of memory: Kill process 8697 (qemu-system-x86) score 559 or sacrifice child
[ 1440.925093] Killed process 8697 (qemu-system-x86) total-vm:2486184kB, anon-rss:1142780kB, file-rss:508kB
[ 1441.061570] virbr0: port 1(vnet0) entered disabled state
[ 1441.062864] device vnet0 left promiscuous mode
[ 1441.062868] virbr0: port 1(vnet0) entered disabled state
[ 1441.645025] audit: type=1400 audit(1485182104.977:39): apparmor="STATUS" operation="profile_remove" profile="unconfined" name="libvirt-d71d29f0-b411-4ad6-8b17-55d2cf16773c" pid=8755 comm="apparmor_parser"
[ 1817.949134] kvm: no hardware support
[ 1825.126022] kvm: no hardware support
[ 1828.070295] has_svm: not amd
[ 1828.070298] kvm: no hardware support

```
[Libvirt错误总结](http://liuzhijun.iteye.com/blog/1783698)

> 结论：在宿主机上安装的的虚拟软件中的虚拟机不支持KVM加速


### 3.安装和使用virt-manager
如果用 ps -ef | grep qemu 命令查看一下，发现 kvm 命令运行的还是 qemu-system-x86_64 程序，只不过加上了 -enable-kvm 参数

另外，对于桌面用户来说，有一个好用的图形化界面也是很重要的。虽然 QEMU 和 KVM 自身不带图形界面的虚拟机管理器，但是我们可以使用第 3 方软件，比如 virt-manager。只需要使用 sudo apt-get install virt-manager 即可安装该软件。该软件依赖于 libvirt，在安装过程中也会自动安装。运行 virt-manager 的效果如下图，注意必须使用 sudo 运行，因为该软件需要超级用户权限。
![image](/{{site.url}}/assets/2017/02/VirtualBox_Ubuntu16_virtmanger.png)

- [虚拟机体验之 KVM 篇](http://www.cnblogs.com/youxia/p/linux020.html)

## 四、XEN
- [虚拟机体验之 Xen 篇 —— 令人脑洞大开的奇异架构](http://www.cnblogs.com/youxia/p/linux022.html)

比如说在经典的虚拟机架构中，虚拟机软件运行于 Host System 之中，而 Guest System 运行于虚拟机软件之中。为了提高 Guest System 的运行速度，虚拟机软件一般会在 Host System 中使用内核模块开一个洞，将 Guest System 的运行指令直接映射到物理硬件上。但是在 Xen 中，则根本没有 Host System 的概念，传说它所有的虚拟机都直接运行于硬件之上，虚拟机运行的效率非常的高，虚拟机之间的隔离性非常的好。

　　当然，传说只是传说。我刚开始也是很纳闷，怎么可能让所有的虚拟机都直接运行于硬件之上。后来我终于知道，这只是一个噱头。虚拟机和硬件之间，还是有一个管理层的，那就是 Xen Hypervisor。当然 Xen Hypervisor 的功能毕竟是有限的，怎么样它也比不上一个操作系统，因此，在 Xen Hypervisor 上运行的虚拟机中，有一个虚拟机是具有特权的，它称之为 Domain 0，而其它的虚拟机都称之为 Domain U。

　　Xen的架构如下图：
![image](/{{site.url}}/assets/2017/02/xen.png)

从图中可以看出，Xen 虚拟机架构中没有 Host System，在硬件层之上是薄薄的一层 Xen Hypervisor，在这之上就是各个虚拟机了，没有 Host System，只有 Domain 0，而 Guest System 都是 Domain U，不管是 Domain 0 还是 Domain U，都是虚拟机，都是被虚拟机软件管理的对象。

　　既然 Domain 0 也是一个虚拟机，也是被管理的对象，所以可以给它分配很少的资源，然后将其余的资源公平地分配到其它的 Domain。但是很奇怪的是，所有的虚拟机管理软件其实都是运行在这个 Domain 0 中的。同时，如果要连接到其它 Guest System 的控制台，而又不是使用远程桌面（VNC）的话，这些控制台也是显示在 Domian 0 中的。所以说，这是一个奇异的架构，是一个让人很不容易理解的架构。

　　这种架构桌面用户不喜欢，因为 Host System 变成了 Domain 0，本来应该掌控所有资源的主操作系统变成了一个受管理的虚拟机，本来用来打游戏、编程、聊天的主战场受到限制了，可能不能完全发挥硬件的性能了，还有可能运行不稳定了，自然会心里不爽。（Domain 0确实不能安装专用显卡驱动，确实会运行不稳定，这个后面会讲。）但是企业级用户喜欢，因为所有的 Domain 都是虚拟机，所以可以更加公平地分配资源，而且由于 Domain U 不再是运行于 Domian 0 里面的软件，而是和 Domain 0 平级的系统，这样即使 Domain 0 崩溃了，也不会影响到正在运行的 Domain U。

### Mac VirtualBox Ubuntu 文件夹共享

- [mac Virtualbox Ubuntu 设置共享目录](http://www.cnblogs.com/200911/p/4947207.html)

Ubuntu16.10 启用root账户

```plain
sudo passwd root

```

Ubuntu16.10 启用root账户登陆
- [ubuntu 16.04启用root用户方法](http://www.linuxdiyf.com/linux/22630.html)


## 五、bochs

操作系统Ubuntu16.04
源码 Bochs 2.6.8

- [Ubuntu下安装DOS环境与安装配置Bochs](http://codingstory.com.cn/ubuntuxia-an-zhuang-doshuan-jing-yu-an-zhuang-pei-zhi-bochs/)

### 1.安装前准备

```plain
 sudo apt-get install libx11-dev
 sudo apt-get install libxkbcommon-x11-dev
 sudo apt-get install libghc-x11-dev
```

### 2.安装步骤

```plian
./configure --enable-debugger --enable-disasm
make
make install
```

### 3.make遇到的问题

```plain
x.cc:42:35: fatal error: X11/extensions/Xrandr.h: 没有那个文件或目录
compilation terminated.
Makefile:114: recipe for target 'x.o' failed
make[1]: *** [x.o] Error 1
make[1]: Leaving directory '/root/bochs-2.6.8/gui'
Makefile:349: recipe for target 'gui/libgui.a' failed
make: *** [gui/libgui.a] Error 2

```

- [ Ubuntu 14.04LTS 安装和配置Bochs](http://blog.csdn.net/qq_23827747/article/details/51377291)

```plain
apt-get install xorg-dev

make 成功如下

/bin/bash ./libtool --mode=link --tag CXX g++ -o bximage -g -O2 -D_FILE_OFFSET_BITS=64 -D_LARGE_FILES     misc/bximage.o misc/hdimage.o misc/vmware3.o misc/vmware4.o misc/vpc-img.o misc/vbox.o
g++ -o bximage -g -O2 -D_FILE_OFFSET_BITS=64 -D_LARGE_FILES misc/bximage.o misc/hdimage.o misc/vmware3.o misc/vmware4.o misc/vpc-img.o misc/vbox.o


make install 结果
cat ./build/linux/README.linux-binary ./README > /usr/local/share/doc/bochs/README
install -m 644 ./.bochsrc /usr/local/share/doc/bochs/bochsrc-sample.txt


```
### 4.启动bochs遇到的问题

```plain
bochs
========================================================================
                       Bochs x86 Emulator 2.6.8
                Built from SVN snapshot on May 3, 2015
                  Compiled on Jan 23 2017 at 18:54:06
========================================================================
00000000000i[      ] BXSHARE not set. using compile time default '/usr/local/share/bochs'
00000000000i[      ] reading configuration from .bochsrc
00000000000e[      ] .bochsrc:186: invalid choice 'core2_penryn_t9600' parameter 'model'
00000000000p[      ] >>PANIC<< .bochsrc:186: cpu directive malformed.
00000000000e[SIM   ] notify called, but no bxevent_callback function is registered
========================================================================
Bochs is exiting with the following message:
[      ] .bochsrc:186: cpu directive malformed.
========================================================================
00000000000i[SIM   ] quit_sim called with exit code 1
```

解决办法

```plain
[      ] .bochsrc:186: cpu directive malformed.

[      ] .bochsrc:907: Bochs is not compiled with lowlevel sound support

各种原因都有，总的来说就是bochs配置文件不对

指定配置文件bochsrc，写入

romimage: file="/home/dj/bochs-2.6.8/bios/BIOS-bochs-latest"
vgaromimage: file="/home/dj/bochs-2.6.8/bios/VGABIOS-lgpl-latest"

ata0-master: type=disk, path="disk.img", cylinders=200, heads=16, spt=63

boot: disk

#日志输出
log: bochsout.txt

运行OK

# bochs
========================================================================
                       Bochs x86 Emulator 2.6.8
                Built from SVN snapshot on May 3, 2015
                  Compiled on Jan 23 2017 at 19:10:23
========================================================================
00000000000i[      ] BXSHARE not set. using compile time default '/usr/local/share/bochs'
00000000000i[      ] reading configuration from .bochsrc
00000000000e[      ] .bochsrc:716: ataX-master/slave CHS set to 0/0/0 - autodetection enabled
00000000000p[      ] >>PANIC<< .bochsrc:909: Bochs is not compiled with lowlevel sound support
00000000000e[SIM   ] notify called, but no bxevent_callback function is registered
========================================================================
Bochs is exiting with the following message:
[      ] .bochsrc:909: Bochs is not compiled with lowlevel sound support
========================================================================
00000000000i[CPU0  ] CPU is in real mode (active)
00000000000i[CPU0  ] CS.mode = 16 bit
00000000000i[CPU0  ] SS.mode = 16 bit
00000000000i[CPU0  ] EFER   = 0x00000000
00000000000i[CPU0  ] | EAX=00000000  EBX=00000000  ECX=00000000  EDX=00000000
00000000000i[CPU0  ] | ESP=00000000  EBP=00000000  ESI=00000000  EDI=00000000
00000000000i[CPU0  ] | IOPL=0 id vip vif ac vm rf nt of df if tf sf ZF af PF cf
00000000000i[CPU0  ] | SEG sltr(index|ti|rpl)     base    limit G D
00000000000i[CPU0  ] |  CS:0000( 0000| 0|  0) 00000000 00000000 0 0
00000000000i[CPU0  ] |  DS:0000( 0000| 0|  0) 00000000 00000000 0 0
00000000000i[CPU0  ] |  SS:0000( 0000| 0|  0) 00000000 00000000 0 0
00000000000i[CPU0  ] |  ES:0000( 0000| 0|  0) 00000000 00000000 0 0
00000000000i[CPU0  ] |  FS:0000( 0000| 0|  0) 00000000 00000000 0 0
00000000000i[CPU0  ] |  GS:0000( 0000| 0|  0) 00000000 00000000 0 0
00000000000i[CPU0  ] | EIP=00000000 (00000000)
00000000000i[CPU0  ] | CR0=0x00000000 CR2=0x00000000
00000000000i[CPU0  ] | CR3=0x00000000 CR4=0x00000000
bx_dbg_read_linear: physical memory read error (phy=0x000000000000, lin=0x00000000)
00000000000i[SIM   ] quit_sim called with exit code 1
root@dj-VirtualBox:~/bochs-2.6.8# bochs -f bochsrc
========================================================================
                       Bochs x86 Emulator 2.6.8
                Built from SVN snapshot on May 3, 2015
                  Compiled on Jan 23 2017 at 19:10:23
========================================================================
00000000000i[      ] BXSHARE not set. using compile time default '/usr/local/share/bochs'
00000000000i[      ] reading configuration from bochsrc
------------------------------
Bochs Configuration: Main Menu
------------------------------

This is the Bochs Configuration Interface, where you can describe the
machine that you want to simulate.  Bochs has already searched for a
configuration file (typically called bochsrc.txt) and loaded it if it
could be found.  When you are satisfied with the configuration, go
ahead and start the simulation.

You can also start bochs with the -q option to skip these menus.

1. Restore factory default configuration
2. Read options from...
3. Edit options
4. Save options to...
5. Restore the Bochs state from...
6. Begin simulation
7. Quit now

Please choose one: [6] 7
00000000000i[SIM   ] quit_sim called with exit code 1



```


### 5.XP镜像启动配置文件 bochs.xp

```plain
# bochsrc
#
megs: 128
#
mouse: enabled=1
#
ata0-master: type=disk, path="/root/os/c.img", mode=flat, cylinders=609, heads=16, spt=63
#ata1-master: type=cdrom, path=./plan9.iso, status=inserted
#
log: bochsout.txt
#
boot: disk

```

![image](/{{site.url}}/assets/2017/02/VirtualBox_Ubuntu16_xp_start.png)

## 参考

- [installing-qemu-on-mac-os-x-el-capitan](https://theintobooks.wordpress.com/2016/03/03/installing-qemu-on-mac-os-x-el-capitan/)
- [虚拟机比较](http://wiki.osdev.org/Emulator_Comparison)


### 搭建集群
- [3 Vgrant使用入门](https://github.com/astaxie/go-best-practice/blob/master/ebook/zh/01.3.md)
- [路径（七）：用 Vagrant 管理虚拟机](https://ninghao.net/blog/2077)
- [Discover Vagrant Boxes](https://atlas.hashicorp.com/boxes/search)
- [开始使用 Vagrant](https://imququ.com/post/vagrantup.html)

### Bochs编译
- [Ubuntu 14.04LTS 安装和配置Bochs](http://blog.csdn.net/qq_23827747/article/details/51377291)
- [bochs 编译](http://blog.csdn.net/cooljiansir/article/details/38114043)

### 操作系统实验
- [自己动手写操作系统](http://www.cnblogs.com/helloweworld/category/738062.html)
- [自制操作系统](http://zhaodedong.leanote.com/cate/%E8%87%AA%E5%88%B6%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F)
- [操作系统实验](https://github.com/hoverwinter/HIT-OSLab)
- [操作系统指导手册](https://hoverwinter.gitbooks.io/hit-oslab-manual/content/)
