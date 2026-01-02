title: c语言
author: ming
date: 2020-08-17 10:56:00
tags:
---
## 1.编译
```c
// hello.c
#include <stdio.h>

typedef struct test
{
    int a;
    int b;
} T;

int main(int argc, char const *argv[])
{
    T t1;
    printf("t1=%d, %d, %ld\n", t1.a, t1.b, &t1);

    T t2 = t1;
    printf("t2=%d, %d, %ld\n", t2.a, t2.b, &t2);
    return 0;
}
```

```bash
# 头文件所在目录
find / -name "stdio.h" 2>/dev/null
/usr/include/stdio.h
/usr/include/c++/9/tr1/stdio.h
/usr/include/x86_64-linux-gnu/bits/stdio.h

# 编译hello.c
# 编译时，使用/usr/include/stdio.h替换#include<stdio.h>
# 链接时，自动链接
gcc hello.c -o hello

```



## typedef还能这么写?
```c
typedef const struct JNIInvokeInterface_ *JavaVM;
```

## 直接生成预编译的文件
```bash
gcc -I "/System/Library/Frameworks/JavaVM.framework/Versions/Current/Headers" -E hello.c -o hello.i
```

## static
```
静态变量保存在数据区
用来限制作用域
```








