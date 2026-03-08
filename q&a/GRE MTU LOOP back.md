这题关键只看一句话：

> **using loopback to hide internal networks**

意思是：

👉 **GRE tunnel 的 source / destination 要用 loopback 地址**

------

# 一、GRE tunnel 基本结构

GRE 配置核心只有两个：

```
tunnel source
tunnel destination
```

例如：

```
interface Tunnel1
 ip address 172.20.1.2 255.255.255.0
 tunnel source 10.10.2.2
 tunnel destination 10.10.1.1
```

其中：

| 项                 | 作用        |
| ------------------ | ----------- |
| tunnel source      | 本地接口 IP |
| tunnel destination | 对端接口 IP |

------

# 二、题目要求

题目明确说：

```
using loopback
```

所以：

```
tunnel source = loopback address
tunnel destination = loopback address
```

通常 loopback 用：

```
10.x.x.x
```

而不是：

```
192.168.x.x
```

（很多题里 192.168 是 LAN）

------

# 三、逐个选项看

## A

```
tunnel source 192.168.20.1
tunnel destination 192.168.10.1
```

192.168 通常是 **内部 LAN**

❌ 不符合 loopback

------

## B

```
tunnel source 10.10.2.2
tunnel destination 10.10.1.1
ip mtu 1400
ip tcp adjust-mss 1360
```

✔ 使用 **loopback**

并且：

```
GRE header = 24 bytes
```

所以通常要：

```
MTU 1400
MSS 1360
```

这是 Cisco 推荐配置。

✔ 正确

------

## C

```
ip mtu 1500
```

GRE tunnel 如果 MTU 1500：

```
封装后会超过 1500
```

容易 fragmentation。

❌ 不推荐

------

## D

```
source 192.168.20.1
destination 10.10.1.1
```

source 不是 loopback。

❌ 错

------

# 四、正确答案

✅ **B**

------

# 五、GRE 题考试秒杀规则

看到 GRE：

先看三件事：

### 1 tunnel source / destination

```
必须是可达 IP
```

如果题目说：

```
use loopback
```

那就选：

```
10.x.x.x
```

------

### 2 GRE MTU

GRE 头部：

```
24 bytes
```

所以推荐：

```
ip mtu 1400
```

------

### 3 TCP MSS

```
ip tcp adjust-mss 1360
```

防止 fragmentation。

------

# 六、ENARSI GRE 秒杀口诀

```
GRE = 1400 / 1360
ip mtu 1400
ip tcp adjust-mss 1360
```

在网络里 **Loopback（回环接口）** 是一个非常重要的概念，理解它可以从 **三个层次**来看。

------

# 一、最简单理解

**Loopback = 永远不会 down 的虚拟接口**

例如：

```cisco
interface Loopback0
 ip address 10.10.1.1 255.255.255.255
```

特点：

- 不是物理接口
- 不连接任何网线
- 只存在于路由器内部

所以：

```
只要设备开着
Loopback 就是 up
```

------

# 二、为什么叫 Loopback

**loopback = 回环**

意思是：

```
数据从自己出来
又回到自己
```

最典型例子：

```
127.0.0.1
```

这是电脑的 **localhost**

例如：

```
ping 127.0.0.1
```

数据：

```
自己 → 自己
```

------

# 三、为什么网络工程师很喜欢用 Loopback

因为它 **稳定**。

对比：

| 接口            | 是否可能 down |
| --------------- | ------------- |
| GigabitEthernet | 可能          |
| Serial          | 可能          |
| Tunnel          | 可能          |
| Loopback        | 基本不会      |

所以很多协议喜欢用 loopback。

------

# 四、Loopback 常见用途

## 1 Router ID

OSPF / BGP：

```
router-id = loopback
```

例如：

```
router ospf 1
 router-id 1.1.1.1
```

或者自动选：

```
最高 loopback IP
```

原因：

```
loopback 永远 up
```

------

## 2 GRE / Tunnel endpoint

例如：

```
R1 ---- ISP ---- R2
```

如果用物理接口：

```
tunnel source g0/0
```

问题：

```
接口 down
tunnel 也 down
```

所以常用：

```
tunnel source loopback0
```

例如：

```
R1 loopback0 10.10.1.1
R2 loopback0 10.10.2.2
tunnel source 10.10.1.1
tunnel destination 10.10.2.2
```

只要网络能到达：

```
tunnel 就稳定
```

------

## 3 BGP 邻居

BGP 很常用：

```
neighbor 2.2.2.2 update-source loopback0
```

好处：

```
链路换了也不影响
```

------

# 五、一个直观图

### 没用 loopback

```
R1 g0/0 ---- ISP ---- g0/0 R2
       GRE Tunnel
```

如果：

```
g0/0 down
```

GRE 就挂。

------

### 用 loopback

```
R1 Lo0(10.10.1.1)
      |
      g0/0 ---- ISP ---- g0/0
                             |
                       Lo0(10.10.2.2)
```

GRE：

```
tunnel source 10.10.1.1
tunnel destination 10.10.2.2
```

只要 ISP 能到：

```
Tunnel 永远稳定
```

------

# 六、Cisco 考试一句话总结

```
Loopback = stable logical interface used as router identity
```

或者记一句更简单的：

```
Loopback = 永远不会 down 的接口
```

------

