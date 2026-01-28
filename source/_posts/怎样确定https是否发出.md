title: 怎样确定https正确发出了
author: ming
date: 2026-01-28 20:39:57
tags:
---
### 怎样确定https正确发出了

#### 0. 前置信息
今天遇到一个问题，当我使用chrome请求一个url，得到的结果，与预期不一致，但是10多分钟后，再请求，又没问题，怀疑是服务端有缓存，但是后端开发说没缓存，所以我需要确定，http请求是否真的发出了，在进行操作之前，对于如下几件事，有些确定，有些不确定：
```
1. 我确定linux内核不会缓存我的http请求(虽然没看过kernel源码，但是坚信内核不会做这种事)
2. 我不太确定chrome是否会缓存我的http请求，尽管Cache-Control: no-cache\r\n
```


#### 1. 使用ping命令获得域名的ip地址
```
ping stg-jk.pingan.com.cn
PING stg-jk.pingan.com.cn (124.196.81.174): 56 data bytes
Request timeout for icmp_seq 0
Request timeout for icmp_seq 1
Request timeout for icmp_seq 2
```

#### 2. 查看这个ip匹配到哪个路由，这里是en0
```
route -n get 124.196.81.174
   route to: 124.196.81.174
destination: default
       mask: default
    gateway: 192.168.0.1
  interface: en0
      flags: <UP,GATEWAY,DONE,STATIC,PRCLONING,GLOBAL>
 recvpipe  sendpipe  ssthresh  rtt,msec    rttvar  hopcount      mtu     expire
       0         0         0         0         0         0      1500         0
``` 

#### 3. 确定使用的网卡，添加过滤规则
![upload successful](/images/445f4f63-582b-41ec-9ba3-7d313f364110.png)
```
ip.addr == 124.196.81.174
```

#### 4. 由于是https协议，所以抓到的包，是加密的。
```
1. 要先关到所有chrome，在命令行启动时，配置环境变量(因为进程fork时，写时拷贝，所以这个环境变量必然生效)
export SSLKEYLOGFILE=/tmp/sslkeys.log
open -a "Google Chrome"

2. 然后在wireshark中配置ssl密钥（/tmp/sslkeys.log）
Wireshark -> Preferences -> Protocols -> TLS -> (Pre)-Master-Secret log filename
```
#### 5. 抓包时，看到的就是明文了，此时可以进一步过滤，并且找到请求对应的response 
```
ip.addr == 124.196.81.174 && http
```
![upload successfull](/images/154ba8bf-bd38-423b-a2c1-dd13ee9a8648.png)
