title: curl代理
author: ming
date: 2018-07-20 12:11:38
tags:
---
```bash
curl ip.cn
# 当前 IP：xxx.xxx.xxx.xxx 来自：广东省深圳市
# 设置环境变量：
export ALL_PROXY=socks5://192.168.3.3:1080
curl ip.cn
# 当前 IP：xxx.xxx.xxx.xxx 来自：台湾省 Google
```