title: 实现一个最小bootloader
author: ming
tags: []
categories:
  - i386
date: 2026-02-22 17:51:00
---
## 如何实现一个最小bootloader 
### 1. bootloader的汇编代码
```asm
; bootloader.asm - minimal boot sector
; assemble: nasm -f bin bootloader.asm -o boot.bin

BITS 16
ORG 0x7C00

start:
    cli
    xor ax, ax
    mov ds, ax
    mov es, ax
    mov ss, ax
    mov sp, 0x7C00          ; 简单栈：放在自身下方
    sti

    mov si, msg
.print:
    lodsb                   ; AL = [DS:SI], SI++
    test al, al
    jz .hang
    mov ah, 0x0E            ; teletype 输出
    mov bh, 0x00
    mov bl, 0x07            ; 灰字黑底
    int 0x10
    jmp .print

.hang:
    jmp .hang

msg db "Hello from bootloader!", 0

; 填充到 510 字节
times 510-($-$$) db 0
dw 0xAA55                   ; boot signature
```

### 2. 编译运行
```
nasm -f bin bootloader.asm -o boot.bin
ls -l boot.bin   # 应该正好 512 bytes

# 运行
qemu-system-i386 -drive format=raw,file=boot.bin
```

### 3. qemu+GDB调试
  * 用 QEMU 挂起 CPU + 开 gdbserver
```
# 关键参数：-S（上电就暂停），-s（在 1234 端口开 gdb stub）
qemu-system-i386 \
  -drive format=raw,file=boot.bin \
  -S -s \
  -monitor stdio
```

* 另开一个终端，用 GDB 连接 QEMU
```
gdb -q
```

```
set architecture i8086
target remote :1234

# BIOS 会把你加载到物理 0x7C00，入口一般是 0000:7C00
break *0x7c00
continue
```

```
#查看寄存器状态和汇编
info registers
x/16i 0x7c00          # 从 0x7C00 反汇编
x/32xb 0x7c00         # 看机器码字节
```

```
si      # step instruction，单步一条指令
ni      # next instruction，通常对 call/int 更“跳过”一些（但 int 的行为比较特殊）
```