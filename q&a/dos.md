IPv6 网络里有一种攻击：

```
NDP exhaustion attack
```

攻击者发送大量：

```
Neighbor Solicitation
```

去解析大量 **不存在的 IPv6 地址**。

结果：

- 设备不停做 NDP resolution
- neighbor table 被填满
- CPU 占用暴涨
- 网络瘫痪

这就是 **DoS**。



# 3 Destination Guard 的解决办法

Destination Guard 会：

```
检查 destination address
```

如果：

```
destination 不在 binding table
```

直接：

```
drop
```

所以：

```
不进行 NDP resolution
```

这样就防止了：

```
NDP flooding
Neighbor cache exhaustion
DoS
```

------

# 4 Binding Table 来源

Destination Guard 依赖：

```
IPv6 Snooping binding table
```

表项来自：

```
DHCPv6
NDP
Static binding
```

------

# 5 机制流程

```
packet arrives
     │
     ▼
destination guard check
     │
     ├─ destination 在 binding table
     │        │
     │        ▼
     │      forward
     │
     └─ destination 不在 binding table
              │
              ▼
            DROP
```

------

# 6 为什么不是 RA Guard

RA Guard 解决的是：

```
Rogue Router
```

攻击类型：

```
RA spoofing
```

它不是专门解决 DoS。

------

# 7 IPv6 First Hop Security 对比表

| Feature               | 防护               |
| --------------------- | ------------------ |
| **RA Guard**          | Rogue router       |
| **DHCPv6 Guard**      | Rogue DHCP         |
| **Source Guard**      | IP spoofing        |
| **ND Inspection**     | Neighbor spoofing  |
| **Destination Guard** | **NDP DoS attack** |

很好，这个问题其实是**网络安全最基础的概念之一**。我们把 **DoS** 从最直观的角度解释，你就能完全理解 IPv6 那题为什么提到它。

------

# 1 什么是 DoS

**DoS = Denial of Service（拒绝服务）**

意思就是：

> **让正常用户无法使用网络服务。**

攻击者不一定要“黑进去”，
只要让系统 **忙到无法工作**，目的就达到了。

------

# 2 一个最直观的现实类比

想象一家餐厅：

- 餐厅只有 **10 张桌子**
- 正常顾客来吃饭

突然有攻击者：

- 打电话订 **10000 个座位**
- 但没人来

结果：

```
餐厅资源被占满
真正顾客进不来
```

这就是：

```
Denial of Service
```

------

# 3 网络里的 DoS

网络设备也有资源限制：

| 资源             | 限制     |
| ---------------- | -------- |
| CPU              | 处理数据 |
| Memory           | 存表     |
| Connection table | 会话     |
| ARP/NDP table    | 地址解析 |

攻击者如果让这些资源 **耗尽**：

```
正常流量就无法处理
```

------

# 4 一个经典 DoS 例子

假设服务器支持：

```
1000 TCP connections
```

攻击者发送：

```
100000 SYN packets
```

服务器必须：

```
记录每个连接
```

结果：

```
connection table 满
```

真实用户：

```
无法连接
```

------

# 5 网络设备里的 DoS

路由器和交换机也一样。

比如 IPv6 里的 **NDP 表**：

```
neighbor table
```

如果攻击者发很多：

```
neighbor request
```

设备就要不断：

```
查找
创建表项
```

最终：

```
table overflow
CPU 100%
```

网络就瘫痪了。

------

# 6 IPv6 里的 DoS 例子

攻击者发送：

```
Neighbor Solicitation
for
2001::1
2001::2
2001::3
2001::4
...
```

这些地址其实：

```
根本不存在
```

但设备仍然要：

```
尝试解析
```

这就是：

```
NDP DoS attack
```

------

# 7 Destination Guard 为什么能防 DoS

Destination Guard 的策略是：

```
只允许解析 binding table 里的地址
```

如果地址：

```
不在 binding table
```

直接：

```
DROP
```

设备就不会去：

```
NDP resolution
```

所以：

```
CPU 不会被拖死
```

------

# 8 DoS vs DDoS

| 类型 | 含义       |
| ---- | ---------- |
| DoS  | 一个攻击源 |
| DDoS | 多个攻击源 |

DDoS 就是：

```
Distributed Denial of Service
```

比如：

```
10000 infected computers
一起攻击
```

------

# 9 常见 DoS 攻击类型

| 类型              | 攻击                         |
| ----------------- | ---------------------------- |
| SYN flood         | TCP 连接表耗尽               |
| ICMP flood        | CPU overload                 |
| DNS amplification | 流量放大                     |
| ARP flood         | ARP table overflow           |
| NDP flood         | IPv6 neighbor table overflow |

------

# 10 网络设备常见防 DoS 技术

| 技术              | 作用            |
| ----------------- | --------------- |
| CoPP              | 保护控制平面    |
| RA Guard          | 防 rogue router |
| DHCP Guard        | 防 rogue DHCP   |
| Source Guard      | 防 IP spoof     |
| Destination Guard | 防 NDP DoS      |

------

# 11 一句话理解 DoS

最简单理解：

```
DoS = 把设备忙死
```

不是破解系统，而是：

```
消耗资源
```

------

对的 👍。
你理解得很正确：**这些资源都可以成为 DoS 攻击的目标**。
DoS 的本质就是 **耗尽设备资源（resource exhaustion）**。

我们把你列的四个资源分别讲清楚，并给你真实网络里的攻击例子。

------

# 1 CPU DoS（最常见）

攻击目标：**让设备 CPU 100%**

方法：发送大量需要 **CPU 处理的控制平面流量**。

例如：

| 攻击                   | 解释                  |
| ---------------------- | --------------------- |
| ICMP flood             | 大量 ping             |
| Routing protocol flood | OSPF / BGP packet     |
| NDP flood              | IPv6 neighbor request |
| ARP flood              | ARP request           |

结果：

```
CPU 100%
路由计算变慢
设备卡死
```

防护技术：

```
CoPP (Control Plane Policing)
```

------

# 2 Memory DoS

攻击目标：**耗尽设备内存**

例如：

| 攻击            | 解释             |
| --------------- | ---------------- |
| DHCP starvation | 创建大量租约     |
| NDP table flood | IPv6 neighbor 表 |
| ARP table flood | IPv4 ARP 表      |

结果：

```
memory exhausted
新连接无法建立
```

------

# 3 Connection table DoS

攻击目标：**耗尽连接表**

最经典攻击：

```
SYN flood
```

攻击流程：

```
client -> SYN
server -> SYN ACK
client 不回 ACK
```

服务器必须保存状态：

```
half-open connection
```

如果攻击者发很多：

```
connection table 满
```

真实用户：

```
无法连接
```

防护：

```
SYN cookies
rate limit
firewall protection
```

------

# 4 ARP / NDP table DoS

攻击目标：

```
ARP table
IPv6 neighbor table
```

攻击方法：

发送大量：

```
ARP request
Neighbor Solicitation
```

例如：

```
Who has 10.1.1.1
Who has 10.1.1.2
Who has 10.1.1.3
...
```

设备会：

```
创建表项
```

如果数量很多：

```
table overflow
```

结果：

```
正常主机无法解析 MAC
网络通信失败
```

防护：

| 技术              | 作用               |
| ----------------- | ------------------ |
| DHCP snooping     | 生成 binding table |
| ARP inspection    | 检查 ARP           |
| IPv6 source guard | 检查 source        |
| Destination guard | 检查 destination   |

------

# 5 总结：DoS 的本质

所有 DoS 攻击其实都是：

```
耗尽资源
```

资源主要四类：

| 资源           | DoS 方法              |
| -------------- | --------------------- |
| CPU            | flood control packets |
| Memory         | table exhaustion      |
| Connection     | SYN flood             |
| Address tables | ARP / NDP flood       |

------

# 6 一句话记忆

DoS 的核心：

```
资源耗尽攻击
```

攻击者的目的不是：

```
入侵系统
```

而是：