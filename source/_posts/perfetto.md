title: perfetto
author: ming
date: 2024-02-15 22:23:58
tags:
---
#### 录制trace
```shell
./record_android_trace -o mgo_2.perfetto-trace -t 30s -b 100mb -a org.ming.mgo sched freq idle am wm gfx view binder_driver hal dalvik camera input res memory sched/sched_switch raw_syscalls/sys_enter raw_syscalls/sys_exit
```

#### 查看app内存概览
```
mingliu@mingdeMacBook-Pro perfetto % adb shell dumpsys meminfo org.ming.mgo
Applications Memory Usage (in Kilobytes):
Uptime: 455229664 Realtime: 455229664

** MEMINFO in pid 5369 [org.ming.mgo] **
                   Pss  Private  Private  SwapPss      Rss     Heap     Heap     Heap
                 Total    Dirty    Clean    Dirty    Total     Size    Alloc     Free
                ------   ------   ------   ------   ------   ------   ------   ------
  Native Heap    12523    12464       36       53    13460    22464    16561     1277
  Dalvik Heap     1912     1892        0      183     2548     6915     3474     3441
 Dalvik Other     1586     1536        0       10     1868
        Stack      480      480        0        0      488
       Ashmem       15        0        0        0      396
    Other dev        5        4        0        0      312
     .so mmap     5222      252     2700       13    33240
    .jar mmap     1674        0      244        0    26772
    .apk mmap      369        8       76        0     2204
    .ttf mmap      244        0      100        0      708
    .dex mmap    12533        4    12500        0    13236
    .oat mmap      541        0        4        0    11820
    .art mmap     1069      556       32      278    16092
   Other mmap       62        8       16        0      616
      Unknown      541      504       32        8      916
        TOTAL    39321    17708    15740      545   124676    29379    20035     4718

 App Summary
                       Pss(KB)                        Rss(KB)
                        ------                         ------
           Java Heap:     2480                          18640
         Native Heap:    12464                          13460
                Code:    15888                          88088
               Stack:      480                            488
            Graphics:        0                              0
       Private Other:     2136
              System:     5873
             Unknown:                                    4000

           TOTAL PSS:    39321            TOTAL RSS:   124676       TOTAL SWAP PSS:      545

 Objects
               Views:       24         ViewRootImpl:        3
         AppContexts:        9           Activities:        3
              Assets:       18        AssetManagers:        0
       Local Binders:       21        Proxy Binders:       44
       Parcel memory:        5         Parcel count:       20
    Death Recipients:        0             WebViews:        0

 SQL
         MEMORY_USED:        0
  PAGECACHE_OVERFLOW:        0          MALLOC_SIZE:        0

```

#### 堆转储
```

```