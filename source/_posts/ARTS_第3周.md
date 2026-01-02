title: ARTS_第3周
author: ming
date: 2020-07-20 10:56:00
tags:
---

## Algorithm

## Review

## Tip
1. 注册退出进程时的回调
```c
#include <stdio.h>
#include <stdlib.h>

void exit1(void);
void exit2(void);

int main(int argc, char* argv[]) {
    if (atexit(exit1) != 0) {
        printf("attext exit1 error\n");
        return 1;
    }

    if (atexit(exit1) != 0) {
        printf("attext exit1 error\n");
        return 1;
    }

    if (atexit(exit2) != 0) {
        printf("attext exit2 error\n");
        return 1;
    }

    printf("main return\n");

    return 0;
}

void exit1() {
    printf("call exit1\n");
}

void exit2() {
    printf("call exit2\n");
}
```

2. 设置和获取环境变量
```c
#include <stdio.h>
#include <stdlib.h>

void print_env(char*);

int main(int argc, char* argv[]) {
    if (putenv("MING=9988") != 0) {
        printf("putenv error\n");
        return 1;
    }
    print_env("MING");

    if (unsetenv("MING") < 0) {
        printf("unsetenv error\n");
        return 1;
    }
    print_env("MING");

    return 0;
}

void print_env(char* name) {
    char* path = getenv(name);
    if (path) {
        printf("%s\n", path);
    } else {
        printf("not found\n");
    }
}
```

3. 异常控制setjmp/longjmp
```c
#include <stdio.h>
#include <setjmp.h>

jmp_buf jmpbuffer;

void func1();
void func2();

int main(int argc, char* argv[]) {
    int val = setjmp(jmpbuffer);
    if (val == 0) {
        printf("000 setjmp val=%d\n", val);
    } else {
        printf("setjmp val=%d\n", val);
        return 1;
    }

    func1();
    printf("main return\n");
    return 0;
}

void func1() {
    printf("call func1 start\n");
    func2();
    printf("call func1 end\n");
}

void func2() {
    printf("call func2 start\n");
    longjmp(jmpbuffer, 10);
    printf("call func2 end\n");
}
```

4. fork
```c
#include <stdio.h>
#include <unistd.h>

int main(int argc, char const *argv[])
{
    //setbuf(stdout, NULL);
    printf("start...\n");
    fflush(stdout);
    printf("nani...\n");

    int a = 0;
    pid_t pid = fork();
    if (pid == 0) {
        a = 10;
    } else {
        a--;
        sleep(2);
    }

    printf("pid=%d, a=%d\n", getpid(), a);
    return 0;
}
```

5. vfork
```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

int main(int argc, char const *argv[])
{
    printf("vfork begin");

    int a = 0;
    pid_t pid = vfork();
    if (pid < 0) {
        printf("vfork error\n");
    }

    if (pid == 0) {
        a = 10;
        //sleep(2);
        exit(0);
    }

    printf("pid=%d, a=%d\n", getpid(), a);
    return 0;
}
```

6.wait
```c
#include <stdio.h>
#include <sys/wait.h>
#include <stdlib.h>
#include <unistd.h>

void pr_exit(int status);

int main(int argc, char const *argv[])
{
    int status;
    pid_t pid = fork();    
    if (pid < 0) {
        printf("fork error\n");
    } else if (pid == 0) {
        printf("child work pid=%d\n", pid);
        //exit(4);
        //abort();
        status /= 0;
    }

    if (wait(&status) != pid) {
        printf("wait error\n");
    }

    pr_exit(status);
    return 0;
}

void pr_exit(int status) {
    if (WIFEXITED(status)) {
        printf("WIFEXITED\n");

    } else if (WIFSIGNALED(status)) {
        printf("WIFSIGNALED\n");

    } else if (WIFSTOPPED(status)) {
        printf("WIFSTOPPED\n");
        
    } else if (WIFCONTINUED(status)) {
        printf("WIFCONTINUED\n");
    }
}
```

7. waitpid
```c
#include <stdio.h>
#include <sys/wait.h>
#include <unistd.h>
#include <stdlib.h>

int main(int argc, char const *argv[])
{
    pid_t pid = fork();
    if (pid < 0) {
        printf("fork error\n");
    } else if (pid == 0) {
        printf("child proc start\n");
        sleep(4);
        printf("child proc end\n");
        // if ((pid = fork()) > 0) {
        //     exit(0);
        // } else {
        //     printf("child 2 proc start ppid=%d\n", getppid());
        //     sleep(3);
        // }
        exit(0);
    }

    int status;
    if (waitpid(pid, &status, 0) == 0) {
        printf("waitpid error\n");
    }

    printf("waitpid end ppid=%d\n", getppid());
    return 0;
}
```

8. exec
```c
// show_argc
#include <stdio.h>
#include <unistd.h>

int main(int argc, char const *argv[])
{
    for (int i=0; i<argc; i++) {
        printf("argv[%d]=%s\n", i, argv[i]);
    }

    for (char** p=__environ; *p != 0; p++) {
        printf("%s\n", *p);
    }
    return 0;
}
```

```c

#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main(int argc, char const *argv[])
{
    char* env[] = {
        "MING=333000",
        "PATH=.",
        NULL
    };
    pid_t pid = fork();
    if (pid < 0) {
        printf("fork error\n");
    } else if (pid == 0) {
        //if (execle("/home/ming/_work/apue/chapter8/show_argc", "aaaa", "bbbb", (char*)0, env) < 0) {
        if (execle("show_argc", "aaaa", "bbbb", (char*)0, env) < 0) {
            printf("execle error\n");
        }
    }

    if (waitpid(pid, NULL, 0) == 0) {
        printf("waitpid error\n");
    }

    printf("main end\n");
    return 0;
}
```



## Share

* ## 经济状态
1. 难知如阴，最近黄金冲破1800美金，感觉经济越来越动荡了。

* ## linux文件系统
1. 文件系统是什么？

2. 文件系统为什么存在？
因为unix系统把所有东西当作IO的目标，一切皆可IO，因为IO操作的目标分很多类，所以unix抽象了不同的类型的文件。文件太多了，查找和修改就会不方便，因此，要有一个管理系统，这就是文件系统。

3. 文件系统的数据结构是怎样的？
文件系统的的数据结构，主要分三部分：
   * 文件描述符表
   * 文件状态表
   * 文件inode表

* ## 很多时候，你觉得你是在写代码，但其实，你并不知道自己在干什么。