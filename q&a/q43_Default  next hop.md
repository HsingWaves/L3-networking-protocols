| 命令                    | 什么时候用           |
| ----------------------- | -------------------- |
| set ip next-hop         | 强制改下一跳         |
| set ip default next-hop | 只有无匹配路由时用 ✅ |
| verify-availability     | 配 SLA               |
| recursive               | 递归解析             |

数据包进入接口
        ↓
是否配置了 ip policy route-map？
        ↓
        YES
        ↓
匹配 route-map 条件？
        ↓
        YES
        ↓
执行 set ip next-hop
        ↓
转发 (结束)

​      NO

​        ↓

进入正常路由查找

1️⃣ PBR (Policy Based Routing)
2️⃣ 正常路由表查找 (Longest Match)
3️⃣ 默认路由 0.0.0.0/0
4️⃣ 丢弃

## 1️⃣ PBR（最优先）

特点：

- 不看 routing table
- 可以基于：
  - source IP
  - ACL
  - DSCP
  - 长度
- 强制指定下一跳

例子：

```
route-map PBR permit 10
 match ip address 101
 set ip next-hop 10.1.1.1
```

👉 即使 routing table 说走 10.2.2.2
 PBR 也会强制改走 10.1.1.1

⚠️ 这就是为什么 PBR 很危险。



## 2️⃣ 静态路由 / 动态路由

这是正常转发表。

核心规则：

```
Longest Prefix Match
```

优先级：

1. /32
2. /30
3. /24
4. /16
5. /8
6. 0.0.0.0/0

与是否静态/OSPF/BGP无关。

------

## 3️⃣ 默认路由

```
ip route 0.0.0.0 0.0.0.0 10.1.1.1
```

只有在：

> 没有更具体匹配时才使用

------

# 🔥 四、经典考试陷阱

------

### ⚠️ 陷阱 1

PBR 会不会被默认路由覆盖？

❌ 不会
 PBR 在最前面执行

------

### ⚠️ 陷阱 2

PBR set ip next-hop 如果 next-hop 不可达？

默认行为：

- 直接丢弃
- 不会回退到 routing table

除非用：

```
set ip default next-hop
```

------

### ⚠️ 陷阱 3

多个路由同时存在怎么办？

看：

```
最长匹配
```

不是看 AD（管理距离）

                数据包进入接口
                         ↓
        是否配置 ip policy route-map？
                         ↓
              ┌──────────┴──────────┐
              │                     │
             YES                    NO
              │                     │
       是否匹配 route-map？         ↓
              │              查 routing table
              │                     ↓
        ┌─────┴─────┐        有最长匹配？
        │           │              │
       YES          NO             │
        │           │              │
set ip next-hop     ↓              │
        │       进入 routing table  │
        ↓                          │
      转发                         ↓
                             有默认路由？
                                    │
                           ┌────────┴────────┐
                           │                 │
                          YES               NO
                           │                 │
                       默认路由            丢弃

# 结合真实网络场景

在企业里：

- PBR 用于：
  - 分流
  - 走不同 ISP
  - 引流到防火墙
- 默认路由：
  - 出口
- 静态路由：
  - 特定子网

PBR 优先于 routing table
routing table 优先于 default
default 优先于 drop

好，我们把 **Policy-Based Routing (PBR) + route-map** 从 0 讲清楚。
你学过 BGP route-map，现在把它放到“转发层面”理解一下。

------

# 🔥 一、什么是 Policy-Based Routing？

正常路由转发是：

```text
看目的地址 → 查 routing table → 最长匹配 → 转发
```

PBR 是：

```text
不按目的地址
而按“策略”决定走哪
```

比如：

- 192.168.1.0 的用户走 ISP1
- 192.168.2.0 的用户走 ISP2
- 某些流量强制走防火墙

这就是 PBR。

------

# 🔥 二、核心组件

PBR =

```text
ACL + route-map + 接口绑定
```

------

# 🔥 三、工作流程图

```text
流量进入接口
        ↓
接口是否有 ip policy route-map？
        ↓
        YES
        ↓
route-map 逐条匹配
        ↓
匹配成功？
        ↓
        YES → 执行 set 动作
        NO  → 正常 routing table
```

⚠️ PBR 只对“进入接口的流量”生效

------

# 🔥 四、基本配置结构

------

## ① 定义 ACL（匹配流量）

```bash
access-list 101 permit ip 192.168.1.0 0.0.0.255 any
```

------

## ② 定义 route-map

```bash
route-map PBR permit 10
 match ip address 101
 set ip next-hop 10.1.1.1
```

意思：

> 如果匹配 ACL 101，就把流量送到 10.1.1.1

------

## ③ 绑定到接口

```bash
interface Gi0/0
 ip policy route-map PBR
```

------

# 🔥 五、关键 set 命令区别

| 命令                                | 作用           |
| ----------------------------------- | -------------- |
| set ip next-hop                     | 强制改下一跳   |
| set ip default next-hop             | 无匹配路由时用 |
| set interface                       | 指定出口接口   |
| set ip next-hop verify-availability | 配 SLA         |

------

# 🔥 六、经典考试陷阱

------

### ⚠️ 1. PBR 在哪里生效？

只在：

```text
入方向接口
```

不是出方向。

------

### ⚠️ 2. PBR 优先级

```text
PBR > routing table > default route
```

------

### ⚠️ 3. next-hop 不可达怎么办？

默认：

- 丢弃
- 不回退到 routing table

除非用：

```bash
set ip default next-hop
```

------

# 🔥 七、一个典型双 ISP 场景

```text
            ISP1
              |
          10.1.1.1
              |
LAN -- R1
              |
          20.1.1.1
              |
            ISP2
```

目标：

- 192.168.1.0 走 ISP1
- 192.168.2.0 走 ISP2

用两个 ACL + route-map sequence 即可实现。

------

# 🔥 八、和 BGP route-map 区别

| 用途         | PBR route-map | BGP route-map |
| ------------ | ------------- | ------------- |
| 作用层       | 转发层        | 控制平面      |
| 改什么       | 数据流向      | 属性          |
| 是否查路由表 | 不查          | 查            |

------

# 🔥 九、一句话记忆

```text
PBR = 用策略决定下一跳，而不是目的地址
```

------

