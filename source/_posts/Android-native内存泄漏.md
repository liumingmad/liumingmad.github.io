title: <Android>native内存泄漏
author: ming
date: 2025-05-02 21:25:06
tags:
---
### 内存泄漏检测工具
memory-leak-detector(https://github.com/bytedance/memory-leak-detector)

```
(base) mingliu@192 raphael % python /Users/mingliu/_work/android_learn/memory-leak-detector/library/src/main/python/mmap.py -m maps   
========== maps ==========
17,026,457,600  totals
1,452,969,984   unknown
  137,805,824   library
  102,236,160   native
    7,208,960   object
    3,571,712   bss
    2,682,880   ttf
    1,966,080   linker
    1,888,256   hyb
      622,592   thread
      581,632   ashmem
       69,632   atexit
15,314,853,888  extras

```
```
(base) mingliu@192 raphael % python /Users/mingliu/_work/android_learn/memory-leak-detector/library/src/main/python/raphael.py -r ./report -o leak-doubts.txt

```

#### 原理

* ###### 先用PLT hook的方式，统计malloc/free等内存分配的函数，分配和释放了多少内存
* ###### 然后获取调用栈，堆栈信息还原
* ###### 每隔10分钟，用总分配减去总释放，看内存是否不断变大





