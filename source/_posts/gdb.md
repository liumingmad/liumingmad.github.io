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
```

### 6. 调试汇编
```
# 查看反汇编
objdump -d -M intel a.out

(gdb) set disassembly-flavor intel

# starti 停在最开始，再查真实地址
(gdb) starti
(gdb) info proc mappings    # 找到程序加载的基地址
(gdb) b *0x555555555060     # 基地址 + 0x1060

(gdb) r
(gdb) layout asm
(gdb) layout regs
# 之后用 ni/si 单步，观察寄存器变化
(gdb) x/8xg $rsp       # 随时查看栈
```
