title: "iptables 介绍"
author: ming
date: 2026-03-10 23:12:31
tags: 协议
---
# iptables 介绍

## 是什么

iptables 是 Linux 内核中 **netfilter** 框架的用户态管理工具，用于配置内核的包过滤规则。它工作在网络层（L3）和传输层（L4），可以对进出系统的网络包进行过滤、NAT、修改等操作。

---

## 核心概念：表（Table）和链（Chain）

iptables 用 **表** 来组织不同功能，每张表包含若干 **链**，链里存放具体的 **规则**。

### 四张表（按优先级）

| 表名 | 用途 |
|------|------|
| `raw` | 连接跟踪前的处理，优先级最高 |
| `mangle` | 修改包头字段（TTL、TOS 等） |
| `nat` | 网络地址转换（SNAT / DNAT） |
| `filter` | **默认表**，包过滤（ACCEPT / DROP） |

### 五条内置链（对应 netfilter 钩子）

```
外部数据包
    │
    ▼
PREROUTING ──→ 路由判断 ──→ FORWARD ──→ POSTROUTING ──→ 出网卡
                   │                          ▲
                   ▼                          │
                INPUT                      OUTPUT
                   │                          ▲
                   ▼                          │
              本地进程 ─────────────────────────
```

---

## 规则匹配与动作

每条规则 = **匹配条件** + **目标动作（Target）**

常见 Target：
- `ACCEPT` — 放行
- `DROP` — 静默丢弃
- `REJECT` — 拒绝并回应
- `LOG` — 记录日志
- `SNAT / DNAT / MASQUERADE` — 地址转换
- `RETURN` — 返回父链

---

## 常用命令

```bash
# 查看 filter 表规则（带行号）
iptables -L -n -v --line-numbers

# 查看 nat 表
iptables -t nat -L -n -v

# 放行 80 端口入站
iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# 拒绝某个 IP
iptables -A INPUT -s 192.168.1.100 -j DROP

# SNAT：内网出口伪装
iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -o eth0 -j MASQUERADE

# DNAT：端口转发（外部 8080 → 内部 192.168.1.10:80）
iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to-destination 192.168.1.10:80

# 删除规则（按行号）
iptables -D INPUT 3

# 保存规则（Debian/Ubuntu）
iptables-save > /etc/iptables/rules.v4
```

---

## 包处理完整流程

```
入站包:   PREROUTING(raw→mangle→nat) → INPUT(mangle→filter) → 进程
转发包:   PREROUTING → FORWARD(mangle→filter) → POSTROUTING(mangle→nat)
出站包:   OUTPUT(raw→mangle→nat→filter) → POSTROUTING(mangle→nat)
```

---

## iptables vs nftables

| | iptables | nftables |
|--|--|--|
| 内核版本 | 2.4+ | 3.13+ |
| 规则语法 | 分散（每个表独立命令） | 统一语法 |
| 性能 | 线性匹配 | 支持集合/映射，更高效 |
| 现状 | 仍广泛使用 | 新发行版默认（iptables 已是 nft 的 shim） |

现代系统（如 Ubuntu 22.04+）的 `iptables` 命令实际上是 `iptables-nft` 的软链接，底层已经是 nftables 了。

---

## 与你熟悉的场景对应

- **Docker**：大量使用 iptables 的 nat 表和 filter 表来实现容器网络隔离和端口映射
- **Tailscale / Cilium**：也会操作 iptables 或直接用 eBPF 绕过它
- **eBPF 的崛起**：XDP 和 TC hook 可以在 netfilter 之前处理包，性能更高，是未来的方向