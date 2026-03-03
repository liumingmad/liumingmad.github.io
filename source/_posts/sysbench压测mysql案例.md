title: sysbench压测mysql案例
author: ming
date: 2026-03-03 20:35:00
tags: 协议
---

### 0. 背景
* 这是plantegg在知识星球上的一个实验案例
* 我们的数据库需要做在线升级，所以构造了一个测试环境，客户端Sysbench 用长连接一直打压力，同时Server 端的数据库做在线升级，这个在线升级会让 Server进程重启，所以毫无疑问连接会断开重连，所以期望升级的时候 Sysbench端 QPS 跌0几秒钟然后快速恢复，但是每次升级都是 Sysbench端 QPS永久跌0，再也不能恢复，所以需要分析为什么，问题出在哪里？有人说是服务端的问题因为只有服务端做了变更。

### 1. 实验环境
* 服务端的环境
```shell
[ming@iZwz93i14h0qw5ez8anc3cZ ~]$ uname -r
5.10.134-19.2.al8.x86_64
[ming@iZwz93i14h0qw5ez8anc3cZ ~]$ cat /etc/redhat-release
Alibaba Cloud Linux release 3 (OpenAnolis Edition)
```
```
#安装docker/sysbench/mysql/tcpdump

#先跑一个mysql
docker run -it -d --net=host -e MYSQL_ROOT_PASSWORD=123 --name=plantegg mysql

#连接
mysql -h127.1 --ssl-mode=DISABLED -uroot -p123

#密码问题
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123';

#创建一个数据库
mysql -h127.1 --ssl-mode=DISABLED -uroot -p123 -e "create database test"

#随机生成点数据
sysbench --mysql-user='root' --mysql-password='123' --mysql-db='test' --mysql-host='127.0.0.1' --mysql-port='3306' --tables='16'  --table-size='10000' --range-size='5' --db-ps-mode='disable' --skip-trx='on' --mysql-ignore-errors='all' --time='1180' --report-interval='1' --histogram='on' --threads=1 oltp_read_only prepare

#启动sysbench
sysbench --mysql-user='root' --mysql-password='123' --mysql-db='test' --mysql-host='127.0.0.1' --mysql-port='3306' --tables='16'  --table-size='10000' --range-size='5' --db-ps-mode='disable' --skip-trx='on' --mysql-ignore-errors='all' --time='1180' --report-interval='1' --histogram='on' --threads=1 oltp_read_only run
```


* 客户端有两种，使用ubuntu的是没问题的
```shell
# 有问题
Alibaba Cloud Linux release 3
# 没问题
Ubuntu 22.04.2 LTS (GNU/Linux 6.12.68-linuxkit aarch64)
```
  
### 2. 分析过程
* 开始的时候问题没有这么清晰，每次升级才能稳定重现，后来想要定位问题就必须降低重现难度，考虑到重启客户端ECS 就能恢复，于是：
* 不再重启ECS，只重启 Sysbench —— 能恢复
* 不真正升级只重启Server —— 问题能稳定重现，重现容易很多了
* 不重启 Server，只是kill掉Sysbench 的一条连接 —— 能重现
* 将Sysbench 连接数从最开始100个，改成1个压 Server，然后 kill 掉 Sysbench 的一条连接 —— 能重现

* 至此，通过show processlist;查看连接，会发现大量Connect
```
 #mysql -h127.1 --ssl-mode=DISABLED -uroot -p123 test
 ​
 mysql> show processlist;
 +------+-----------------+-----------------+------+---------+---------+------------------------+------------------+
 | Id   | User            | Host            | db   | Command | Time    | State                  | Info             |
 +------+-----------------+-----------------+------+---------+---------+------------------------+------------------+
 |    5 | event_scheduler | localhost       | NULL | Daemon  | 5715214 | Waiting on empty queue | NULL             |
 | 5736 | root            | 127.0.0.1:36588 | test | Sleep   |       0 |                        | NULL             |
 | 5737 | root            | 127.0.0.1:50964 | test | Query   |       0 | init                   | show processlist |
 +------+-----------------+-----------------+------+---------+---------+------------------------+------------------+
 3 rows in set, 1 warning (0.00 sec)
 ​
 mysql> kill 5736;
 Query OK, 0 rows affected (0.01 sec)
 ​
 mysql> show processlist;
 +------+----------------------+-----------------+------+---------+---------+------------------------+------------------+
 | Id   | User                 | Host            | db   | Command | Time    | State                  | Info             |
 +------+----------------------+-----------------+------+---------+---------+------------------------+------------------+
 |    5 | event_scheduler      | localhost       | NULL | Daemon  | 5715249 | Waiting on empty queue | NULL             |
 | 5737 | root                 | 127.0.0.1:50964 | test | Query   |       0 | init                   | show processlist |
 | 5738 | unauthenticated user | 127.0.0.1:50334 | NULL | Connect |       1 | Receiving from client  | NULL             |
 | 5739 | unauthenticated user | 127.0.0.1:50346 | NULL | Connect |       1 | Receiving from client  | NULL             |
 | 5740 | unauthenticated user | 127.0.0.1:50348 | NULL | Connect |       1 | Receiving from client  | NULL             |
 | 5741 | unauthenticated user | 127.0.0.1:50352 | NULL | Connect |       1 | Receiving from client  | NULL             |
 | 5742 | unauthenticated user | 127.0.0.1:50354 | NULL | Connect |       1 | Receiving from client  | NULL             |
 | 5743 | unauthenticated user | 127.0.0.1:50370 | NULL | Connect |       1 | Receiving from client  | NULL             |
 | 5744 | unauthenticated user | 127.0.0.1:50378 | NULL | Connect |       1 | Receiving from client  | NULL             |
 | 5745 | unauthenticated user | 127.0.0.1:50386 | NULL | Connect |       1 | Receiving from client  | NULL             |
 | 5746 | unauthenticated user | 127.0.0.1:50402 | NULL | Connect |       1 | Receiving from client  | NULL             |
 | 5747 | unauthenticated user | 127.0.0.1:50408 | NULL | Connect |       1 | Receiving from client  | NULL             |
 | 5748 | unauthenticated user | 127.0.0.1:50412 | NULL | Connect |       1 | Receiving from client  | NULL             |
 | 5749 | unauthenticated user | 127.0.0.1:50420 | NULL | Connect |       1 | Receiving from client  | NULL             |
 | 5750 | unauthenticated user | 127.0.0.1:50430 | NULL | Connect |       1 | Receiving from client  | NULL             |
 | 5751 | unauthenticated user | 127.0.0.1:50438 | NULL | Connect |       1 | Receiving from client  | NULL             |
 | 5752 | unauthenticated user | 127.0.0.1:50448 | NULL | Connect |       1 | Receiving from client  | NULL             |
 | 5753 | unauthenticated user | 127.0.0.1:50456 | NULL | Connect |       1 | Receiving from client  | NULL             |
 | 5754 | unauthenticated user | 127.0.0.1:50468 | NULL | Connect |       1 | Receiving from client  | NULL             |
 | 5755 | unauthenticated user | 127.0.0.1:50480 | NULL | Connect |       1 | Receiving from client  | NULL             |
 | 5756 | unauthenticated user | 127.0.0.1:50492 | NULL | Connect |       1 | Receiving from client  | NULL             |
 | 5757 | unauthenticated user | 127.0.0.1:50504 | NULL | Connect |       1 | Receiving from client  | NULL             |
 | 5758 | unauthenticated user | 127.0.0.1:50520 | NULL | Connect |       1 | Receiving from client  | NULL             |
 | 5759 | unauthenticated user | 127.0.0.1:50522 | NULL | Connect |       1 | Receiving from client  | NULL             |
 +------+----------------------+-----------------+------+---------+---------+------------------------+------------------+
 153 rows in set, 1 warning (0.00 sec)
```

* 通过GPT得知：
```
	•	User = unauthenticated user → 还没验证身份
	•	Command = Connect → 正在尝试连接
	•	Time = 6 → 已经卡了 6 秒
	•	State = Receiving from client → 正在等客户端发认证数据
``` 

* 另外通过ss -tn | grep 3306, 得到大量连接，FIN-WAIT-2表示，本机是主动关闭的一方，并且对方回复了ack
```
FIN-WAIT-2 0      0      [::ffff:127.0.0.1]:3306  [::ffff:127.0.0.1]:40672
FIN-WAIT-2 0      0      [::ffff:127.0.0.1]:3306  [::ffff:127.0.0.1]:33748
FIN-WAIT-2 0      0      [::ffff:127.0.0.1]:3306  [::ffff:127.0.0.1]:57406
FIN-WAIT-2 0      0      [::ffff:127.0.0.1]:3306  [::ffff:127.0.0.1]:40094
FIN-WAIT-2 0      0      [::ffff:127.0.0.1]:3306  [::ffff:127.0.0.1]:58176
FIN-WAIT-2 0      0      [::ffff:127.0.0.1]:3306  [::ffff:127.0.0.1]:50176
FIN-WAIT-2 0      0      [::ffff:127.0.0.1]:3306  [::ffff:127.0.0.1]:49342
FIN-WAIT-2 0      0      [::ffff:127.0.0.1]:3306  [::ffff:127.0.0.1]:53442
FIN-WAIT-2 0      0      [::ffff:127.0.0.1]:3306  [::ffff:127.0.0.1]:42282
FIN-WAIT-2 0      0      [::ffff:127.0.0.1]:3306  [::ffff:127.0.0.1]:59408
FIN-WAIT-2 0      0      [::ffff:127.0.0.1]:3306  [::ffff:127.0.0.1]:55334
ESTAB      0      0      [::ffff:127.0.0.1]:3306  [::ffff:127.0.0.1]:53444
FIN-WAIT-2 0      0      [::ffff:127.0.0.1]:3306  [::ffff:127.0.0.1]:54006
FIN-WAIT-2 0      0      [::ffff:127.0.0.1]:3306  [::ffff:127.0.0.1]:58254
FIN-WAIT-2 0      0      [::ffff:127.0.0.1]:3306  [::ffff:127.0.0.1]:37044
FIN-WAIT-2 0      0      [::ffff:127.0.0.1]:3306  [::ffff:127.0.0.1]:60838
FIN-WAIT-2 0      0      [::ffff:127.0.0.1]:3306  [::ffff:127.0.0.1]:52766
FIN-WAIT-2 0      0      [::ffff:127.0.0.1]:3306  [::ffff:127.0.0.1]:37436
FIN-WAIT-2 0      0      [::ffff:127.0.0.1]:3306  [::ffff:127.0.0.1]:34862
FIN-WAIT-2 0      0      [::ffff:127.0.0.1]:3306  [::ffff:127.0.0.1]:32978
FIN-WAIT-2 0      0      [::ffff:127.0.0.1]:3306  [::ffff:127.0.0.1]:43728
FIN-WAIT-2 0      0      [::ffff:127.0.0.1]:3306  [::ffff:127.0.0.1]:35108
```

* 至此，问题基本已经猜到是客户端有问题了，不然哪来的这么多连接？
* 再抓包确认下
```
# 先确认下网卡，虽然127.0.0.1显然是lo
ip route get 127.0.0.1

# 查看网卡名
tcpdump -D
1.eth0 [Up, Running]
2.lo [Up, Running, Loopback]
3.any (Pseudo-device that captures on all interfaces) [Up, Running]
4.docker0 [Up]
5.bluetooth-monitor (Bluetooth Linux Monitor) [none]
6.nflog (Linux netfilter log (NFLOG) interface) [none]
7.nfqueue (Linux netfilter queue (NFQUEUE) interface) [none]
8.usbmon0 (Raw USB traffic, all USB buses) [none]

# 抓包并保存到文件
sudo tcpdump -i lo port 3306 -w 3306.pcap

# 拷贝回本地
scp ming@ali-sz:/home/ming/3306.pcap ./
```

* 用wireshark打开
    * **正常情况**
    ![alt text](/images/32839910012.png)

    * **kill掉连接后**
    ![alt text](/images/kkdklsd4343434.png)

* 显然，当客户端重连时，没有走正常鉴权的协议，等了10秒，服务端超时，于是主动把连接fin掉。而客户端没有正常挥手，导致服务端的状态是FIN-WAIT-2。客户端应用在调用 close() 时，接收缓冲区中还有未读取的数据（即 Error 1159 那条消息和 FIN）。当 socket 被关闭但 receive buffer 中仍有未被应用层读取的数据时，内核不会走正常的 FIN 流程，而是直接发送 RST，表示异常终止（abortive close）。

