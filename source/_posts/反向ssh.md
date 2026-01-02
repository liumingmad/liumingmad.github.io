title: 反向ssh
author: ming
date: 2018-07-03 12:30:55
tags:
---
目标：从外网访问内网的某台机器
步骤：
1. 需要让三台ssh互相持有公钥

2. 在内网机器执行
autossh -M 6768 -fCNR 6767:localhost:22 ming@xx.xx.xx.xx

3. 在外网服务器执行
ssh -fCNL *:1236:localhost:6767 ming@localhost

4. [使用SSH反向隧道进行内网穿透](http://arondight.me/2016/02/17/%E4%BD%BF%E7%94%A8SSH%E5%8F%8D%E5%90%91%E9%9A%A7%E9%81%93%E8%BF%9B%E8%A1%8C%E5%86%85%E7%BD%91%E7%A9%BF%E9%80%8F/)