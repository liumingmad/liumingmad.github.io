title: cpu cache是怎样影响程序执行的
author: ming
date: 2026-02-04 20:39:57
tags:
---
## cpu cache是怎样影响程序执行的

### 0. 前置知识
* 通常存储器分为多级，依次L1, L2, L3, 主内存，硬盘
* L1和L2在CPU内，属于某个核独享
* 我的macos m2
```
🔹 性能核（P-Core ×8）
	•	L1 I：192 KB / core
	•	L1 D：128 KB / core
	•	L2：16 MB（8 核共享）

🔹 能效核（E-Core ×4）
	•	L1 I：128 KB / core
	•	L1 D：64 KB / core
	•	L2：4 MB（4 核共享）

🔹 System Level Cache（SLC）
	•	48 MB
	•	SoC 全共享
	•	macOS 不暴露

🔹 Cache Line
	•	128 bytes
```

* 我的google pixel 2 
```
✅ CPU 架构
	•	Snapdragon 835（MSM8998）
	•	8 核 big.LITTLE
	•	4× Cortex-A73（性能核）
	•	4× Cortex-A53（能效核）

⸻

✅ Cache 架构

大核（A73 ×4）
	•	L1 I：64 KB / core
	•	L1 D：64 KB / core
	•	L2：2 MB（4 核共享）

小核（A53 ×4）
	•	L1 I：32 KB / core
	•	L1 D：32 KB / core
	•	L2：1 MB（4 核共享）

SoC 级缓存
	•	Qualcomm System Cache（存在）
	•	Android 不暴露
	•	不等同于 x86 的 L3
```

```
// cache line大小, 字节
cat /sys/devices/system/cpu/cpu0/cache/index0/coherency_line_size
64
```

### 1. 缓存如何被加载
```
硬件完成的，无需指令参与加载，但是需要知道大概的存取速度:
L1 的存取速度：4 个CPU时钟周期
L2 的存取速度： 11 个CPU时钟周期
L3 的存取速度：39 个CPU时钟周期
RAM内存的存取速度：107 个CPU时钟周期
```

### 2. CPU怎样查找缓存
```
内存地址被分为三段：
|  Tag(24bit)  | Index(6bit) | Offset(6bit) |

硬件步骤：
	1.	用 Set Index 选中一个 set（64 个之一）
	2.	在这个 set 的 8 个 way 里并行比较 Tag
	3.	找到 Tag 匹配的那一条 line
	4.	用 Offset 在这 64B 里取具体数据
```
* 对于32KB的L1，64byte的cache line，相当于512条cache line
* Index用于定位到哪个set
* Tag表示哪个line
* Offset是数据在cache line中的偏移
* 因此，很容易想到一种性能极差的场景，就是cache冲突，也就是Index一致，但是Tag不同，就会Cache miss，导致重新从下级缓存加载

### 3. 对于多核cpu，缓存如何保持一致
```
现在基本都是使用Snoopy的总线的设计, 也就是当同一个cache line被读到多个L1中时，如果有一个核写了某个字节，其他核的cache line会被标记为失效
```

### 4. 程序应该怎样做，一些例子(抄了左耳朵耗子的例子，这些例子已经足够好了)
##### 1. 使用不同步长遍历数组，主要关注的点有两个：
   * 两种遍历方式，耗时差不多，之所以这样，是因为cache miss的次数差不多
   * 乘法基本不耗时 
   ```
   const int LEN = 64*1024*1024;
   int *arr = new int[LEN];

   for (int i = 0; i < LEN; i += 2) arr[i] *= i;

   for (int i = 0; i < LEN; i += 8) arr[i] *= i;
   ```

##### 2. cache冲突，当步长是1024时，因为是int类型，所以每次跳过2的12次幂，低12位基本不变，大量的数据都命中同一个set，而一个set中只有8个cache line，对于32K的L1来说，只有这一个set被使用，其他set完全没有利用到
```
for (int i = 0; i < 10000000; i++) {
    for (int j = 0; j < size; j += increment) {
        memory[j] += j;
    }
}
| count |   1    |   16  |  512  | 1024  |
------------------------------------------
|     1 |  17539 | 16726 | 15143 | 14477 |
|     2 |  15420 | 14648 | 13552 | 13343 |
|     3 |  14716 | 14463 | 15086 | 17509 |
|     4 |  18976 | 18829 | 18961 | 21645 |
|     5 |  23693 | 23436 | 74349 | 29796 |
|     6 |  23264 | 23707 | 27005 | 44103 |
|     7 |  28574 | 28979 | 33169 | 58759 |
|     8 |  33155 | 34405 | 39339 | 65182 |
|     9 |  37088 | 37788 | 49863 |156745 |
|    10 |  41543 | 42103 | 58533 |215278 |
|    11 |  47638 | 50329 | 66620 |335603 |
|    12 |  49759 | 51228 | 75087 |305075 |
|    13 |  53938 | 53924 | 77790 |366879 |
|    14 |  58422 | 59565 | 90501 |466368 |
|    15 |  62161 | 64129 | 90814 |525780 |
|    16 |  67061 | 66663 | 98734 |440558 |
|    17 |  71132 | 69753 |171203 |506631 |
|    18 |  74102 | 73130 |293947 |550920 |

```
我们可以看到，从 [9，1024] 以后，时间显著上升。包括 [17，512] 和 [18,512] 也显著上升。这是因为，我机器的 L1 Cache 是 32KB, 8 Way 的，前面说过，8 Way的有64组，每组8个Cache Line，当for-loop步长超过1024个整型，也就是正好 4096 Bytes时，也就是导致内存地址的变化是变化在高位的24bits上，而低位的12bits变化不大，尤其是中间6bits没有变化，导致全部命中同一组set，导致大量的cache 冲突，导致性能下降，时间上升。而 [16, 512]也是一样的，其中的几步开始导致L1 Cache开始冲突失效。

##### 3. 这个例子中，分为按行和按列遍历
* 按列遍历的问题在于, 因为每行1024个int，所以按列遍历，表示每次都跳过4096字节，因此还是每次都命中同一个set，而一个set中只有8个车位，到第9个之后的每一次，一定cache miss
* 按行遍历时，是均匀遍历，如果出现一次cache miss，一定会有15次cache hit，而且也不会对着一个set猛干，因为内存地址的6～11位是会均匀变的
```
const int row = 1024;
const int col = 512
int matrix[row][col];

//逐行遍历
int sum_row=0;
for(int _r=0; _r<row; _r++) {
    for(int _c=0; _c<col; _c++){
        sum_row += matrix[_r][_c];
    }
}

//逐列遍历
int sum_col=0;
for(int _c=0; _c<col; _c++) {
    for(int _r=0; _r<row; _r++){
        sum_col += matrix[_r][_c];
    }
}
逐行遍历：0.081ms
逐列遍历：1.069ms
```

##### 4. 这个例子是，当同一个cache line被多个核加载到各自的L1时，不断互相被置为无效，导致每次无效时，都要重新加载一次cache line
```
void fn (int* data) {
    for(int i = 0; i < 10*1024*1024; ++i)
        *data += rand();
}

int p[32];

int *p1 = &p[0];
int *p2 = &p[1]; // int *p2 = &p[30];

thread t1(fn, p1);
thread t2(fn, p2);
```

##### 5. 这个例子中，多线程分片读，但是写入结果的数组，在同一个cache line，还是会导致无效
```
int total_size = 16 * 1024 * 1024; //数组长度
int* test_data = new test_data[total_size]; //数组
int nthread = 6; //线程数（因为我的机器是6核的）
int result[nthread]; //收集结果的数组

void thread_func (int id) {
    result[id] = 0;
    int chunk_size = total_size / nthread + 1;
    int start = id * chunk_size;
    int end = min(start + chunk_size, total_size);

    for ( int i = start; i < end; ++i ) {
        if (test_data[i] % 2 != 0 ) ++result[id];
    }
}
```

### 5. 综述
导致性能下降的原因主要有两个：
   * L1的利用率下降
   * 多核为了cache的一致性，做同步操作，导致cache line无效