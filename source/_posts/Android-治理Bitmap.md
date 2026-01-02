title: <Android>治理Bitmap
author: ming
date: 2025-05-03 17:06:04
tags:
---
#### 工具Lancet
#####  能通过ASM+注解器+gradle插件的方式，实现字节码插桩

#### 原理
##### 1. 拦截Bitmap.createBitmap
##### 2. 对于图片尺寸超过屏幕宽高，等比缩放。对于格式是ARGB_8888，降为RGB_565
##### 3. 对于Bitmap泄漏，当Bitmap被GC挥手回收后，NativeAllocationRegistry可以辅助Native内存回收。