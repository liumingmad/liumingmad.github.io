title: <Android>内存优化概览
author: ming
date: 2025-05-03 16:50:46
tags:
---
#### 1. 内存泄漏
* ##### java内存泄漏
	
* ##### native内存泄漏

#### 2. 治理Bitmap

#### 3. 对线程池分类，用于计算和逻辑处理的线程池和用于io的线程池。对于io线程，可以减小线程栈空间的大小。

#### 4. 默认webview内存，在64位机1G，在32位机130M，在非ARM机上是190M。如果app没用到webview，就应该释放掉。虽然只是虚拟内存。

#### 5. 查看内存占用情况
```
(base) mingliu@192 tmp % adb shell ps | grep ming
u0_a207       3847   373   16557464 119520 do_freezer_trap     0 S org.ming.mgo
u0_a209      19540   373   16499788 163272 do_epoll_wait       0 S org.ming.test_native
(base) mingliu@192 tmp % adb shell dumpsys meminfo 19540
Applications Memory Usage (in Kilobytes):
Uptime: 101223963 Realtime: 101223963

** MEMINFO in pid 19540 [org.ming.test_native] **
                   Pss  Private  Private  SwapPss      Rss     Heap     Heap     Heap
                 Total    Dirty    Clean    Dirty    Total     Size    Alloc     Free
                ------   ------   ------   ------   ------   ------   ------   ------
  Native Heap     8730     8692        4       62    10244    32684    26121     1956
  Dalvik Heap     1186     1064        0       98     5280     5302     2651     2651
 Dalvik Other     5021     1756        0        3     8692                           
        Stack      476      476        0        0      480                           
       Ashmem       17        0        0        0      560                           
    Other dev       12        0       12        0      324                           
     .so mmap     1947      164        0       12    32436                           
    .jar mmap     1739        0        0        0    40864                           
    .apk mmap    28728       60    28324        0    30552                           
    .ttf mmap      209        0        0        0     1080                           
    .dex mmap       18        0        0        0      848                           
    .oat mmap       40        0        0        0     1560                           
    .art mmap     8675     7804       16      111    25436                           
   Other mmap     1702        8        0        0     4212                           
      Unknown      590      532       48        0     1072                           
        TOTAL    59376    20556    28404      286   163640    37986    28772     4607
 
 App Summary
                       Pss(KB)                        Rss(KB)
                        ------                         ------
           Java Heap:     8884                          30716
         Native Heap:     8692                          10244
                Code:    28664                         113976
               Stack:      476                            480
            Graphics:        0                              0
       Private Other:     2244
              System:    10416
             Unknown:                                    8224
 
           TOTAL PSS:    59376            TOTAL RSS:   163640       TOTAL SWAP PSS:      286
 
 Objects
               Views:        6         ViewRootImpl:        1
         AppContexts:        5           Activities:        1
              Assets:       20        AssetManagers:        0
       Local Binders:       14        Proxy Binders:       52
       Parcel memory:        4         Parcel count:       16
    Death Recipients:        0             WebViews:        0
 
 SQL
         MEMORY_USED:        0
  PAGECACHE_OVERFLOW:        0          MALLOC_SIZE:        0

```