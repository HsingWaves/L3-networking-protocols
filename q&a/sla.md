**SLA = Service Level Agreement**

中文：

**服务等级协议**

意思是：

> 对网络服务质量的一个约定或保证

例如 ISP 可能承诺：

| 指标   | SLA          |
| ------ | ------------ |
| 可用性 | 99.9% uptime |
| 延迟   | < 50 ms      |
| 丢包   | < 1%         |
| 抖动   | < 30 ms      |

这些就是 **SLA 指标**。

------

# 2️⃣ Cisco IP SLA 是什么

Cisco 做了一个工具：

**IP SLA**

用来：

> 主动测量网络质量

路由器会 **主动发测试流量** 来测量：

| 测量指标   | 方法        |
| ---------- | ----------- |
| 延迟       | ping        |
| 服务可用性 | tcp connect |
| jitter     | udp jitter  |
| 丢包       | udp test    |

所以：

**IP SLA = 网络质量测试工具**

------

# 3️⃣ IP SLA 工作原理

普通网络流量

```
Host ---- Router ---- Server
```

IP SLA

```
Router --- probe ---> Server
```

Router 会主动发送 **probe**

例如：

```
TCP SYN → server:80
```

如果 server 回复

```
SYN-ACK
```

说明：

```
HTTP服务正常
```

------

# 4️⃣ 一个真实例子（最常见）

企业有两个 ISP

```
        ISP1
          |
        Router
          |
        ISP2
```

希望：

```
ISP1 不通 → 自动切 ISP2
```

做法：

Router 每 5 秒 ping Google

```
ip sla 1
 icmp-echo 8.8.8.8
 frequency 5
```

启动：

```
ip sla schedule 1 life forever start-time now
```

绑定 track

```
track 1 ip sla 1 reachability
```

静态路由：

```
ip route 0.0.0.0 0.0.0.0 ISP1 track 1
ip route 0.0.0.0 0.0.0.0 ISP2 200
```

如果：

```
ping失败
```

Router 会自动：

```
切换 ISP
```

------

# 5️⃣ IP SLA 常见用途

| 用途         | 例子       |
| ------------ | ---------- |
| ISP failover | 双线路     |
| 服务检测     | HTTP / DNS |
| QoS监控      | jitter     |
| VoIP质量检测 | UDP jitter |
| 路由控制     | 配合 track |

------

# 6️⃣ IP SLA 常见类型

| 类型        | 作用       |
| ----------- | ---------- |
| icmp-echo   | ping       |
| tcp-connect | 检查端口   |
| udp-jitter  | 测语音质量 |
| http        | 测 web     |

------

# 7️⃣ 为什么考试老考 IP SLA

因为它能和很多技术结合：

| 技术         | 用途             |
| ------------ | ---------------- |
| Static route | failover         |
| PBR          | policy routing   |
| HSRP         | gateway failover |
| BGP          | route tracking   |

所以在 **ENARSI 300-410** 经常出。

------

# 8️⃣ 一句话理解 IP SLA

最简单理解：

> **IP SLA = 路由器主动做网络健康检查**

就像：

```
路由器每隔几秒 ping 一下网络
```

如果不通：

```
自动触发策略
```

------

# 1️⃣ IP SLA 最常见 4 种 Probe

| 类型            | 用途               | 常见端口          | 是否需要 responder |
| --------------- | ------------------ | ----------------- | ------------------ |
| **icmp-echo**   | 测连通性 (ping)    | ICMP              | ❌ 不需要           |
| **tcp-connect** | 测服务端口是否开放 | TCP (80/443/22等) | ❌ 不需要           |
| **udp-jitter**  | 测延迟/抖动/丢包   | UDP               | ✅ 需要             |
| **http**        | 测 HTTP server     | TCP 80            | ❌ 不需要           |

例子：

### ICMP

```cisco
ip sla 1
 icmp-echo 8.8.8.8
```

### TCP

```cisco
ip sla 2
 tcp connect 10.1.1.1 80
```

### UDP Jitter

```cisco
ip sla 3
 udp-jitter 10.1.1.1 5000
```

------

# 2️⃣ Responder 什么时候需要

只有这些需要：

```
udp-jitter
voip
path-jitter
```

配置：

Server端：

```cisco
ip sla responder
```

------

# 3️⃣ Schedule 命令结构

```cisco
ip sla schedule <ID> life <time> start-time now
```

常见：

### 永久运行

```cisco
ip sla schedule 10 life forever start-time now
```

### 运行30秒

```cisco
ip sla schedule 10 life 30 start-time now
```

------

# 4️⃣ non SLA host 必考点

如果题目写：

```
non SLA host
```

意味着：

**对端没有 responder**

这时必须：

```
control disable
```

例子

```cisco
ip sla 10
 tcp connect 10.1.1.1 80 control disable
```

------

# 5️⃣ IP SLA + Tracking (最常考)

典型场景：

**ISP failover**

```
Primary ISP
Backup ISP
```

配置

```cisco
ip sla 1
 icmp-echo 8.8.8.8
 frequency 5

ip sla schedule 1 life forever start-time now

track 1 ip sla 1 reachability
```

静态路由：

```cisco
ip route 0.0.0.0 0.0.0.0 ISP1 track 1
ip route 0.0.0.0 0.0.0.0 ISP2 200
```

------

# 6️⃣ ENARSI 做题秒杀口诀

```
ping → icmp-echo
端口 → tcp-connect
jitter → udp-jitter
non-SLA host → control disable
failover → track
```

------

