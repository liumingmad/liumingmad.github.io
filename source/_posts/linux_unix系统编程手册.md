title: linux-unix系统编程手册
author: ming
tags:
  - linux
  - unix
  - syscall
  - ''
categories:
  - linux
date: 2021-05-05 09:55:00
---

### 0. 前置
```shell
0. 因为想了解linux的系统调用，所以查到这本书
1. 下载这本书的pdf
2. 快速扫过前三章，基本无感，当看到第4章，看到有代码，于是然后看到#include "tlpi_hdr.h", 所以google到了官网
https://www.man7.org/tlpi/index.html
3. 下载源码并编译
    1. 在根目录直接敲make，提示没../libtlpi.a, 官网说要先在lib目录下make
    2. 在lib目录下make，提示没有头文件userns_functions.c:25:10: fatal error: sys/capability.h: No such file or directory, 解决办法在FAQ中，apt install libcap-dev
    3. 再次在根目录make
4. 再次回到，./fileio/copy.c的代码，很奇怪，为什么#include "tlpi_hdr.h" 
没带全路径，因为它不在当前目录，这他妈是怎么找到的? 
看了Makefile，实在看不懂，于是找了个教程
https://seisman.github.io/how-to-write-makefile/Makefile.pdf
5. 苦逼的翻过80页的教程，再去看项目里的Makefile。才知道，CFLAGS中，包含查找头文件的目录 -I${TLPI_INCL_DIR}, 由于隐式规则，copy这个目标的命令应该用的cc –c $(CFLAGS) copy.c
6. 终于可以愉快的开始了，fucking！！
7. 又有问题，在我自己的demo目录，提示链接失败，因此又配置了
CFLAGS = -std=c99 -D_XOPEN_SOURCE=600 \
	        -D_DEFAULT_SOURCE \
		-g -I${TLPI_INCL_DIR} \
		-L${TLPI_LIB} \
		-pedantic \
		-Wall \
		-W \
		-Wmissing-prototypes \
		-Wno-sign-compare \
		-Wimplicit-fallthrough \
		-Wno-unused-parameter
LDLIBS = -ltlpi
```

### 4. 文件I/O:通用的I/O模型
```
// open, read, write, close
// lseek
// ioctl
```

```c
// 文件拷贝
// open, read, write, close
#include <string.h>
#include <sys/stat.h>
#include <fcntl.h>
#include "tlpi_hdr.h"

int main(int argc, char* argv[]) {
    if (argc != 3) {
        usageErr("error:%d\n", argc);
    }

    if (strcmp(argv[1], "--help") == 0) {
        usageErr("error: No support '--help'\n");
    }

    int ifd;
    int ofd;

    ifd = open(argv[1], O_RDONLY);
    if (ifd < 0) {
        errExit("error: open fd=%d fail!", ifd);
    }

    int filePerms = S_IRUSR | S_IWUSR;
    ofd = open(argv[2], O_WRONLY | O_CREAT | O_APPEND, filePerms);
    if (ofd < 0) {
        errExit("error: open fd=%d fail!", ofd);
    }

    int readcount = 0;
    char buf[1024];
    while ((readcount = read(ifd, buf, 1024)) > 0) {
        printf("readcount=%d\n", readcount);
        if (write(ofd, buf, readcount) != readcount) {
            fatal("error: fatal");
        }
    }

    if (readcount == -1) {
        errExit("read");
    }

    if (close(ifd) != 0) {
        errExit("error: close error fd=%d\n", ifd);
    }

    if (close(ofd) != 0) {
        errExit("error: close error fd=%d\n", ofd);
    }

    return 0;
}
```

```c
// tee test
// test -a test
#include <sys/stat.h>
#include <fcntl.h>
#include <string.h>
#include "tlpi_hdr.h"

#define BUF_SIZE 1024

int main(int argc, char* argv[]) {
    if (argc > 3) {
        usageErr("argv too long");
    }

    int ifd = 0;
    int ofd = 1;
    if (argc >= 2) {
        char* filename = argv[1];
        int mode = O_CREAT | O_WRONLY;
        if (argc == 3 && strcmp(argv[1], "-a") == 0) {
            filename = argv[2];
            mode |= O_APPEND;
        }

        int perms = S_IRUSR | S_IWUSR | S_IRGRP | S_IWGRP;
        if ((ofd = open(filename, mode, perms)) == -1) {
            errExit("open file %s failed!", filename);
        }
    }

    ssize_t numRead;
    char buf[BUF_SIZE];
    while ((numRead = read(ifd, buf, BUF_SIZE)) > 0) {
        if (write(ofd, buf, numRead) != numRead) {
            fatal("write");
        }
    }

    if (numRead == -1) {
        errExit("read");
    }

    if (close(ifd) == -1) {
        errExit("close ifd");
    }

    if (close(ofd) == -1) {
        errExit("close ofd");
    }

    return 0;
}
```

```c
// 文件空洞
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include "tlpi_hdr.h"

void writeText(int, char*);

int main(int argc, char* argv[]) {
    int fd = open("hole", O_CREAT | O_WRONLY, S_IRUSR | S_IWUSR);
    if (fd == -1) {
        errExit("open");
    }

    writeText(fd, "hello");
    if (lseek(fd, 10000, SEEK_CUR) == -1) {
        errExit("lseek");
    }
    writeText(fd, "world");

    if (close(fd) == -1) {
        errExit("close");
    }

    return 0;
}

void writeText(int fd, char* buf) {
    int size = strlen(buf);
    if (write(fd, buf, size) != size) {
        errExit("write");
    }
}
```

### 5. 文件操作，更多特性
1. O_EXCL | O_CREAT, 保证了创建和打开的原子性，如果打开的文件以经存在，则会报错. 场景：当两个进程同时打开一个fd时

2. O_APPEND, 保证了seek和write的原子性

3. 获取／设置一个已经打开的fd的标记位
```c
#include <sys/stat.h>
#include <fcntl.h>
#include "tlpi_hdr.h"

void lookup(int);

int main(int argc, char* argv[]) {
    int fd = open("fcntl.c", O_WRONLY);
    if (fd == -1) {
        errExit("open");
    }

    int flags;
    flags = fcntl(fd, F_GETFL);
    if (flags == -1) {
        errExit("fnctl");
    }
    lookup(flags);

    flags |= O_APPEND;
    if (fcntl(fd, F_SETFL, flags) == -1) {
        errExit("fnctl");
    }

    flags = fcntl(fd, F_GETFL);
    if (flags == -1) {
        errExit("fnctl");
    }
    lookup(flags);

    return 0;
}

void lookup(int flags) {

    if (flags & O_SYNC) {
        printf("O_SYNC is open\n");
    }

    if (flags & O_APPEND) {
        printf("O_APPEND is open\n");
    }

    int accmode = flags & O_ACCMODE;
    if (accmode == O_RDONLY) {
        printf("O_RDONLY is open\n");
    }

    if (accmode == O_WRONLY) {
        printf("O_WRONLY is open\n");
    }

    if (accmode == O_RDWR) {
        printf("O_RDWR is open\n");
    }

    printf("----finish\n");
}
```

### 4. 复制fd(dup, dup2, dup3)
```c
// 把标准输出重定向到日志文件
#include <sys/stat.h>
#include <fcntl.h>
#include "tlpi_hdr.h"

int main(int argc, char* argv[]) {
    int perms = S_IRUSR | S_IWUSR | S_IRGRP | S_IWGRP;
    int ofd = open(argv[1], O_CREAT | O_WRONLY | O_APPEND, perms);
    if (ofd == -1) {
        errExit("open");
    }

    if (close(1) == -1) {
        errExit("close");
    }

    if (dup2(ofd, 1) == -1) {
        errExit("dup");
    }

    printf("hello world dup2\n");
    return 0;
}
```

### 5. 带偏移的原子读写, pread/pwrite
```c
// 
#include <sys/stat.h>
#include <fcntl.h>
#include <string.h>
#include "tlpi_hdr.h"

int main(int argc, char* argv[]) {
    int flags = O_CREAT | O_EXCL | O_RDWR;
    int perms = S_IRUSR | S_IWUSR | S_IRGRP | S_IRGRP;
    int fd = open(argv[1], flags, perms);
    if (fd == -1) {
        errExit("open");
    }

    char* buf = "hello";
    if (pwrite(fd, buf, strlen(buf), 100) == -1) {
        errExit("pwrite");
    }

    char* buf1 = malloc(10);
    if (pread(fd, buf1, 10, 101) == -1) {
        errExit("pread");
    }
    printf("pread:%s\n", buf1);
    return 0;
}
```

### 6. 分散缓冲区的集中原子读写，readv/writev
```c
#include <sys/stat.h>
#include <fcntl.h>
#include <sys/uio.h>
#include "tlpi_hdr.h"

struct Cat {
    char name[10];
    int age;
};

void try_write(int fd);
void try_read(int fd);

int main(int argc, char* argv[]) {
    if (argc != 3) {
        usageErr("Error: argc is not 3\n");
    }

    int flags = O_CREAT | O_RDWR;
    int perms = S_IRUSR | S_IWUSR | S_IRGRP | S_IWGRP;
    int fd = open(argv[2], flags, perms);
    if (fd == -1) {
        errExit("open");
    }

    if (strcmp(argv[1], "r") == 0) {
        try_read(fd);
    } else {
        try_write(fd);
    }

    if (close(fd) == -1) {
        errExit("close");
    }
    return 0;
}

void try_write(int fd) {
    struct iovec data[2];

    struct Cat cat = { "mimi", 10 };
    data[0].iov_base = &cat;
    data[0].iov_len = sizeof(struct Cat);

    int x = 99;
    data[1].iov_base = &x;
    data[1].iov_len = sizeof(int);

    if (writev(fd, data, 2) == -1) {
        errExit("writev");
    }
}

void try_read(int fd) {
    struct iovec data[2];

    struct Cat cat;
    data[0].iov_base = &cat;
    data[0].iov_len = sizeof(struct Cat);

    int x = 0;
    data[1].iov_base = &x;
    data[1].iov_len = sizeof(int);

    if (readv(fd, data, 2) == -1) {
        errExit("readv");
    }

    printf("cat:%s, %d\n", cat.name, cat.age);
    printf("x:%d\n", x);
}
```

### 7. 设置文件大小，多退少补，truncate/ftruncate
```c
#include <unistd.h>
#include <stdio.h>

int main(int argc, char* argv[]) {
    if (truncate(argv[1], 1024) == -1) {
        printf("error");
    }
    return 0;
}
```

### 8. 非阻塞IO标记，O_NONBLOCK
```c
// 如果陷入阻塞，则直接返回错误EAGAIN
```

### 9. 大文件标记，O_LARGEFILE
```c
// 如果open的文件大于2GB，且没有此标记，则直接报错
```

### 10. 临时文件，mkstemp/tmpfile
```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/stat.h>
#include <unistd.h>
#include <string.h>
#include "tlpi_hdr.h"

void try_mkstemp();
void try_tmpfile();

int main(int argc, char* argv[]) {
    try_tmpfile();
    return 0;
}

void try_mkstemp() {
    // XXXXXX必须大写
    char name[] = "/tmp/tmpfileXXXXXX";
    int fd;
    if ((fd = mkstemp(name)) == -1) {
        errExit("error mkstemp");
    }

    printf("tmp name:%s\n", name);
    unlink(name);

    if (close(fd) == -1) {
        errExit("error mkstemp");
    }
}

void try_tmpfile() {
    FILE* f = tmpfile();
    int fd = f->_fileno;
    printf("fd:%d\n", fd);


    char* buf = "helloabcdefghijklmn";
    if (write(fd, buf, strlen(buf)) == -1) {
        errExit("write");
    }

    int pos;
    if ((pos = lseek(fd, 0, SEEK_SET)) == -1) {
        errExit("lseek");
    }

    int numread;
    char* rbuf = malloc(10);
    if ((numread = read(fd, rbuf, 10)) == -1) {
        errExit("read");
    }

    printf("rbuf:%d %s\n", numread, rbuf);

    if (close(fd) == -1) {
        errExit("close");
    }
}
```
### 11. getpid/getppid
```c
如果一个进程的父进程被杀，则子进程归属到init进程
```

### 12. getenv/setenv/clearenv/
```c
#include "tlpi_hdr.h"

extern char** environ;

int main(int argc, char* argv[], char* envp[]) {
    // 清除所有环境变量
    clearenv();
    for (char** one=environ; one&&*one; one++) {
        printf("%s\n", *one);
    }

    if (setenv("ming", "hello", 1) == -1) {
        errExit("setenv");
    }
    printf("----->%s\n", getenv("ming"));
    if (unsetenv("ming") == -1) {
        errExit("unsetenv");
    }
    printf("----->%s\n", getenv("ming"));

    return 0;
}
```

### 13. setjmp/longjmp
```c
#include "tlpi_hdr.h"
#include <setjmp.h>

jmp_buf env;

void f2(void) {
    longjmp(env, 2);
    printf("f2 finish\n");// 不会打印
}

void f1(int argc) {
    if (argc == 1) {
        longjmp(env, 1);
    }
    f2();
    printf("f1 finish\n");// 不会打印
}

int main(int argc, char* argv[]) {
    printf("setjmp\n");
    switch(setjmp(env)) {
        case 0:
            printf("first jmp\n");
            f1(argc);
            break;
        case 1:
            printf("second jmp from f1\n");
            break;
        case 2:
            printf("third jmp from f2\n");
            break;
    }
    printf("finish\n");
    return 0;
}
```

### 14. brk/sbrk
```c
#include <stdio.h>
#include <unistd.h>

int main(int argc, char* argv[]) {
    void* p1 = sbrk(0);
    printf("cur=%p\n", p1);

    //char* buf = malloc(0x10);
    brk(p1+1024);

    void* p2 = sbrk(0);
    printf("cur=%p\n", p2);

    printf("dx=%ld\n", (p2-p1));
    return 0;
}

//cur=0x55e333f69000
//cur=0x55e333f69400
//dx=1024
```

### 15. malloc/free
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(int argc, char* argv[]) {
    printf("cur=%p\n", sbrk(0));

    void* buf = malloc(100);
    printf("cur=%p\n", sbrk(0));

    void* buf1 = malloc(200);
    printf("cur=%p\n", sbrk(0));

    return 0;
}

//cur=0x55608a112000
//cur=0x55608a133000
//cur=0x55608a133000
```

### 16. calloc/realloc


### 17. getpwnam/getpwuid/getgrnam/getgrgid
```c
#include <stdio.h>
#include <pwd.h>

int main(int argc, char* argv[]) {
    struct passwd* pwd = getpwnam("ming");
    printf("uid:%d\n", pwd->pw_uid);
    printf("name:%s\n", pwd->pw_name);
    printf("dir:%s\n", pwd->pw_dir);
    printf("shell:%s\n", pwd->pw_shell);

    struct passwd* pwd1 = getpwuid(pwd->pw_uid);
    printf("name:%s\n", pwd1->pw_name);
    printf("dir:%s\n", pwd->pw_dir);
    printf("shell:%s\n", pwd->pw_shell);
    return 0;
}
```

### 18. 登陆校验，crypt/getpass
```c
#include <stdio.h>
#include "tlpi_hdr.h"
#include <pwd.h>
#include <string.h>
#include <shadow.h>
#include <unistd.h>
#include <crypt.h>

int main(int argc, char* argv[]) {

    printf("username:");

    char* username = malloc(8);
    memset(username, 0, 8);
    if (fgets(username, 8, stdin) == NULL) {
        errExit("fgets");
    }

    int len = strlen(username);
    if (username[len-1] == '\n') {
        username[len-1] = '\0';
    }

    struct passwd* pwd = getpwnam(username);
    if (pwd == NULL) {
        errExit("getpwnam");
    }
    struct spwd* sp = getspnam(pwd->pw_name);
    if (sp == NULL) {
        errExit("getpwnam");
    }
    printf("sp=%s\n", sp->sp_pwdp);

    char* password = getpass("password:");
    char* encrypted = crypt(password, sp->sp_pwdp);
    if (0 == strcmp(encrypted, sp->sp_pwdp)) {
        printf("welcome %s\n", username);
    } else {
        printf("username or password error!\n");
    }
    return 0;
}
```

### 19. 实现getpwnam()
```c
#include <stdio.h>
#include <pwd.h>
#include <string.h>

struct passwd *getpwnam(const char *name);

int main(int argc, char* argv[]) {
    struct passwd *pwd = getpwnam("ming");
    printf("dir=%s\n", pwd->pw_dir);
    return 0;
}

struct passwd *getpwnam(const char *name) {
   struct passwd *pwd = NULL;
   while ((pwd = getpwent()) != NULL) {
       if (strcmp(name, pwd->pw_name) == 0) {
           break;
       }
   }
   endpwent();
   return pwd;
}
```

### 20. setresuid/setfsuid
```c
// 1. 实际用户ID
// 2. 有效用户ID
// 3. 保存用户ID
// 4. 文件系统用户ID


// aaa文件，只对1000用户有读写权限
// bbb文件，只对1001用户有读写权限
// 如果有效用户ID是1001，则无法open aaa

#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/stat.h>
#include "tlpi_hdr.h"

void show_resuid();
void try_read_file(const char* name);

int main(int argc, char* argv[]) {
    show_resuid();

    try_read_file("aaa");
    try_read_file("bbb");

    // 修改有效用户id
    if (setresuid(-1, 1001, -1) == -1) {
        errExit("setresuid");
    }
    show_resuid();
    try_read_file("aaa");
    try_read_file("bbb");

    // 尝试改回root, 如果ruid/euid/suid中有一个是0，则可用改回root，否则会失败
    if (setresuid(-1, 1001, -1) == -1) {
        errExit("setresuid");
    }
    show_resuid();
    return 0;
}

void try_read_file(const char* name) {
    printf("read %s\n", name);
    int fd = open(name, O_RDONLY);
    if (fd == -1) {
        errMsg("open");
        return;
    }

    char buf[10];
    if (read(fd, buf, 10) == -1) {
        errMsg("read");
    }
    printf("read file=%s\n", buf);

    if (close(fd) == -1) {
        errExit("close");
    }
}

void show_resuid() {
    // 获取实际用户id，有效用户id，保存用户id
    uid_t ruid, euid, suid;
    if (getresuid(&ruid, &euid, &suid) == -1) {
        errExit("getresuid");
    }
    printf("%d, %d, %d\n", ruid, euid, suid);
}
```

### 21. 日历时间转换 
```c
#include <stdio.h>
#include <time.h>
#include <sys/time.h>
#include "tlpi_hdr.h"

#define TIME_LEN 100

int main(int argc, char* argv[]) {
    struct timeval tv;
    int r = gettimeofday(&tv, NULL);
    if (r == -1) {
        errMsg("gettimeofday");
    }
    printf("cal=%ld, %ld\n", tv.tv_sec, tv.tv_usec);

    // 获取当前UTC秒数
    time_t t = time(NULL);
    printf("t=%ld\n", t);

    // Sun May  9 10:11:32 2021
    char* timestr = ctime(&t);
    printf("%s\n", timestr);

    // 以GMT时区分解时间2:18
    struct tm *tmp = gmtime(&t);
    printf("hour=%d:%d\n", tmp->tm_hour, tmp->tm_min);

    // 以本地时区分解时间10:18
    struct tm *ltmp = localtime(&t);
    printf("hour=%d:%d\n", ltmp->tm_hour, ltmp->tm_min);

    // 合并分解时间
    // 增加10秒
    ltmp->tm_sec += 10;
    time_t mkt = mktime(ltmp);
    printf("mkt=%ld\n", mkt);

    // 把分解时间转成字符串
    char* tstr = asctime(ltmp);
    printf("tstr=%s\n", tstr);

    // 格式化时间
    char tbuf[TIME_LEN];
    strftime(tbuf, TIME_LEN, "%Z %Y %X", ltmp);
    printf("%s\n", tbuf);

    return 0;
}
```

### 21. 软件时钟(jiffies)
```c
```

### 22. 进程时间, times/clock
```c
// 12-1
#include <stdio.h>
#include <pwd.h>
#include <dirent.h>
#include <ctype.h>
#include "tlpi_hdr.h"
#include "ps.h"

int main(int argc, char* argv[]) {
    if (argc < 2) {
        errExit("Not found username");
    }

    char* username = argv[1];
    int uid = get_uid_by_name(username);
    if (uid == -1) {
        errExit("get uid failed");
    }

    DIR *rootdir = opendir(ROOT_DIR);
    if (rootdir == NULL) {
        errExit("opendir");
    }

    struct dirent *dir;
    while ((dir = readdir(rootdir)) != NULL) {
        if (dir->d_type != DT_DIR) continue;
        if (!digit(dir->d_name)) continue;

        char* pid = dir->d_name;
        char* status_path = gen_status_path(pid);
        // printf("path=%s\n", status_path);
        struct Entry *entry = readentry(uid, status_path);
        if (entry != NULL) {
            printf("    %s    %ld\n", entry->name, entry->pid);
        }
    }
    // printf("finish\n");

    return 0;
}
```

```c
// 12-2 生成进程树
#include <stdio.h>
#include "ps.h"

int main(int argc, char* argv[]) {
    struct node *head = NULL;
    struct node *tail = NULL;

    DIR *rootdir = opendir(ROOT_DIR);
    if (rootdir == NULL) {
        errExit("opendir");
    }

    struct dirent *dir;
    while ((dir = readdir(rootdir)) != NULL) {
        if (dir->d_type != DT_DIR) continue;
        if (!digit(dir->d_name)) continue;

        char* pid = dir->d_name;
        char* status_path = gen_status_path(pid);
        // printf("path=%s\n", status_path);
        // 1.create node
        struct node *np = createnode(status_path);

        // 2. sort
        if (head == NULL) {
            head = np;
            tail = np;
        } else {
            struct node *pre = NULL;
            struct node *p = head;
            while (p != NULL && p->entry->pid < np->entry->pid) {
                pre = p;
                p = p->next;
            }
            np->next = p;
            pre->next = np;
            if (tail->next != NULL) {
                tail = tail->next;
            }
        }
    }

    print_queue(head);

    // 3.gen tree
    struct node *root = createtree(head);

    // 4.print tree
    print_tree(head, root);
    return 0;
}
```

```c
#include <stdio.h>
#include <pwd.h>
#include <dirent.h>
#include <ctype.h>
#include "tlpi_hdr.h"

#define ROOT_DIR "/proc"
#define STATUS "status"
#define READ_BUFFER_SIZE 1024

int get_uid_by_name(char*);
int digit(char* s);
char* gen_status_path(char* pid);
char* getval(char* line, char* name);

struct Entry {
    char name[100];
    long pid;
    long ppid;
};

struct node {
    struct Entry *entry;
    struct node *child; // for tree
    struct node *next; // for queue
};

void print_queue(struct node *head);
struct node *createnode(char* path);
struct node *createtree(struct node *head);
struct node *clonenode(struct node *p);
struct node *searchnode(struct node *root, long pid);
void print_tree(struct node *head, struct node *root);

struct Entry *readentry(int uid, char* path);
```

```c
#include "ps.h"

int get_uid_by_name(char* name) {
    struct passwd *pwd = getpwnam(name);
    if (pwd == NULL) {
        return -1;
    }
    return pwd->pw_uid;
}

int digit(char* s) {
    for (char* p=s; (*p)!='\0'; p++) {
        if ((*p) < '0' || (*p) > '9') return 0;
    }
    return 1;
}

char* gen_status_path(char* pid) {
    int len = strlen(ROOT_DIR) + 1 + strlen(pid) + 1 + strlen(STATUS);
    char *buf = malloc(len);
    memset(buf, 0, len);
    strcat(buf, ROOT_DIR);
    strcat(buf, "/");
    strcat(buf, pid);
    strcat(buf, "/");
    strcat(buf, STATUS);
    return buf;
}

struct Entry *readentry(int uid, char* path) {
    struct Entry *pentry = malloc(sizeof(struct Entry));

    FILE* f = fopen(path, "r");

    char* linebuf = malloc(READ_BUFFER_SIZE);
    memset(linebuf, 0, READ_BUFFER_SIZE);

    // Name:	sshd
    // Pid:	1
    // Uid:	1000	1000	1000	1000

    // uid is match
    while ((linebuf = fgets(linebuf, READ_BUFFER_SIZE, f)) != NULL) {
        char* uidstr = getval(linebuf, "Uid:");
        if (uidstr == NULL) continue;
        int target = atoi(uidstr);
        if (target == uid) {
            break;
        } else {
            return NULL;
        }
    }

    if (fseek(f, 0, SEEK_SET) == -1) {
        errMsg("fseek");
        return NULL;
    }

    while ((linebuf = fgets(linebuf, READ_BUFFER_SIZE, f)) != NULL) {
        char* name = getval(linebuf, "Name:");
        if (name != NULL) {
            strcpy(pentry->name, name);
        }

        char* pid = getval(linebuf, "Pid:");
        if (pid != NULL) {
            pentry->pid = atol(pid);
        }
    }
    return pentry;
}

char* getval(char* line, char* name) {
    char* p1 = line;
    char* p2 = name;
    while ((*p1) == (*p2)) {
        p1++;
        p2++;
    }
    if (p2 - name != strlen(name)) {
        return NULL;
    }

    int size = strlen(line);
    char* buf = malloc(size);
    memset(buf, 0, size);
    int i = 0;
    for (char* p=p1+1; (*p)!='\0'; p++) {
        if (isspace(*p)) {
            if (i > 0) {
                break;
            } else {
                continue;
            }
        }
        buf[i] = *p;
        i++;
    }
    return buf;
}

struct node *createnode(char* path) {
    struct Entry *pentry = malloc(sizeof(struct Entry));

    FILE *f = fopen(path, "r");

    char *linebuf = malloc(READ_BUFFER_SIZE);
    memset(linebuf, 0, READ_BUFFER_SIZE);

    while ((linebuf = fgets(linebuf, READ_BUFFER_SIZE, f)) != NULL) {
        char* name = getval(linebuf, "Name:");
        if (name != NULL) {
            strcpy(pentry->name, name);
        }

        char* pid = getval(linebuf, "Pid:");
        if (pid != NULL) {
            pentry->pid = atol(pid);
        }

        char* ppid = getval(linebuf, "PPid:");
        if (ppid != NULL) {
            pentry->ppid = atol(ppid);
        }
    }

    struct node *p = malloc(sizeof(struct node));
    p->entry = pentry;
    return p;
}

void print_queue(struct node *head) {
    if (head == NULL) return;
    struct node *p = head;
    while (p->next != NULL) {
        printf("%s(%ld)-->", p->entry->name,  p->entry->pid);
        p = p->next;
    }
    printf("\n");
}

struct node *createtree(struct node *head) {
    struct node *root = NULL;
    struct node *p = head;
    while (p != NULL) {
        // 从队列里找child节点, 如果找到就clone，然后加入child队列
        struct node *q = head;
        while (q != NULL) {
            if (p->entry->pid == q->entry->ppid) {
                struct node *x = clonenode(q);
                if (root == NULL) {
                    root = x;
                }
                x->next = p->child;
                p->child = x;
            }
            q = q->next;
        }
        p = p->next;
    }
    return root;
}

struct node *clonenode(struct node *p) {
    struct node *r = malloc(sizeof(struct node));
    r->next = NULL;
    r->child = NULL;
    r->entry = malloc(sizeof(struct Entry));
    strcpy(r->entry->name, p->entry->name);
    r->entry->pid = p->entry->pid;
    r->entry->ppid = p->entry->ppid;
    return r;
}

struct node *searchnode(struct node *root, long pid) {
    if (root == NULL) return NULL;
    if (root->entry->pid == pid) return root;

    struct node *p = root->child;
    while (p != NULL) {
        struct node *x = searchnode(p, pid);
        if (x != NULL) return x;
        p = p->next;
    }
    return NULL;
}

void print_tree(struct node *head, struct node *root) {
    if (root == NULL) return;

    struct node *p = head;
    while (p != NULL) {
        printf("%s(%ld)-->>", p->entry->name,  p->entry->pid);
        struct node *x = p->child;
        while (x != NULL) {
            printf("%s(%ld)-->", x->entry->name,  x->entry->pid);
            x = x->next;
        }
        printf("\n");
        p = p->next;
    }
}
```

```c
// 12-3 打开某个文件fd的进程id列表
#include <stdio.h>
#include <stdlib.h>
#include <limits.h>
#include "ps.h"

struct pidnode {
    long pid;
    struct pidnode *next;
};

struct item {
    char* path;
    struct pidnode *node;
};

char* gen_fd_dir_path(char* pid);
struct item *createitem(char* path, long pid);
void print_list(struct item *list[]);

int main(int argc, char* argv[]) {
    struct item *list[2000];
    int i = 0;

    DIR *rootdir = opendir(ROOT_DIR);
    struct dirent *dir;
    while ((dir = readdir(rootdir)) != NULL) {
        if (dir->d_type != DT_DIR) continue;
        if (!digit(dir->d_name)) continue;

        char* pid = dir->d_name;
        char* fdpath = gen_fd_dir_path(pid);

        DIR *fddir = opendir(fdpath);
        if (fddir == NULL) {
            errExit("opendir");
        }

        struct dirent *fdd;
        while ((fdd = readdir(fddir)) != NULL) {
            if (fdd->d_type != DT_LNK) continue;
            if (!digit(fdd->d_name)) continue;

            char* filepath = malloc(PATH_MAX);
            strcpy(filepath, fdpath);
            strcat(filepath, fdd->d_name);

            char* buf = malloc(PATH_MAX);
            if (readlink(filepath, buf, PATH_MAX) == -1) {
                errMsg("readlink");
            }

            // printf("%d    %s\n", i,  buf);
            list[i++] = createitem(buf, atol(pid));
        }

    }

    print_list(list);
    return 0;
}
void print_list(struct item *list[]) {
    for (struct item **p=list; (*p) != NULL; p++) {
    //for (int i=0; list[i]!=NULL; i++) {
        //struct item *p = list[i];
        printf("%s    ", (*p)->path);
        struct pidnode *x = (*p)->node;
        while (x != NULL) {
            printf("%ld-->", x->pid);
            x = x->next;
        }
        printf("\n");
    }
}

struct item *createitem(char* path, long pid) {
    int size = sizeof(struct item);
    struct item *one = malloc(size);
    memset(one, 0, size);
    one->path = path;
    one->node = malloc(sizeof(struct pidnode));
    one->node->pid = pid;
    return one;
}

char* gen_fd_dir_path(char* pid) {
    char* path = malloc(PATH_MAX);
    memset(path, 0, PATH_MAX);
    strcat(path, "/proc/");
    strcat(path, pid);
    strcat(path, "/fd/");
    return path;
}
```

### 23. I/O的两层缓冲（内核缓冲和libc的缓冲）
```c
// 控制用户空间缓冲
// setbuf/setvbuf/setbuffer/fflush

// 控制内核空间的缓冲
// fsync/fdatasync/O_SYNC 

// 绕过用户空间和内核空间的缓冲
// O_DIRECT

// fileno
// fdopen
```

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int
main(int argc, char *argv[])
{
    // setbuf(stdout, NULL);
    printf("If I had more time, \n");
    write(STDOUT_FILENO, "I would have written you a shorter letter.\n", 43);
    exit(EXIT_SUCCESS);
}
```

### 24. 文件系统
```c
// 文件系统要解决的核心问题是, 目录和文件组成的树结构是怎样在一条线性结构上存储的？
```

```shell
# 如何创建一个文件系统, 使用文件模拟的方式
# 1. 使用dd拷贝zero中的字节到myfs，每次拷贝256字节，拷贝4K次
dd if=/dev/zero of=myfs count=256 bs=4K
256+0 records in
256+0 records out
1048576 bytes (1.0 MB, 1.0 MiB) copied, 0.0012152 s, 863 MB/s

# 2. 在myfs上格式化, 每个block 1024字节
mke2fs -b 1024 myfs
mke2fs 1.45.5 (07-Jan-2020)
Discarding device blocks: done
Creating filesystem with 256 4k blocks and 128 inodes

Allocating group tables: done
Writing inode tables: done
Writing superblocks and filesystem accounting information: done

# 3. dump文件系统
dumpe2fs myfs
dumpe2fs 1.45.5 (07-Jan-2020)
Filesystem volume name:   <none>
Last mounted on:          <not available>
Filesystem UUID:          4eaa38d0-0996-465f-876f-62651858305f
Filesystem magic number:  0xEF53
Filesystem revision #:    1 (dynamic)
Filesystem features:      ext_attr resize_inode dir_index filetype sparse_super large_file
Filesystem flags:         signed_directory_hash
Default mount options:    user_xattr acl
Filesystem state:         clean
Errors behavior:          Continue
Filesystem OS type:       Linux
Inode count:              128
Block count:              1024
Reserved block count:     51
Free blocks:              986
Free inodes:              117
First block:              1
Block size:               1024
Fragment size:            1024
Reserved GDT blocks:      3
Blocks per group:         8192
Fragments per group:      8192
Inodes per group:         128
Inode blocks per group:   16
Filesystem created:       Wed May 12 17:12:29 2021
Last mount time:          n/a
Last write time:          Wed May 12 17:12:29 2021
Mount count:              0
Maximum mount count:      -1
Last checked:             Wed May 12 17:12:29 2021
Check interval:           0 (<none>)
Reserved blocks uid:      0 (user root)
Reserved blocks gid:      0 (group root)
First inode:              11
Inode size:	          128
Default directory hash:   half_md4
Directory Hash Seed:      422d2f7b-6f06-41cb-866e-469150bf95be


Group 0: (Blocks 1-1023)
  Primary superblock at 1, Group descriptors at 2-2
  Reserved GDT blocks at 3-5
  Block bitmap at 6 (+5)
  Inode bitmap at 7 (+6)
  Inode table at 8-23 (+7)
  986 free blocks, 117 free inodes, 2 directories
  Free blocks: 38-1023
  Free inodes: 12-128

# 4. mount文件系统
sudo mount -o loop /mnt

# 5. 在文件系统上创建文件
touch ming
vim ming
文件内容：123456

# 6. umount /mnt
```

```shell
# 7. od -tx1 -Ax myfs 
000000~000400, 启动块(block 0), 
000000 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
* 
(上边是64行00, 1024字节, 刚好1个block)

000400~0007ff, superblock(block 1)
000400 80 00 00 00 00 04 00 00 33 00 00 00 d9 03 00 00
000410 74 00 00 00 01 00 00 00 00 00 00 00 00 00 00 00
000420 00 20 00 00 00 20 00 00 80 00 00 00 27 9c 9b 60
000430 5e 9c 9b 60 01 00 ff ff 53 ef 01 00 01 00 00 00
000440 fd 9b 9b 60 00 00 00 00 00 00 00 00 01 00 00 00
000450 00 00 00 00 0b 00 00 00 80 00 00 00 38 00 00 00
000460 02 00 00 00 03 00 00 00 4e aa 38 d0 09 96 46 5f
000470 87 6f 62 65 18 58 30 5f 00 00 00 00 00 00 00 00
000480 00 00 00 00 00 00 00 00 2f 6d 6e 74 00 00 00 00
000490 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
*
0004c0 00 00 00 00 00 00 00 00 00 00 00 00 00 00 03 00
0004d0 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
0004e0 00 00 00 00 00 00 00 00 00 00 00 00 42 2d 2f 7b
0004f0 6f 06 41 cb 86 6e 46 91 50 bf 95 be 01 00 00 00
000500 0c 00 00 00 00 00 00 00 fd 9b 9b 60 00 00 00 00
000510 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
*
000560 01 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
000570 00 00 00 00 00 00 00 00 09 00 00 00 00 00 00 00
000580 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
*

000800~000bff, Group descriptors (block 2-5)
000800 06 00 00 00 07 00 00 00 08 00 00 00 d9 03 74 00
000810 02 00 04 00 00 00 00 00 00 00 00 00 00 00 00 00
000820 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
*

001800~001c00, block bitmap (block 6)
001800 ff ff ff ff 3f 00 00 00 00 00 00 00 00 00 00 00
001810 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
*
001870 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 80
001880 ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff
*

001c00~002000, inode bitmap (block 7)
001c00 ff 17 00 00 00 00 00 00 00 00 00 00 00 00 00 00
001c10 ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff
*




inode-------------begin
inode 1
002000 00 00 00 00 00 00 00 00 fd 9b 9b 60 fd 9b 9b 60
002010 fd 9b 9b 60 00 00 00 00 00 00 00 00 00 00 00 00
002020 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
*

inode 2
002080 ed 41 00 00 00 04 00 00 62 89 9c 60 5e 89 9c 60
002090 5e 89 9c 60 00 00 00 00 00 00 04 00 02 00 00 00
0020a0 00 00 00 00 0a 00 00 00 18 00 00 00 00 00 00 00
0020b0 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
*
002300 80 81 00 00 00 30 04 04 fd 9b 9b 60 fd 9b 9b 60
002310 fd 9b 9b 60 00 00 00 00 00 00 01 00 08 00 00 00
002320 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
*
002350 00 00 00 00 00 00 00 00 00 00 00 00 25 00 00 00
002360 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
*
002500 c0 41 00 00 00 30 00 00 fd 9b 9b 60 fd 9b 9b 60
002510 fd 9b 9b 60 00 00 00 00 00 00 02 00 18 00 00 00
002520 00 00 00 00 00 00 00 00 19 00 00 00 1a 00 00 00
002530 1b 00 00 00 1c 00 00 00 1d 00 00 00 1e 00 00 00
002540 1f 00 00 00 20 00 00 00 21 00 00 00 22 00 00 00
002550 23 00 00 00 24 00 00 00 00 00 00 00 00 00 00 00
002560 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
*

inode 12 0xc mings
002580 ff a1 00 00 04 00 00 00 4b 89 9c 60 49 89 9c 60
002590 49 89 9c 60 00 00 00 00 00 00 01 00 00 00 00 00
0025a0 00 00 00 00 01 00 00 00 6d 69 6e 67 00 00 00 00
0025b0 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
*
0025e0 00 00 00 00 6b 2d f6 bf 00 00 00 00 00 00 00 00
0025f0 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00

inode 13 0xd ming1
002600 a4 81 00 00 07 00 00 00 41 9c 9b 60 2c 89 9c 60
002610 41 9c 9b 60 00 00 00 00 00 00 02 00 02 00 00 00
002620 00 00 00 00 01 00 00 00 26 00 00 00 00 00 00 00
002630 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
*
002660 00 00 00 00 38 35 13 02 00 00 00 00 00 00 00 00
002670 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00

inode 14 0xe kk
002680 ed 41 00 00 00 04 00 00 5e 89 9c 60 5e 89 9c 60
002690 5e 89 9c 60 00 00 00 00 00 00 02 00 02 00 00 00
0026a0 00 00 00 00 01 00 00 00 27 00 00 00 00 00 00 00
0026b0 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
*
0026e0 00 00 00 00 d3 24 b1 7e 00 00 00 00 00 00 00 00
0026f0 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
*
inode------------end

block 1
006000 02 00 00 00 0c 00 01 02 2e 00 00 00 02 00 00 00
006010 0c 00 02 02 2e 2e 00 00 0b 00 00 00 14 00 0a 02
006020 6c 6f 73 74 2b 66 6f 75 6e 64 00 00 0d 00 00 00
006030 0c 00 04 01 6d 69 6e 67 0d 00 00 00 10 00 05 01
006040 6d 69 6e 67 31 00 00 00 0c 00 00 00 10 00 05 07
006050 6d 69 6e 67 73 00 00 00 0e 00 00 00 a8 03 02 02
006060 6b 6b 00 00 00 00 00 00 00 00 00 00 00 00 00 00
006070 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
*
006400 0b 00 00 00 0c 00 01 02 2e 00 00 00 02 00 00 00
006410 f4 03 02 02 2e 2e 00 00 00 00 00 00 00 00 00 00
006420 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
*
006800 00 00 00 00 00 04 00 00 00 00 00 00 00 00 00 00
006810 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
*
006c00 00 00 00 00 00 04 00 00 00 00 00 00 00 00 00 00
006c10 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
*
007000 00 00 00 00 00 04 00 00 00 00 00 00 00 00 00 00
007010 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
*
007400 00 00 00 00 00 04 00 00 00 00 00 00 00 00 00 00
007410 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
*
007800 00 00 00 00 00 04 00 00 00 00 00 00 00 00 00 00
007810 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
*
007c00 00 00 00 00 00 04 00 00 00 00 00 00 00 00 00 00
007c10 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
*
008000 00 00 00 00 00 04 00 00 00 00 00 00 00 00 00 00
008010 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
*
008400 00 00 00 00 00 04 00 00 00 00 00 00 00 00 00 00
008410 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
*
008800 00 00 00 00 00 04 00 00 00 00 00 00 00 00 00 00
008810 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
*
008c00 00 00 00 00 00 04 00 00 00 00 00 00 00 00 00 00
008c10 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
*
009000 00 00 00 00 00 04 00 00 00 00 00 00 00 00 00 00
009010 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
*
009400 00 00 00 00 03 00 00 00 04 00 00 00 05 00 00 00
009410 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
*

block 38 ming的数据
009800 31 32 33 34 35 36 0a 00 00 00 00 00 00 00 00 00
009810 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
*

block 39 kk的数据
009c00 0e 00 00 00 0c 00 01 02 2e 00 00 00 02 00 00 00
009c10 f4 03 02 02 2e 2e 00 00 00 00 00 00 00 00 00 00
009c20 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
*
100000
```

### 25. 获取/修改inode节点的信息
```c
// 1. stat/fstat/lstat

// 2. utime/utimes

// 3. chmod/fchmod/lchmod
```