title: 使用scapy模拟tcp的握手和挥手
author: ming
date: 2025-04-28 00:18:36
tags: 协议
---
### 0. 直接上结果


![upload successful](/images/pasted-5.png)
![upload successful](/images/pasted-4.png)
![upload successful](/images/pasted-3.png)
![upload successful](/images/pasted-2.png)
```python
#!/usr/bin/env python3
from scapy.all import *
import time

# 关闭IPv6警告
conf.ipv6_enabled = False
conf.L3socket = L3RawSocket

def complete_tcp_handshake(target_ip, target_port):
    """
    使用Scapy模拟完整的TCP三次握手过程
    """
    # 确保保存相同的源端口
    source_port = RandShort()._fix()  # _fix() 确保值被固定而不是每次调用都随机生成

    # 初始序列号
    seq_num = random.randint(1000, 9000)
    
    print(f"[+] 开始TCP连接到 {target_ip}:{target_port} seq {seq_num}")
    print(f"[+] 使用源端口: {source_port}")
    
    # 第一步: 发送SYN包
    print(f"[+] 步骤1: 发送SYN包...")
    syn_packet = IP(dst=target_ip)/TCP(sport=source_port, dport=target_port, flags="S", seq=seq_num)
    # 打印完整的包信息
    print(f"[+] 发送包详情: {syn_packet.summary()}")
    
    # 增加超时时间，并启用详细输出
    syn_ack_response = sr1(syn_packet, timeout=5, verbose=1)
    
    if syn_ack_response is None:
        print("[-] 未收到响应，目标可能关闭或被过滤")
        # 尝试使用普通套接字测试端口
        try:
            import socket
            s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            s.settimeout(2)
            result = s.connect_ex((target_ip, target_port))
            if result == 0:
                print(f"[+] 使用普通套接字连接到 {target_ip}:{target_port} 成功，但Scapy无法获取响应")
            else:
                print(f"[-] 使用普通套接字连接到 {target_ip}:{target_port} 失败，错误码: {result}")
            s.close()
        except Exception as e:
            print(f"[-] 套接字测试异常: {e}")
        return False
    
    # 打印接收到的完整响应
    print(f"[+] 收到响应: {syn_ack_response.summary()}")
    
    # 检查是否收到SYN-ACK响应
    if not syn_ack_response.haslayer(TCP) or syn_ack_response[TCP].flags != 0x12:
        print("[-] 没有收到预期的SYN-ACK响应")
        print(f"[-] 收到的标志: {syn_ack_response[TCP].flags}")
        return False
    
    print(f"[+] 步骤2: 收到SYN-ACK响应，服务器序列号={syn_ack_response[TCP].seq}, 确认号={syn_ack_response[TCP].ack}")
    
    # 第三步: 发送ACK确认
    server_seq = syn_ack_response[TCP].seq
    ack_packet = IP(dst=target_ip)/TCP(
        sport=source_port, 
        dport=target_port, 
        flags="A", 
        seq=syn_ack_response[TCP].ack,  # 使用服务器确认的序列号
        ack=server_seq + 1  # 确认服务器的序列号+1
    )
    
    # 发送ACK包
    print(f"[+] 步骤3: 发送ACK包...")
    send(ack_packet, verbose=1)
    
    print(f"[+] TCP三次握手完成!")


    print(f"[+] TCP连接已建立！")

    # 可选: 发送一些数据
    print(f"[+] 发送一些数据...")
    data_packet = IP(dst=target_ip)/TCP(
        sport=source_port, 
        dport=target_port, 
        flags="PA", 
        seq=syn_ack_response[TCP].ack,
        ack=server_seq + 1
    )/Raw(load="Hello from Scapy!")
    
    data_response = sr1(data_packet, timeout=2, verbose=0)
    
    # time.sleep(20)
    if data_response and data_response.haslayer(TCP):
        print(f"[+] 收到数据响应!")
    
    # 关闭连接 (发送FIN包)
    print(f"[+] 发送FIN包开始关闭连接...")
    fin_packet = IP(dst=target_ip)/TCP(
        sport=source_port, 
        dport=target_port, 
        flags="FA", 
        seq=syn_ack_response[TCP].ack + len("Hello from Scapy!"),
        ack=server_seq + 1
    )
    
    fin_response = sr1(fin_packet, timeout=2, verbose=0)
    
    if fin_response and fin_response.haslayer(TCP) and fin_response[TCP].flags & 0x11:  # FIN or FIN-ACK
        # 发送最后的ACK
        last_ack = IP(dst=target_ip)/TCP(
            sport=source_port, 
            dport=target_port, 
            flags="A", 
            seq=fin_response[TCP].ack,
            ack=fin_response[TCP].seq + 1
        )
        send(last_ack, verbose=0)
        print(f"[+] TCP连接已正常关闭")
    
    return True

if __name__ == "__main__":
    # target_ip = "127.0.0.1"
    # target_port = 8000
    target_ip = "192.168.1.75"
    target_port = 8000
    
    complete_tcp_handshake(target_ip, target_port)

```

### 1. 环境搭建
* macos(server端)
* ubuntu（client端）
* python3 (anaconda)
* 用到的库和工具(scapy/nc/tcpdump/ssh等)

```bash
# 在macos上，启动server端，之所以没在没用ubuntu/localhost做server端，原因等下说
nc -lk 8000
```

```bash
# 查看网卡
tcpdump -D

# 抓包
sudo tcpdump -i any port 8000 -w 8000.pcap

```

```
# 在命令行执行自己写的python脚本，必须用sudo，因为scapy要直接读写rawsocket，需要超级权限
sudo python3 test.py
```

### 2. 遇到的问题
1. 最开始使用ubuntu的docker做server端，映射了端口到宿主机（macos），但是发现只能在macos上抓到syn，docker内，完全抓不到包。
2. 改成localhost做server端，可以抓到包，但是内核会帮你回复rst.
3. 换成用macos做server端，用一台真实的ubuntu做client，是可以抓到包，但是发了syn后，服务端回复syn+ack后，client直接回复rst（其实是内核回复的）。
4. 修改了出站规则后，问题2被解决，但是新问题出现了，就是我明明发了一个syn，但是抓到的包里有两个syn，而且源端口号不同



### 3. 怎样解决
```
sudo iptables -A OUTPUT -p tcp --tcp-flags RST RST -s 192.168.1.66 -j DROP

(base) ming@ming-Thurley:~/_work/procotol$ sudo iptables -L
[sudo] password for ming:
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
DROP       tcp  --  192.168.1.66         anywhere             tcp flags:RST/RST
DROP       tcp  --  localhost            anywhere             tcp flags:RST/RST
```

### 4. 总结