title: 编译aosp
author: ming
date: 2020-12-23 10:56:00
tags:
---
### 环境
*  ubuntu 20.04.2
*  SamSung SSD T7 1T
*  内存16GB

### 下载aosp源码
* 下面这两个源，任选一个就行，如果中断了，也可以换另外一个镜像继续下载，因为两个tar的md5都是一样的 
* 这个tar包大概95GB，我下载了两天，可以中断，wget -c有断点续传的功能
```bash
# 科大镜像
wget -c https://mirrors.ustc.edu.cn/aosp-monthly/aosp-latest.tar &
# 清华镜像
wget -c https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar
```

### 校验md5
```bash
# 超级慢，因为包太大
md5sum aosp-latest.tar
803b418c730ab10db3c44df3ae8e7b1f aosp-latest.tar
```

### 解压
```bash
tar xf aosp-latest.tar
```

* 问题: 如果遇到解压错误，tar Cannot create symlink to Operation not permitted, 
* 原因: 一般是磁盘文件系统不一致导致的，我的ssd是exfat，不支持软链接
* 解决: 
    1. lsblk -f 先查看磁盘文件系统
    2. 把文件系统格式化成ext4

### 同步代码
```bash
# 因为使用清华的镜像做同步时，总提示error: RPC failed; curl 56 GnuTLS recv error (-9): Error decoding the received TLS packet. 所以配置了vpn，改成google的源
repo init -u https://android.googlesource.com/platform/manifest -b android-11.0.0_r4
repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest -b android-11.0.0_r4

# 可以不断重复，同步命令, 直到出现这个提示
# Checking out projects: 100% (781/781), done.
# repo sync has finished successfully.
repo sync -c -j8

```

### repo操作
```bash
# 创建dev分支
repo start dev --all
```

### 编译
```bash

source build/envsetup.sh

# 对应Pixel 2
# https://source.android.com/setup/build/running#selecting-device-build
lunch aosp_walleye-userdebug

# 如果中断，可以重复执行m
# 直到#### build completed successfully (24:07 (mm:ss)) ####
m
```

### 刷机
```bash
ming@ming-Thurley:/media/ming/mingssd/android/aosp$ which adb
/media/ming/mingssd/android/aosp/out/soong/host/linux-x86/bin/adb
ming@ming-Thurley:/media/ming/mingssd/android/aosp$ adb reboot bootloader
ming@ming-Thurley:/media/ming/mingssd/android/aosp$ fastboot flashall -w
--------------------------------------------
Bootloader Version...: mw8998-002.0083.00
Baseband Version.....: g8998-00023-2004070200
Serial Number........: HT8591A01651
--------------------------------------------
Checking 'product'                                 OKAY [  0.001s]
Setting current slot to 'b'                        OKAY [  0.009s]
Sending 'boot_b' (32768 KB)                        OKAY [  1.309s]
Writing 'boot_b'                                   OKAY [  0.199s]
Sending 'dtbo_b' (8192 KB)                         OKAY [  0.349s]
Writing 'dtbo_b'                                   OKAY [  0.047s]
Sending 'vbmeta_b' (8 KB)                          OKAY [  0.011s]
Writing 'vbmeta_b'                                 OKAY [  0.001s]
Sending sparse 'system_b' 1/3 (458752 KB)          OKAY [ 17.126s]
Writing 'system_b'                                 OKAY [  0.000s]
Sending sparse 'system_b' 2/3 (458752 KB)          OKAY [ 18.620s]
Writing 'system_b'                                 OKAY [  0.000s]
Sending sparse 'system_b' 3/3 (200392 KB)          OKAY [  9.960s]
Writing 'system_b'                                 OKAY [  0.000s]
Sending 'system_a' (78268 KB)                      OKAY [  2.959s]
Writing 'system_a'                                 OKAY [  0.000s]
Sending 'vendor_b' (468008 KB)                     OKAY [ 17.787s]
Writing 'vendor_b'                                 OKAY [  0.000s]
Erasing 'userdata'                                 OKAY [  2.659s]
mke2fs 1.45.4 (23-Sep-2019)
Creating filesystem with 13933563 4k blocks and 3489792 inodes
Filesystem UUID: 0e6436b0-06fc-496e-9aaf-a352dea25ede
Superblock backups stored on blocks:
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
	4096000, 7962624, 11239424

Allocating group tables: done
Writing inode tables: done
Creating journal (65536 blocks): done
Writing superblocks and filesystem accounting information: done

Sending 'userdata' (4669 KB)                       OKAY [  0.189s]
Writing 'userdata'                                 OKAY [  0.001s]
Rebooting                                          OKAY [  0.000s]
Finished. Total time: 76.247s
```