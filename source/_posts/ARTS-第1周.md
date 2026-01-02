title: ARTS_第1周
author: ming
date: 2020-07-06 21:55:03
tags:
---

## Algorithm
### [1.leecode300最长递增子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence/submissions/)

* 标准的动态规划问题，也就是使用dp数组，保存了从[0,i]的最长递增子序列，避免了子问题的重复计算

```java
    public int lengthOfLIS(int[] nums) {
        int size = nums.length;
        if (size == 0) return 0;

        int[] dp = new int[size];
        for (int i=0; i<size; i++) {
            dp[i] = 1;
        }

        for (int i=1; i<size; i++) {
            for (int j=0; j<i; j++) {
                if (nums[j] < nums[i]) {
                    dp[i] = Math.max(dp[i], dp[j]+1);
                }
            }
        }

        int max = 0;
        for (int i=0; i<size; i++) {
            max = Math.max(dp[i], max);
        }
        return max;
    }
```

* 对于这道题，如果想使用递归+备忘录的方式做，也可以，只不过有点别扭，因为无论是递归还是递推，都要把每个位置的最长上升子序列的长度求出来
* 另外，这道题的递归，其实比较诡异，因为一般情况，递归是使用前一种状态，能够推导出下一种状态，但是这道题，是使用子问题中的多种状态，才能推导出当前状态。
```java
    public int lengthOfLIS(int[] nums) {
        int size = nums.length;
        if (size == 0) return 0;

        int[] dp = new int[size];
        for (int i=0; i<size; i++) {
            dp[i] = 1;
        }

        for (int i=0; i<size; i++) {
            help(nums, i);
        }

        int max = 0;
        for (int i=0; i<size; i++) {
            max = Math.max(dp[i], max);
        }
        return max;
    }

    private int help(int[] nums, int index) {
        if (index == 0) return 1;
        if (mem[index] >= 1) return mem[index];
        int max = 0;
        for (int i=0; i<index; i++) {
            if (nums[i] < nums[index]) {
                max = Math.max(help(nums, i), max);
            }
        }
        mem[index] = max + 1;
        return mem[index]
    }
```

### [2.最大连续子数组的和](https://leetcode-cn.com/problems/maximum-subarray/)

```java
    public int maxSubArray(int[] nums) {
        if (nums.length == 0) return 0;

        int[] dp = new int[nums.length];
        dp[0] = nums[0];
        for (int i=1; i<nums.length; i++) {
            if (dp[i-1] > 0) {
                dp[i] = dp[i-1] + nums[i];
            } else {
                dp[i] = Math.max(dp[i-1], nums[i]);
            }
        }

        int max = dp[0];
        for (int i=1; i<nums.length; i++) {
            if (dp[i] > max) {
                max = dp[i];
            }
        }
        return max;
    }
```

* 与上一题类似，差别就是递推公式，只用到了前一个dp状态

### [3. 01背包](https://labuladong.gitbook.io/algo/dong-tai-gui-hua-xi-lie/bei-bao-wen-ti)
```java
    private static int knapsack01(int n, int maxW, int[] w, int[] p) {
        int[][] dp = new int[n][maxW+1];
        for (int i=0; i<n; i++) {
            for (int j=0; j<maxW+1; j++) {
                dp[i][j] = -1;
            }
        }
        
        // 初始化第一行
        dp[0][0] = 0;
        if (w[0] <= maxW) dp[0][w[0]] = p[0]; 

        for (int i=1; i<n; i++) {
            // 不选i
            for (int j=0; j<=maxW; j++) {
                if (dp[i-1][j] < 0) continue;
                dp[i][j] = dp[i-1][j];
            }

            // 选i
            for (int j=0; j<=maxW; j++) {
                if (dp[i-1][j] < 0) continue;
                int x = j + w[i]; 
                if (x > maxW) break; 
                dp[i][x] = Math.max(dp[i-1][x], dp[i-1][j]+p[i]);
            }
        }

        int maxP = 0;
        for (int i=0; i<maxW+1; i++) {
            if (dp[n-1][i] > maxP) {
                maxP = dp[n-1][i];
            }
        }
        return maxP;
    }
```

### [4.杨辉三角]()
```java
    private static class Node {
        int val;
        Node left;
        Node right;

        public Node(int val) {
            this.val = val;
        }
    }

    // 极客时间，从顶到底的最短路径
    private int minLen(Node root, Map<Node, Integer> mem) {
        if (root == null) return 0;
        if (mem.containsKey(root)) return mem.get(root);
        int res = root.val + Math.min(minLen(root.left, mem), minLen(root.right, mem));
        mem.put(root, res);
        return res;
    }
```

[参考1](https://blog.csdn.net/zw6161080123/article/details/80639932)
[dp问题分类](https://blog.csdn.net/eagle_or_snail/article/details/50987044)

## Review
## [Difference between Java IO and Java NIO](https://www.geeksforgeeks.org/difference-between-java-io-and-java-nio/)

这篇文章介绍java IO和NIO的区别，主要包含如下差别
1. 分别在java.io包和java.nio包
2. io面向流, nio面向缓冲区
3. io是阻塞的，nio非阻塞
4. Channels, io不可用，nio可用
5. io处理字节，nio处理数据块
6. Selectors, io没有多路复用的概念，nio可以多路复用


## Tip
* 开始读apue，先下了apue的source code，编译，然后拷贝apue.h和error.c到/usr/include/。

* 跑了三个demo：
1. 简单实现ls命令
    ```c
    #include <apue.h>
    #include <error.c>
    #include <dirent.h> 

    int main(int argc, char* argv[]) {
        DIR* dp;
        struct dirent* dirp;

        if (argc != 2) {
            err_quit("usage: ls dir_name");
        }

        if ((dp = opendir(argv[1])) == NULL) {
            err_sys("can't open %s", argv[1]);
        }

        while ((dirp = readdir(dp)) != NULL) {
            printf("%s\n", dirp->d_name);
        }

        closedir(dp);
        exit(0);
    }
    ```

2. 用POSIX接口，read/write标准输入输出, ./in_out < infile > outfile
    ```c
    #include <apue.h>
    #include <error.c>

    #define BUF_SIZE 100

    int main(void) {
        int n;
        char buf[BUF_SIZE];

        while ((n = read(STDIN_FILENO, buf, BUF_SIZE)) > 0) {
            int count = write(STDOUT_FILENO, buf, n);
            if (count != n) {
                err_sys("write error!");
            }
        }

        if (n < 0) {
            err_sys("read error");
        } 
        exit(0);
    }
    ```

3. 使用标准库函数getc/putc（stdio.h), 做标准输入输出, 与2的区别就是，库函数中自带了缓冲，最终也是调用read／write  
    ```c
    #include <apue.h> 
    #include <error.c>

    int main() {
        int c;
        while ((c = getc(stdin)) != EOF) {
            if (EOF == putc(c, stdout)) {
                err_sys("putc EOF");
            }
        }

        if (ferror(stdin)) {
            err_sys("stdin EOF");
        }
        exit(0);
    }
    ```
4. 进程相关，fork，execlp，getpid/getppid。
    ```c
    #include <apue.h>
    #include <error.c>

    #define BUFFER_SIZE 100

    int main() {
        long pid;
        char buf[BUFFER_SIZE];
        int status;

        printf("%% ");
        while ((fgets(buf, BUFFER_SIZE, stdin)) != NULL) {
            int len = strlen(buf);
            if (buf[len-1] == '\n') {
                buf[len-1] = 0;
            }

            pid = fork();
            if (pid == 0) {
                execlp(buf, buf, (char*)0);
                err_ret("could'nt execute: %s", buf);
                exit(127);
            } else if (pid < 0) {
                err_sys("fork error!");
            }

            if ((pid = waitpid(pid, &status, 0)) < 0) {
                err_sys("waitpid error");
            }
            printf("%% ");
        }

        exit(0);
    }
    ```

5. 出错处理，strerr, perror
    ```c
    #include <stdio.h>
    #include <string.h>
    #include <error.c>
    #include <errno.h>

    int main(int argc, char* argv[]) {
        fprintf(stderr, "EACCES: %s\n", strerror(EACCES));
        errno = ENOENT;
        perror(argv[0]);
        exit(0);
    }
    ```

6. 用户id和组id
    ```c
    #include <apue.h>

    int main() {
        printf("uid is %d, gid is %d\n", getuid(), getgid());
        exit(0);
    }
    ```

7. 信号处理
    ```c
    if (signal(SIGINT, sig_int) == SIG_ERR) {
        err_sys("signal error");
    }

    void sig_int(int sig) {
        printf("\nsig_int-->%d\n", sig);
    }
    ```
## Share
#### 并发是他妈什么鬼
* 站在单核的cpu的汇编指令层，是不存在并发的，指令永远是一条一条执行的，不可能同时执行两条指令。
* 站在操作系统的层面，因为cpu的速度太快，不想等其他速度慢的设备（比如IO），所以操作系统制造了进程和线程的概念。
* 站在应用层，操作系统提供了进程，为了方便的共享内存，在进程内，OS提供了线程。
* 因为同一个进程内，多线程共享虚拟内存，由于并发的不确定性，所以太容易导致多个线程读写了同一块内存，出现意料之外的错误。因此，出现了锁的概念。
* 锁按照从严格到宽松的顺序，互斥锁，信号量，临界区，读写锁，CAS。

#### NIO是他妈什么鬼
[参考1](https://www.cnblogs.com/binarylei/p/8933516.html)
[参考2](https://github.com/justtreee/blog/issues/17)
[参考3](https://tech.meituan.com/2016/11/04/nio.html)