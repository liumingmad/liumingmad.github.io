title: "http1.1是怎样升级到http2.0\b的？"
author: ming
date: 2019-03-27 23:12:31
tags:
---
**** 背景知识1：****
1. http协议是是基于tcp之上的协议。
2. http2.0之前的版本是http1.1

*** 目标：***
* 让client与server用http2.0通信


*** 问题推演：***

*** 在不知道server端的http版本的情况下，想要使用http2.0，需要什么手段？***

* 因为通信之前，server和client都彼此不知道对方的http的协议版本，所以client只能先使用http1.1尝试通信
* 因为client想要升级到http2.0，所以需要与server端协商使用http的版本，目前有两种协商协议：NPN和ALPN


*** NPN和ALPN的区别是什么？***
* NPN 是服务端发送所支持的 HTTP 协议列表，由客户端选择；而 ALPN 是客户端发送所支持的 HTTP 协议列表，由服务端选择；
* NPN 的协商结果是在 Change Cipher Spec 之后加密发送给服务端；而 ALPN 的协商结果是通过 Server Hello 明文发给客户端；


*** 用wireshark抓包（ALPN）发现，app中okhttp的请求，已经是http2.0了，为什么log中还显示http1.1？***
![](/images/7693D9E4025DAF2A7197E6CE7420E190.jpg)

![](/images/E6BD85B420B431330997FBD3F7367D1A.jpg)

* 因为添加拦截器（HttpLoggingInterceptor）的层级不对。
1. addInterceptor添加的拦截器，打印请求行log时，长链接还没连上。
2. addNetworkInterceptor是在协商之后才打印日志。