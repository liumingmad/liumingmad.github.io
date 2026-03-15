title: crackme-crackit
author: ming
tags:
  - 逆向
categories:
  - 逆向
date: 2026-03-15 10:56:00
---
# CrackIt;)

https://crackmes.one/crackme/69a65ee60f5b9757a6a5f73f

### 1. 下载解压
zip解压密码：crackmes.one

### 2. IDA free 
* 先看字符串
字符串中没有直接给出

* 再看结构
![](/images/kkdklsd434343aaa.png)

### 3. GDB
```shell
# 先越过字符串长度的限制，判断了0x28=40个字符

# 找到关键指令
# b *0x55555555547b
call    _memcmp
```


### 5. 验证结果
```
./crackit "CTF{My_S3cr3t_Fl4g}W0WY0uF0undM3{0r N0t}"
```
