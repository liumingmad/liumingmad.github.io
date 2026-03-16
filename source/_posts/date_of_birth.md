title: crackme-Date of Birth
author: ming
tags:
  - 逆向
categories:
  - 逆向
date: 2026-03-16 23:04:00
---

https://crackmes.one/crackme/5f4efbb133c5d4357b3b00a0

## 基本信息

| 项目 | 描述 |
|------|------|
| 文件 | `date_of_birth` |
| 类型 | ELF 64-bit LSB PIE executable, x86-64 |
| 链接 | dynamically linked |
| 编译器 | GCC 7.5.0 (Ubuntu 18.04) |
| 符号表 | stripped |

## 初步分析

### strings 提取

```
Welcome to Discard App -- a new way to get screwed up
We need to verify your age before you can usage the app
Please type you date of birth here:
You are too young! We are sorry but you cannot use Discard app at the moment
Congrats! Now you can use Discard app!
%20s
%m/%d/%Y
```

程序是一个虚构的 "Discard App"（Discord 谐音梗），要求输入生日来验证年龄。输入格式为 `%m/%d/%Y`（如 `03/17/1898`）。

### 导入函数

- `scanf` — 读取用户输入
- `strptime` — 按 `%m/%d/%Y` 格式解析日期字符串
- `mktime` — 将 `struct tm` 转换为 `time_t` 时间戳
- `time` — 获取当前时间
- `localtime` — 将 `time_t` 转换回 `struct tm`
- `puts` — 输出字符串

## 逆向分析

### 主函数流程 (0x710)

反编译后的伪代码：

```c
int main(int argc, char **argv) {
    char input[24];       // rsp+0x70
    struct tm tm_birth;   // rsp+0x30

    puts("Welcome to Discard App -- a new way to get screwed up");
    puts("We need to verify your age before you can usage the app");
    puts("Please type you date of birth here: ");

    memset(input, 0, 16);
    scanf("%20s", input);

    memset(&tm_birth, 0, sizeof(tm_birth));
    strptime(input, "%m/%d/%Y", &tm_birth);

    time_t birth_ts = mktime(&tm_birth);       // 出生日期时间戳
    time_t now = time(NULL);                    // 当前时间戳
    time_t diff = now - birth_ts;               // 时间差

    struct tm *lt = localtime(&diff);           // 关键：对时间差调用 localtime

    int mday  = lt->tm_mday;   // r12d
    int year  = lt->tm_year;   // ebx
    int mon   = lt->tm_mon;    // ebp

    int al  = (mday + 1) & 0xFF;
    int age = year - 70;       // ebx -= 0x46
    int edx = age;

    // ... 年龄验证逻辑 ...
}
```

### 关键设计：用 localtime 计算"年龄"

程序并未用常规方式（年份相减）计算年龄，而是：

1. 计算 `diff = time(NULL) - mktime(birth_date)`（秒级时间差）
2. 对 `diff` 调用 `localtime()`，将其当作一个绝对时间戳来解析
3. 从返回的 `struct tm` 中提取 `tm_year`、`tm_mon`、`tm_mday`
4. 用 `tm_year - 70` 作为 "年龄"（因为 `tm_year` 是自 1900 年起的年数，减去 70 即以 1970 年/Unix 纪元为基准）

### 年龄验证的控制流

反汇编的核心比较逻辑：

```
0x7fb:  lea    0x1(%r12),%eax       ; eax = mday + 1
0x800:  sub    $0x46,%ebx           ; ebx = tm_year - 70 (即 "age")
0x803:  mov    %ebx,%edx            ; edx = age
0x805:  cmp    $0x1e,%al            ; if (mday+1) <= 30 ?
0x807:  jle    0x853                ;   → Path A
                                    ; else → Path B (fall through)

; ---- Path B: mday >= 30 ----
0x809:  lea    0x1(%rbp),%ecx       ; ecx = mon + 1
0x80e:  cmp    $0xb,%cl             ; if (mon+1) > 11 ?
0x811:  jg     0x859                ;   → Path B2

; ---- Path B2: mon >= 11 (December) ----
0x859:  lea    0x1(%rbx),%ecx       ; ecx = age + 1
0x85c:  cmp    %dl,%cl              ; compare (age+1) vs age (as bytes!)
0x85e:  jg     0x81c                ; if (age+1) > age → FAIL
0x860:  jge    0x8ad                ; if (age+1) == age → other path
                                    ; fall through → SUCCESS (0x862)
```

控制流图：

```
                     入口
                      │
              ┌───────┴───────┐
              │  mday+1 <= 30 │
              │   (Path A)    │
              ▼               ▼
           [0x853]         [0x809]
           mon<=11?        mon+1>11?
              │               │
         ┌────┴────┐     ┌────┴────┐
         ▼         ▼     ▼         ▼
      [0x89e]   [0x859] [0x813]  [0x859]
      mday>=     age+1   mon>=    age+1
      mday+1?    vs age  mon+1?   vs age
         │         │       │        │
    ┌────┴────┐  ┌─┴──┐  ┌┴───┐  ┌─┴──┐
    ▼         ▼  ▼    ▼  ▼    ▼  ▼    ▼
 SUCCESS    FAIL S    F  89c  F  S    F
                         │
                    ┌────┴────┐
                    ▼         ▼
                 SUCCESS    [0x89e]
```

## 漏洞：有符号字节溢出

所有通往 SUCCESS 的路径都要求某个值 **大于其自身加一**，这在正常情况下不可能。但程序使用 **字节级比较**（`%al`、`%cl`、`%dl`、`%bpl`、`%r12b`），这就引入了**有符号整数溢出**。

### Path B2 的溢出利用

关键比较在 `0x85c`：

```nasm
lea    0x1(%rbx),%ecx    ; ecx = age + 1 (32位运算)
cmp    %dl,%cl           ; 但只比较低 8 位！(有符号字节)
jg     FAIL              ; if cl > dl (signed byte)
```

当 `age = 127 (0x7F)` 时：

| 变量 | 值 | 有符号字节解释 |
|------|----|----------------|
| `dl` (age) | `0x7F` | **+127** |
| `cl` (age+1) | `0x80` | **-128** |

比较结果：`cl (-128) > dl (127)` → **False！** `jg` 不跳转。

`cl (-128) >= dl (127)` → **False！** `jge` 也不跳转。

于是 **fall through 到 SUCCESS (0x862)**！

### 触发条件

要到达 Path B2，还需要满足：
- `tm_mday >= 30`（进入 Path B）
- `tm_mon >= 11`（即 12 月，进入 Path B2）
- `tm_year - 70 = 127`（触发字节溢出）

因此 `localtime(diff)` 必须返回：

| 字段 | 所需值 | 含义 |
|------|--------|------|
| `tm_year` | 197 | 年份 2097（1900+197） |
| `tm_mon` | 11 | 12 月（0 索引） |
| `tm_mday` | >= 30 | 30 或 31 日 |

即时间差 `diff` 对应的绝对日期落在 **2097 年 12 月 30-31 日**。

## 求解

编写 C 程序暴力搜索满足条件的出生日期：

```c
#include <stdio.h>
#include <time.h>
#include <string.h>

int main() {
    time_t now = time(NULL);

    for (int year = 1895; year <= 1905; year++) {
        for (int month = 1; month <= 12; month++) {
            for (int day = 1; day <= 28; day++) {
                struct tm tm_birth = {0};
                char buf[32];
                snprintf(buf, sizeof(buf), "%02d/%02d/%04d", month, day, year);
                strptime(buf, "%m/%d/%Y", &tm_birth);

                time_t birth_ts = mktime(&tm_birth);
                time_t diff = now - birth_ts;
                struct tm *lt = localtime(&diff);
                if (!lt) continue;

                int age_val = lt->tm_year - 70;
                if (age_val == 127 && lt->tm_mon >= 11 && lt->tm_mday >= 30) {
                    printf("MATCH: %s -> age=%d, mon=%d, mday=%d\n",
                           buf, age_val, lt->tm_mon, lt->tm_mday);
                }
            }
        }
    }
    return 0;
}
```

运行结果（2026-03-16）：

```
MATCH: 03/17/1898 -> age=127, mon=11, mday=31
MATCH: 03/18/1898 -> age=127, mon=11, mday=30
```

> 注意：正确日期与**运行时的当前时间**有关，不同日期运行结果会不同。

## Flag 生成函数 (0x9d0)

通过验证后，程序用日期信息构造 XOR 密钥来解密内置的编码字符串：

### 密钥构造

```nasm
; 打包 tm 值到 rdi
movzbl %bl,%edi           ; edi = (tm_year-70) & 0xFF = 0x7F
movzbl %bpl,%ebx
shl    $0x8,%rbx          ; ebx = tm_mon << 8
or     %rdi,%rbx          ; rbx = (tm_mon << 8) | age
movzbl %r12b,%edi
shl    $0x10,%rdi         ; rdi = tm_mday << 16
or     %rbx,%rdi          ; rdi = (mday << 16) | (mon << 8) | age
call   0x9d0
```

### 解密逻辑

```nasm
0x9d0:  mov    %edi,%edx           ; edx = packed_value
0x9d2:  mov    %edi,%eax           ; eax = packed_value
0x9d4:  mov    %edi,%r8d           ; r8d = packed_value
0x9d7:  movzbl %dh,%edx            ; edx = byte1 = tm_mon (0x0B)
0x9da:  shr    $0x10,%r8d          ; r8d = byte2 = tm_mday (0x1F)
0x9e2:  xor    %edx,%eax           ; eax = packed ^ tm_mon
0x9f1:  xor    %eax,%r8d           ; r8d = tm_mday ^ eax → r8b = XOR key

; 对每个字节做 XOR 解密
0xa78:  xor    %r8b,(%rdi,%rsi,1)  ; buf[i] ^= key
0xa7c:  add    $0x1,%rsi
```

### 密钥计算

对于 `mday=31, mon=11, age=127`：

```
packed  = 0x001F0B7F
edx     = 0x0B (tm_mon)
eax     = 0x001F0B7F ^ 0x0B = 0x001F0B74
r8d     = 0x1F ^ 0x001F0B74 = 0x001F0B6B
r8b     = 0x6B  ← 最终 XOR 密钥
```

### 编码数据

函数中硬编码了以下数据（小端序排列）：

```
0x10: 3e 05 0d 04 19 1f 1e 05
0x18: 0a 1f 0e 07 12 4b 1f 03
0x20: 02 18 4b 0a 1b 1b 4b 03
0x28: 0a 18 4b 0a 09 18 04 07
0x30: 1e 1f 0e 07 12 4b 05 04
0x38: 4b 0d 1e 05 08 1f 02 04
0x40: 05 0a 07 02 1f 12 00
```

每个字节 XOR `0x6B`：

```python
data = [0x3e,0x05,0x0d,0x04,0x19,0x1f,0x1e,0x05,
        0x0a,0x1f,0x0e,0x07,0x12,0x4b,0x1f,0x03,
        0x02,0x18,0x4b,0x0a,0x1b,0x1b,0x4b,0x03,
        0x0a,0x18,0x4b,0x0a,0x09,0x18,0x04,0x07,
        0x1e,0x1f,0x0e,0x07,0x12,0x4b,0x05,0x04,
        0x4b,0x0d,0x1e,0x05,0x08,0x1f,0x02,0x04,
        0x05,0x0a,0x07,0x02,0x1f,0x12,0x00]
print("".join(chr(b ^ 0x6B) for b in data))
```

## 运行结果

```
$ echo "03/17/1898" | ./date_of_birth
Welcome to Discard App -- a new way to get screwed up
We need to verify your age before you can usage the app
Please type you date of birth here:
Congrats! Now you can use Discard app!
Unfortunately this app has absolutely no functionality
```

**Flag: `Unfortunately this app has absolutely no functionality`**

## 总结

| 要素 | 详情 |
|------|------|
| 漏洞类型 | 有符号字节溢出（Signed Byte Overflow） |
| 触发条件 | `tm_year - 70 = 127 (0x7F)`，使 `(age+1)` 的低字节溢出为 `0x80 (-128)` |
| 所需输入 | 约 1898 年的出生日期（精确值取决于运行时间） |
| 解密方式 | 单字节 XOR，密钥由 `tm_mday`、`tm_mon`、`tm_year` 异或派生 |
