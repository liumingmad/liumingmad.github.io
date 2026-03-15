title: crackme编码保存字符串
author: ming
tags:
  - 逆向
categories:
  - 逆向
date: 2026-03-15 15:56:00
---
# ASCII_CRACK

https://crackmes.one/crackme/69a806c17b3cc38c80464e06

### 1. 下载解压
zip解压密码：crackmes.one

### 2. IDA free 
* 先看字符串
![](/images/E6BD85B420B431330997FBD3F7367D1ae.png)

* 再看结构
![](/images/E6BD85B420B431330997FBD3F7367D.png)

### 3. GDB
1. 看到结构图后，基本能猜到，关键分支
```
0x55555555644b
test    al, al

# 比较字符串
0x555555556469
call    _Z6verifyNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE ; verify(std::string)

# 核心编码函数
0x555555556413
call    _Z6encodeci 
```

2. 先跑下
```
./ascii abcd
Try again! You can do it!
```

3. 跑gdb
```
# 这个断点可以看到两个字符串在比较
b *0x555555556469

# 这个断点，决定了是做encode，还是比较编码后的结果
b *0x55555555644b

# 这里是对单个char做转化
b *0x555555556413

r abcd

```

### 4. 编码逻辑说明
```python
# 编码的逻辑
# 对字符串中的每个字符的ascii code加一个数字，第一个字符数字加6，第二个数字加5，依次类推

# 解码的逻辑
# 把编码的逻辑反过来, python代码
s = "IYJ~U4cQ1Q[<mL[(U;`'Ynk/M-i"
result = ''.join(chr(ord(c) - (6 - i)) for i, c in enumerate(s))
print(result)
```

### 5. 验证结果
```
./ascii CTF{S3cR3T_AsSc1_Fl4g}{@_@}
```
