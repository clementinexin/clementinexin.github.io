---
layout: post
title:  "MAC上读写NTFS磁盘"
date: 2017-01-02
categories: Mac,Tools
tags: Mac,NTFS
published: true
---
* 目录
{:toc}


方法很多，失效的也很快，特别是在安装系统更新之后，也不想下载乱七八糟的软件。
参考了这个[Mac OS X Yosemite(10.10.2) 读写 NTFS](https://www.zybuluo.com/haokuixi/note/102816)

## 一、工具使用diskutil list

```plain
diskutil list

daijiajia@daijiajiadeMacBook-Pro  ~  diskutil list
/dev/disk0 (internal, physical):
  #:                       TYPE NAME                    SIZE       IDENTIFIER
  0:      GUID_partition_scheme                        *251.0 GB   disk0
  1:                        EFI EFI                     209.7 MB   disk0s1
  2:          Apple_CoreStorage Macintosh HD            250.1 GB   disk0s2
  3:                 Apple_Boot Recovery HD             650.0 MB   disk0s3

/dev/disk1 (internal, virtual):
  #:                       TYPE NAME                    SIZE       IDENTIFIER
  0:                            Macintosh HD           +249.8 GB   disk1
                                Logical Volume on disk0s2
                                91EBAACD-9342-4378-A604-022C4847E8AE
                                Unencrypted

/dev/disk2 (disk image):
  #:                       TYPE NAME                    SIZE       IDENTIFIER
  0:      GUID_partition_scheme                        +684.6 MB   disk2
  1:                  Apple_HFS IntelliJ IDEA CE        684.5 MB   disk2s1

/dev/disk3 (external, physical):
  #:                       TYPE NAME                    SIZE       IDENTIFIER
  0:     FDisk_partition_scheme                        *1.0 TB     disk3
  1:               Windows_NTFS My Passport             1.0 TB     disk3s1

/dev/disk4 (disk image):
  #:                       TYPE NAME                    SIZE       IDENTIFIER
  0:     Apple_partition_scheme                        +33.6 MB    disk4
  1:        Apple_partition_map                         32.3 KB    disk4s1
  2:                  Apple_HFS Chrome Remote Deskto... 33.6 MB    disk4s2

/dev/disk5 (external, physical):
  #:                       TYPE NAME                    SIZE       IDENTIFIER
  0:     FDisk_partition_scheme                        *320.1 GB   disk5
  1:               Windows_NTFS NESO_xHDD_by_HITACHI    320.1 GB   disk5s1
```

> 在iTerm下无法运行，只能在Terminal中运行不止何为

```plain
⚙ daijiajia@daijiajiadeMacBook-Pro  ~/exercise/js/react/sample/service  diskutil list
Unable to run because unable to use the DiskManagement framework.
Common reasons include, but are not limited to, the DiskArbitration
framework being unavailable due to being booted in single-user mode.
⚙ daijiajia@daijiajiadeMacBook-Pro  ~/exercise/js/react/sample/service  su diskutil list
su: who are you?
✘ ⚙ daijiajia@daijiajiadeMacBook-Pro  ~/exercise/js/react/sample/service  sudo diskutil list
sudo: unknown uid 501: who are you?
```

## 二、安装ntfs-3g

```plain
brew install homebrew/fuse/ntfs-3g
```

```plain
✘ ⚙ daijiajia@daijiajiadeMacBook-Pro  ~/exercise/js/react/sample/service  brew install homebrew/fuse/ntfs-3g

/System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/lib/ruby/2.0.0/universal-darwin16/rbconfig.rb:213: warning: Insecure world writable dir /opt in PATH, mode 040777
==> Tapping homebrew/fuse
Cloning into '/usr/local/Homebrew/Library/Taps/homebrew/homebrew-fuse'...
fatal: unable to access 'https://github.com/Homebrew/homebrew-fuse/': Could not resolve host: github.com
Error: Failure while executing: git clone https://github.com/Homebrew/homebrew-fuse /usr/local/Homebrew/Library/Taps/homebrew/homebrew-fuse --depth=1
```

```plain
vi /etc/reslov.conf
nameserver 8.8.8.8
```
> 同样只能在Terminal中运行

```plain
 sudo vi /etc/resolv.conf
sudo: unknown uid 501: who are you?
```

## 三、挂载磁盘

```plain
sudo /usr/local/bin/ntfs-3g /dev/disk5 /Volumes/NTFS -olocal -oallow_other

Error opening '/dev/disk5': Resource busy
Failed to mount '/dev/disk5': Resource busy
Mount is denied because the NTFS volume is already exclusively opened.
The volume may be already mounted, or another software may use it which
could be identified for example by the help of the 'fuser' command.
```
> 虽然没有应用程序打开磁盘，因为磁盘已经挂载，需要先推出

```plain
sudo /usr/local/bin/ntfs-3g /dev/disk5 /Volumes/NTFS -olocal -oallow_other

Password:
NTFS signature is missing.
Failed to mount '/dev/disk5': Invalid argument
The device '/dev/disk5' doesn't seem to have a valid NTFS.
Maybe the wrong device is used? Or the whole disk instead of a
partition (e.g. /dev/sda, not /dev/sda1)? Or the other way around?
```
> 仔细查看diskutil list的返回结果，将disk5改为disk5s1便可

```plain
sudo /usr/local/bin/ntfs-3g /dev/disk5s1 /Volumes/NTFS -olocal -oallow_other

The disk contains an unclean file system (0, 1).
The file system wasn't safely closed on Windows. Fixing.
```

##  四、结果

![image]({{site.url}}/assets/2017/01/NTFS-in_Mac.png)

## 五、参考

- [NTFS 3G](https://github.com/osxfuse/osxfuse/wiki/NTFS-3G)
