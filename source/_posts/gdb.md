title: gdb
author: ming
tags:
  - c/c++
categories:
  - c/c++
date: 2020-12-19 10:56:00
---
# gdb

### 1. 启动gdb
```shell
gdb ./myps
```

### 2. 设置命令行 参数 
```shell
set args ming
```

### 3. 查看代码行号
```shell
list
```

### 4. 设置断点
```shell
# 在第18行设置断点
b 18

# 查看设置的断点
i b

# disable 编号是1的断点
disable b 1

# enable 编号是1的断点
enable b 1
```

### 5. 查看变量的值
```shell
p linebuf
```

### 6. 下一步
```shell
# 在当前函数中执行下一行
n

# 进入本行的函数中
s

# 跳出当前函数
fin

# 继续执行到下一个断点
c
···