
title: 怎样用ndk编译一个可执行程序
author: ming
date: 2021-04-13 18:34:00
tags:
---

## 怎样用ndk编译一个可执行程序

#### 1. 下载NDK，配置环境变量  
```shell
vim ~/.bash_profile

# 追加下面两行
export ANDROID_NDK=/Users/liuming/Library/Android/sdk/ndk-bundle
export PATH=$ANDROID_NDK:$PATH

# 试一下是否可用
ndk-build
```

#### 2. 准备3个文件 
```c
// hello.c
#include <stdio.h>

int main() {
    printf("hello world!!\n");
    return 0;
}
```

```shell
# Android.mk

LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_SRC_FILES := hello.c
LOCAL_MODULE    := hello

# 生成可执行文件
include $(BUILD_EXECUTABLE)
```

```shell
# Application.mk
# 可用通过adb shell进入手机后，cat /proc/cpuinfo查看cpu架构, 
# walleye:/system/bin # cat /proc/cpuinfo
# Processor	: AArch64 Processor rev 4 (aarch64)
# processor	: 0
# BogoMIPS	: 38.00
# Features	: fp asimd evtstrm aes pmull sha1 sha2 crc32
# CPU implementer	: 0x51
# CPU architecture: 8 -------表示armv8a
# CPU variant	: 0xa
# CPU part	: 0x801
# CPU revision	: 4
# 
# processor	: 1
# BogoMIPS	: 38.00
# Features	: fp asimd evtstrm aes pmull sha1 sha2 crc32
# CPU implementer	: 0x51
# CPU architecture: 8
# CPU variant	: 0xa
# CPU part	: 0x801
# CPU revision	: 4

APP_ABI :=arm64-v8a
```

#### 4. 编译
```shell
# 指定mk文件的位置
ndk-build NDK_PROJECT_PATH=. NDK_APPLICATION_MK=Application.mk APP_BUILD_SCRIPT=Android.mk
```

#### 5. 上传到手机 & 执行
```shell
adb push hello /data/local/tmp

cd /data/local/tmp
./hello
hello world!!
```