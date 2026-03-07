你这个疑问其实是 **ACL 的匹配对象 vs SLA 的目标** 没区分清楚。

我们一步一步看。

------

# 1 先看 IP SLA 发的包

R1 配置：

```cisco
ip sla 700
 icmp-echo 10.66.66.66 source-ip 10.10.10.1
```

所以发送的 **ICMP Echo Request** 是：

| 字段           | 值              |
| -------------- | --------------- |
| Source IP      | **10.10.10.1**  |
| Destination IP | **10.66.66.66** |

也就是：

```
10.10.10.1  ----->  10.66.66.66
```

------

# 2 再看 ACL 在哪里

R6：

```
interface E0/0
 ip access-group DDOS in

interface E0/1
 ip access-group DDOS in
```

ACL 是 **入方向**。

也就是说：

R6 收到的 **incoming packet** 是：

```
src = 10.10.10.1
dst = 10.66.66.66
```

------

# 3 ACL 语法

ACL：

```
permit icmp host 10.10.10.1 host 10.66.66.66
```

语法是：

```
permit icmp <source> <destination>
```

所以：

| ACL字段          | 实际含义    |
| ---------------- | ----------- |
| host 10.10.10.1  | source      |
| host 10.66.66.66 | destination |

------

# 4 为什么 destination 是 10.66.66.66

因为 **IP SLA ping 的目标就是 10.66.66.66**。

```
icmp-echo 10.66.66.66
```

所以 ACL 必须允许：

```
10.10.10.1 → 10.66.66.66
```

------

# 5 为什么 D 是错的

D：

```
permit icmp host 10.66.66.66 host 10.10.10.1
```

意思是：

```
src = 10.66.66.66
dst = 10.10.10.1
```

但这是 **Echo Reply** 的方向：

```
10.66.66.66 → 10.10.10.1
```

而 ACL 是 **inbound on R6**，不会看到 reply。

------

# 6 一张图秒懂

```
R1                    R6                  Server
10.10.10.1 ----------------------> 10.66.66.66
      ICMP Echo Request

ACL on R6 inbound sees:

src = 10.10.10.1
dst = 10.66.66.66
```

所以 ACL 必须写：

```
permit icmp host 10.10.10.1 host 10.66.66.66
```

------

看到 ACL 时先问自己：

**ACL 是在哪个接口、哪个方向。**

口诀：

```
IN ACL = 看进入接口的包
OUT ACL = 看离开接口的包
```

------

我们把 **ICMP（Internet Control Message Protocol）**从 CCNP / 网络工程角度完整展开一下：
包括 **功能、机制、常见类型、与 ACL 的结合、考试常见点**。

------

# 一、ICMP 是什么

**ICMP = Internet Control Message Protocol**

作用：

> **IP 网络的错误报告 + 网络诊断协议**

它不是传输数据的协议，而是 **网络控制信息协议**。

特点：

| 特点     | 说明                     |
| -------- | ------------------------ |
| 工作层   | **Layer 3（IP 协议族）** |
| 封装     | 在 **IP packet** 里面    |
| 没有端口 | 不是 TCP/UDP             |
| 用途     | 错误通知 + 诊断          |

IP header：

```
IP Header
   |
   +---- ICMP Header
           |
           +---- ICMP Data
```

------

# 二、ICMP 的核心功能

ICMP有两个主要用途：

### 1 网络错误报告

例如：

- 目标不可达
- TTL 超时
- 分片问题

典型例子：

```
Destination Unreachable
```

例如：

```
PC -> Router -> Router -> Server

如果 Server 不存在
Router 会返回：

ICMP Destination Unreachable
```

------

### 2 网络诊断

两个最经典工具：

| 工具       | 使用 ICMP         |
| ---------- | ----------------- |
| ping       | ICMP echo         |
| traceroute | ICMP TTL exceeded |

------

# 三、ICMP 常见类型

ICMP通过 **Type / Code** 区分。

最重要的几个：

| Type | 名称                    | 用途       |
| ---- | ----------------------- | ---------- |
| 0    | Echo Reply              | ping 回复  |
| 8    | Echo Request            | ping 请求  |
| 3    | Destination Unreachable | 目标不可达 |
| 11   | Time Exceeded           | TTL 超时   |
| 5    | Redirect                | 路由重定向 |

------

## 1 ping

```
ICMP Echo Request (Type 8)
ICMP Echo Reply   (Type 0)
```

流程：

```
Host A ---- echo request ----> Host B
Host A <--- echo reply  ------ Host B
```

------

## 2 traceroute

TTL 机制：

```
TTL=1 -> Router1 -> TTL=0 -> ICMP Time Exceeded
TTL=2 -> Router2 -> TTL=0 -> ICMP Time Exceeded
```

每个 router 返回：

```
ICMP Time Exceeded (Type 11)
```

------

## 3 Destination Unreachable

常见 Code：

| Code | 含义                |
| ---- | ------------------- |
| 0    | network unreachable |
| 1    | host unreachable    |
| 3    | port unreachable    |

例子：

```
UDP 访问不存在端口
→ 返回 ICMP port unreachable
```

------

# 四、ICMP 工作机制

ICMP **不会主动发送**。

只有 **发生网络事件** 才发送。

例子：

```
IP packet TTL=0
→ Router 触发 ICMP Time Exceeded
```

或：

```
ping
→ 主机主动发送 ICMP Echo Request
```

------

# 五、ICMP 和 ACL

ACL 可以过滤 ICMP。

语法：

```
permit icmp source destination
```

或：

```
permit icmp any any echo
permit icmp any any echo-reply
```

------

## ICMP ACL 语法

基本：

```
permit icmp any any
```

或：

```
permit icmp host A host B
```

------

### 指定类型

Cisco 支持：

```
echo
echo-reply
time-exceeded
unreachable
```

例子：

允许 ping：

```
permit icmp any any echo
permit icmp any any echo-reply
```

------

# 六、ICMP + ACL 实际例子

### 1 只允许 ping

```
ip access-list extended PING_ONLY

permit icmp any any echo
permit icmp any any echo-reply
deny ip any any
```

------

### 2 禁止 ping

```
ip access-list extended BLOCK_PING

deny icmp any any echo
permit ip any any
```

------

### 3 只允许监控服务器 ping

```
permit icmp host 10.10.10.1 any echo
```

------

# 七、ICMP + CoPP（控制平面保护）

很多设备会限制 ICMP。

例如：

```
class-map ICMP
 match protocol icmp
```

限制 CPU：

```
police 1000
```

避免 ICMP flood。

------

# 八、ICMP 在安全中的风险

攻击：

### 1 ping flood

```
大量 ICMP echo
```

DoS。

------

### 2 ICMP redirect attack

Router发送错误路由信息。

------

### 3 ICMP tunneling

利用 ICMP 传输数据。

------

# 九、考试最常见 ICMP 场景

### 1 IP SLA

```
icmp-echo
```

------

### 2 ACL blocking ping

```
deny icmp any any
```

------

### 3 traceroute 故障

如果 ICMP blocked：

```
traceroute 失败
```

------

### 4 Path MTU Discovery

PMTUD 使用：

```
ICMP Fragmentation needed
```

------

# 十、ICMP vs TCP/UDP

| 特征       | ICMP | TCP  | UDP  |
| ---------- | ---- | ---- | ---- |
| 端口       | 无   | 有   | 有   |
| 功能       | 控制 | 传输 | 传输 |
| ping       | ✔    | ❌    | ❌    |
| traceroute | ✔    | ❌    | ❌    |

------

# 十一、CCNP 考试一个经典陷阱

ACL：

```
deny icmp any any
permit ip any any
```

影响：

| 工具       | 结果 |
| ---------- | ---- |
| ping       | 失败 |
| traceroute | 失败 |
| TCP        | 正常 |
| HTTP       | 正常 |

因为：

```
ICMP 被 block
IP 仍然允许
```

------

# 十二、一句话总结

**ICMP 是 IP 网络的“错误通知 + 网络诊断协议”。**

主要功能：

```
ping
traceroute
错误报告
路径 MTU
```

ACL 可以：

```
允许 / 禁止 ICMP
限制 ping
保护设备
```

------

