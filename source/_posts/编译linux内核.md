title: 编译linux内核
author: ming
date: 2024-02-26 22:41:32
tags:
---
#### 1. 安装交叉编译工具
```
sudo apt install gcc-arm-linux-gnueabi
```

#### 2. 配置环境变量
```
export ARCH=arm
CROSS_COMPILE=arm-linux-gnueabi-
```

#### 3. 生成.config
```
make vexpress_defconfig
```

#### 4. 编译
```
make -j4 ARCH=arm CROSS_COMPILE=arm-linux-gnueabi-
```

#### 5. 编译模块
```
make modules
```

#### 6. 编译dtbs
```
make dtbs
```