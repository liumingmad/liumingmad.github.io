title: ARTS_第2周
author: ming
date: 2020-07-13 10:56:00
tags:
---

## Algorithm

## Review
[Exceptions in Java](https://www.geeksforgeeks.org/exceptions-in-java/)

1. java异常和错误的区别:
    * 错误是严重的，不能被try catch的
    * 异常可用被try catch, 然后继续执行的 

2. jvm怎样处理异常:
    1. 递归搜索调用栈的代码，找到匹配异常的处理代码
    2. 如果找到，就执行对应的handle
    3. 如果没找到，就执行默认的handle，默认就是在标准输出打印异常信息(异常对象里包含了异常的信息)

3. 程序员怎样处理异常
    bala bala...一些基础的废话


## Tip
1. 拷贝文件
```c
#include <unistd.h> 
#include <stdio.h>
#include <fcntl.h>
#include <stdlib.h>

#define FILE_MODE   (S_IRUSR | S_IWUSR | S_IRGRP | S_IROTH)
#define BUFFER_SIZE 4096

int main(int argc, char* argv[]) {
    int srcfd = open(argv[1], O_RDONLY);
    int destfd = open(argv[2], O_WRONLY | O_CREAT | O_TRUNC, FILE_MODE);
    printf("srcfd=%d\n", srcfd);
    printf("destfd=%d\n", destfd);

    int n;
    char buf[BUFFER_SIZE];
    while ((n = read(srcfd, buf, BUFFER_SIZE)) > 0) {
        if (write(destfd, buf, n) != n) {
            printf("write error\n");
            exit(1);
        }
    }

    if (n < 0) {
        printf("read error\n");
    }

    if (close(srcfd) == -1 || close(destfd) == -1) {
        printf("close file error\n");
    }

    exit(0);
}
```

2. 以字符查看文件
    ```
    ming@ming_server:~/_work/apue/chapter3$ od -c 11 
    0000000   n   e   w   f   d   =   1  \n   1   2   3   4   5   6   7   8
    0000020   9  \n
    0000022
    ```

3. 文件描述符重定向，demo是把标准输出重定向到argv[1]的文件
    ```c
    #include <unistd.h>
    #include <stdio.h>
    #include <fcntl.h>
    #include <stdlib.h>

    #define BUFFER_SIZE 4096

    int main(int argc, char* argv[]) {
        int fd = open(argv[1], O_RDWR);
        if (fd == -1) {
            printf("open error\n");
            exit(1);
        }

        int newfd = dup2(fd, STDOUT_FILENO);
        if (newfd == -1) {
            printf("dup2 error\n");
            exit(2);
        }

        printf("newfd=%d\n", newfd);

        int n;
        char buf[BUFFER_SIZE];
        while ((n = read(STDIN_FILENO, buf, BUFFER_SIZE)) > 0) {
            printf("%s", buf);
        } 

        if (n < 0) {
            printf("read error\n");
        }

        return 0;
    } 
    ```

4. 查看进程的虚拟内存
    ```c
    #include <stdio.h>

    int main() {
        printf("pid=%ld\n", getpid());
        getchar();
        printf("exit");
        return 0;
    }
    ```

    ```
    ming@ming_server:~/_work$ cat /proc/29858/maps 
    561c252dc000-561c252dd000 r-xp 00000000 08:02 424368                     /home/ming/_work/apue/chapter3/a.out
    561c254dc000-561c254dd000 r--p 00000000 08:02 424368                     /home/ming/_work/apue/chapter3/a.out
    561c254dd000-561c254de000 rw-p 00001000 08:02 424368                     /home/ming/_work/apue/chapter3/a.out
    561c267fd000-561c2681e000 rw-p 00000000 00:00 0                          [heap]
    7f4f196a3000-7f4f1988a000 r-xp 00000000 08:02 136752                     /lib/x86_64-linux-gnu/libc-2.27.so
    7f4f1988a000-7f4f19a8a000 ---p 001e7000 08:02 136752                     /lib/x86_64-linux-gnu/libc-2.27.so
    7f4f19a8a000-7f4f19a8e000 r--p 001e7000 08:02 136752                     /lib/x86_64-linux-gnu/libc-2.27.so
    7f4f19a8e000-7f4f19a90000 rw-p 001eb000 08:02 136752                     /lib/x86_64-linux-gnu/libc-2.27.so
    7f4f19a90000-7f4f19a94000 rw-p 00000000 00:00 0 
    7f4f19a94000-7f4f19abb000 r-xp 00000000 08:02 131706                     /lib/x86_64-linux-gnu/ld-2.27.so
    7f4f19cb2000-7f4f19cb4000 rw-p 00000000 00:00 0 
    7f4f19cbb000-7f4f19cbc000 r--p 00027000 08:02 131706                     /lib/x86_64-linux-gnu/ld-2.27.so
    7f4f19cbc000-7f4f19cbd000 rw-p 00028000 08:02 131706                     /lib/x86_64-linux-gnu/ld-2.27.so
    7f4f19cbd000-7f4f19cbe000 rw-p 00000000 00:00 0 
    7fff83fcb000-7fff83fec000 rw-p 00000000 00:00 0                          [stack]
    7fff83ff9000-7fff83ffc000 r--p 00000000 00:00 0                          [vvar]
    7fff83ffc000-7fff83ffe000 r-xp 00000000 00:00 0                          [vdso]
    ffffffffff600000-ffffffffff601000 r-xp 00000000 00:00 0                  [vsyscall]
    ```

5. fcntl, 获取文件状态标志
   ```c
   #include <stdio.h>
    #include <fcntl.h>

    int main(int argc, char* argv[]) {
        int val = fcntl(atoi(argv[1]), F_GETFL);
        if (val == -1) {
            printf("fcntl error");
            return 1;
        }

        switch (val & O_ACCMODE) {
            case O_RDONLY:
                printf("O_RDONLY");
                break;
            case O_WRONLY:
                printf("O_WRONLY");
                break;
            case O_RDWR:
                printf("O_RDWR");
                break;
            default:
                break;
        }
        printf("\n");
        getchar();
        return 0;
    } 
   ```

6. fcntl, 设置文件状态标志
    ```c
    #include <stdio.h>
    #include <fcntl.h>

    int main(int argc, char* argv[]) {
        int fd = open(argv[1], O_RDWR);
        if (fd == -1) {
            printf("open error");
            return 0;
        }

        set_fl(fd, O_APPEND);

        int res = dprintf(fd, "aaaaa\n");
        printf("res=%d\n", res);
        return 0;
    }

    void set_fl(int fd, int flags) {
        int val = fcntl(fd, F_GETFL);
        printf("get=%d\n", val);
        if (val == -1) {
            printf("get error");
            return 0;
        }

        val |= flags; 
        printf("val=%d\n", val);
        int res = fcntl(fd, F_SETFL, val);
        if (res == -1) {
            printf("set error");
        }
    }
    ```
7. 如果以O_APPEND打开文件，lseek只对读生效，每次写，会自动seek到文件末尾
    ```c
    #include <stdio.h>
    #include <fcntl.h>
    #include <unistd.h>

    #define BUFFERSIZE 4096

    int main(int argc, char* argv[]) {
        int fd;
        if ((fd = open(argv[1], O_RDWR | O_APPEND)) == -1) {
            printf("open error\n");
            return 1;
        }

        char buf[BUFFERSIZE];
        int n = read(fd, buf, BUFFERSIZE);
        if (n < 0) {
            printf("read error\n");
            return 2;
        }
        printf("pre seek: %s\n", buf);

        if (lseek(fd, 10, SEEK_SET) == -1) {
            printf("lseek error");
            return 3;
        } 

        n = read(fd, buf, BUFFERSIZE);
        if (n < 0) {
            printf("read error\n");
            return 2;
        }
        printf("after seek: %s\n", buf);

        dup2(fd, STDOUT_FILENO);

        printf("\nnani\n");

        return 0;
    }
    ```

8. 获取文件类型
```c
#include <stdio.h>
#include <sys/stat.h>

int main(int argc, char* argv[]) {
    struct stat buf;
    for (int i=1; i<argc; i++) {
        printf("%s: ", argv[i]);
        if (stat(argv[i], &buf) < 0) {
            printf("lstat error");
            continue;
        }

        if (S_ISREG(buf.st_mode)) {
            printf("regular");
        } else if (S_ISDIR(buf.st_mode)) {
            printf("dir");
        } else if (S_ISCHR(buf.st_mode)) {
            printf("chr");
        } else if (S_ISBLK(buf.st_mode)) {
            printf("blk");
        } else if (S_ISFIFO(buf.st_mode)) {
            printf("fifo");
        } else if (S_ISLNK(buf.st_mode)) {
            printf("link");
        } else if (S_ISSOCK(buf.st_mode)) {
            printf("sock");
        } else {
            printf("other");
        }
        printf("\n");
    }
    return 0;
}
```

9. 判断是否有访问权限
```c
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>

int main(int argc, char* argv[]) {
    if (access(argv[1], R_OK) == 0) {
        printf("read ok\n");
    } else {
        printf("access error\n");
    }

    if (open(argv[1], O_RDONLY) < 0) {
        printf("open error\n");
    } else {
        printf("open ok\n");
    }
    return 0;
}
```

10. 文件权限掩码
```c
#include <stdio.h>
#include <fcntl.h>
#include <sys/stat.h>

#define RWRWRW (S_IRUSR | S_IWUSR | S_IRGRP | S_IWGRP | S_IROTH | S_IWOTH)

int main(int argc, char* argv[]) {
    umask(0);
    if (creat("foo", RWRWRW) < 0) {
        printf("foo error");
    }

    umask(S_IRGRP | S_IROTH | S_IWOTH);
    if (creat("bar", RWRWRW) < 0) {
        printf("bar error");
    }

    return 0;
}
```

11. 文件系统的数据结构
    主要分4个部分:
    1. 文件描述符表, 包括索引和文件表的指针 
    2. 文件表, 字段包括文件状态标记，文件当前偏移，inode指针
    3. inode节点，包括文件权限，block指针，读写block的时间，修改inode的时间
    4. block节点，主要包含数据

12. 标准库的io
    标准库的io主要是对系统调用的封装，之所以要封装，是为了效率，因为每次系统调用的开销太大。封装主要是加了缓冲，分为三种缓冲：
    * 全缓冲
    * 行缓冲
    * 不缓冲
    只有满足flush的条件或缓冲区满的时候，才调用系统调用，写入内核的缓冲。

13. 内存其实也可用被当作文件，进行读写，定位等。
```c
#include <stdio.h>
#include <string.h>

#define BUFFER_SIZE 4096

int main(int argc, char* argv) {
    char buf[BUFFER_SIZE];
    memset(buf, '\0', BUFFER_SIZE);

    FILE* fp = fmemopen(buf, BUFFER_SIZE, "w+");
    if (!fp) {
        printf("fmemopen error");
        return 1;
    }

    printf("111->%s\n", buf);
    getchar();

    fprintf(fp, "hello world");
    fflush(fp);
    printf("222->%s\n", buf);

    getchar();

    return 0;
}
```
## Share
* 到目前为止(看完了前5章)，apue这本书中，主要讲了IO，包括文件操作，目录操作，内核中文件的数据结构，文件的权限和用户属性等，层次主要包括：系统调用和标准库的实现。
* 其实对于一个unix进程来说，能干的事，全都包括在系统调用里，而IO是很重要的一部分，因为对于unix来说，所有跟读写相关的，全都被抽象成文件，比如：磁盘，内存，网卡等。之所以设计成这样，就是为了简化编程，统一成一类接口。其实，对于这种抽象也很好理解，就是把纷繁复杂的东西，做分类，然后，把自己要干的事，做成几个基本的原子操作。