title: <Android>java内存泄漏
author: ming
date: 2025-04-30 16:44:50
tags:
---
### 工具
#### 0. 举个内存泄漏的例子
```java
# 非静态内部类
        Handler handler = new Handler(Looper.getMainLooper());
        handler.postDelayed(new Runnable() {
            @Override
            public void run() {
                Log.i("ming_test", "post message");
            }
        }, 10 * 60 * 1000);
```


#### 1. 获取heap dump
```
# 堆内存快照
adb shell am dumpheap org.ming.mgo /data/local/tmp/ming_demo.hprof

# 拉到本地
adb pull /data/local/tmp/ming_demo.hprof ./

# 为MAT转换格式
hprof-conv ming_demo.hprof ming_demo_conv.hprof
```

#### 2. 使用Android studio profiler分析

![upload successful](/images/pasted-6.png)


#### 3. 使用MAT分析
```
# 如果已经知道泄漏的对象了，用MAT分析引用链
```

![upload successful](/images/pasted-7.png)