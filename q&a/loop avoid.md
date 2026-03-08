# ENARSI Routing Loop Prevention Cheat Sheet

| 技术                   | 作用                 | 工作方式                             | 常见协议    |
| ---------------------- | -------------------- | ------------------------------------ | ----------- |
| **Split Horizon**      | 防环路               | 不把从某接口学到的路由再从该接口发送 | RIP / EIGRP |
| **Poison Reverse**     | 防环路 + 快收敛      | 把路由发回去但 metric=∞              | RIP         |
| **Route Poisoning**    | 快速通知网络路由失效 | 把失效路由 metric 设为∞              | RIP / EIGRP |
| **Triggered Update**   | 加速收敛             | 路由变化立即发送更新                 | RIP         |
| **Hold-down Timer**    | 防止错误更新         | 暂时忽略坏更新                       | RIP         |
| **Feasible Successor** | 快速切换备份路径     | 提前计算备份路由                     | EIGRP       |
| **DUAL Algorithm**     | 无环路路径计算       | 保证 loop-free                       | EIGRP       |
| **Max Hop Count**      | 防止无限循环         | RIP 最大 hop=15                      | RIP         |

------

# 

### Split Horizon

```
从某接口学到 → 不从该接口发回
```

### Poison Reverse

```
发回去，但 metric = infinity
```

------

# EIGRP 高频坑（ENARSI必考）

如果看到：

```
EIGRP
Frame Relay
multipoint
hub-and-spoke
spokes cannot reach each other
```

答案 99% 是：

```
no ip split-horizon eigrp AS
```

原因：

```
split horizon 阻止 hub 把 A 的路由发给 B
```

------

# RIP Count-to-Infinity 解决机制

RIP 防环主要靠：

```
Split Horizon
Poison Reverse
Route Poisoning
Hold-down Timer
Max Hop Count
```

------

# 一句话记忆

```
Split Horizon  → 不发
Poison Reverse → 发∞
Route Poisoning → 宣布死亡
Triggered Update → 立刻广播
Hold-down → 暂停相信别人
```

很好 👍 我给你做一张 **ENARSI 路由协议核心秒杀图（很多人考试前都会背这一页）**。
主要覆盖 **EIGRP + OSPF + RIP 高频考点**。

------

# 1️⃣ EIGRP 秒杀表（ENARSI重点）

| 机制               | 作用       | 关键点                |
| ------------------ | ---------- | --------------------- |
| DUAL               | 无环路算法 | 保证 loop-free        |
| Successor          | 主路径     | 最优路径              |
| Feasible Successor | 备用路径   | 满足 **FD < AD**      |
| Split Horizon      | 防环路     | multipoint 时可能要关 |
| Stub Router        | 减少查询   | hub-spoke 网络常用    |
| Metric             | 复合度量   | BW + Delay            |

### Feasible Condition（考试必考）

```text
AD < FD
```

意思：

```text
邻居到目标网络的距离 < 当前路径总距离
```

------

# 2️⃣ OSPF 秒杀表

| 概念      | 说明                     |
| --------- | ------------------------ |
| Cost      | metric = 100M / BW       |
| DR / BDR  | broadcast 网络选举       |
| Router ID | OSPF 唯一标识            |
| LSA       | Link State Advertisement |
| SPF       | Dijkstra 算法            |

### DR 选举优先级

```text
1. OSPF Priority
2. Router ID
```

------

# 3️⃣ OSPF LSA 类型（考试非常喜欢问）

| LSA    | 名称          | 区域    |
| ------ | ------------- | ------- |
| Type 1 | Router LSA    | area 内 |
| Type 2 | Network LSA   | DR 产生 |
| Type 3 | Summary LSA   | ABR     |
| Type 4 | ASBR Summary  | ABR     |
| Type 5 | External      | ASBR    |
| Type 7 | NSSA External | NSSA    |

记忆口诀：

```text
1 Router
2 Network
3 Summary
4 ASBR
5 External
7 NSSA
```

------

# 4️⃣ RIP 秒杀表

| 特点    | 值          |
| ------- | ----------- |
| Metric  | Hop count   |
| Max hop | 15          |
| 16      | unreachable |
| Update  | 30 秒       |

### RIP 防环机制

```
Split Horizon
Poison Reverse
Route Poisoning
Hold-down
Triggered Update
```

------

# 5️⃣ ENARSI 最容易考的 5 个 routing 陷阱

### ① multipoint EIGRP 不通

解决：

```
no ip split-horizon eigrp AS
```

------

### ② OSPF 邻居起不来

检查：

```
area
hello timer
network type
authentication
```

------

### ③ EIGRP 邻居起不来

检查：

```
AS number
K values
network
authentication
```

------

### ④ OSPF DR 不正确

修改：

```
ip ospf priority
```

------

### ⑤ 路由不被 redistribution

使用：

```
route-map
prefix-list
```

------

# 6️⃣ 一张记忆图（最重要）

```
Distance Vector
    │
    ├─ RIP
    │    hop metric
    │    split horizon
    │
    └─ EIGRP
         DUAL
         feasible successor
         composite metric

Link State
    │
    └─ OSPF
         LSA
         SPF
         DR/BDR
```

------

| 模式                    | 特点               | 是否需要关 split horizon | 是否需要 neighbor |
| ----------------------- | ------------------ | ------------------------ | ----------------- |
| **multipoint**          | 一个接口多个 PVC   | 常需要                   | 不需要            |
| **point-to-point**      | 一个子接口一个 PVC | 不需要                   | 不需要            |
| **NBMA**                | 不支持广播         | 不需要                   | 需要              |
| **point-to-multipoint** | 类似 p2p 逻辑      | 不需要                   | 不需要            |



Hub
 │
 ├── multipoint  ← 可能需要关 split horizon
 │
 ├── point-to-point  ← 最简单
 │
 ├── NBMA  ← 需要 neighbor
 │
 └── point-to-multipoint

在 **ENARSI 里是两个不同层面的东西**，很多人会混在一起：

1️⃣ **接口/网络类型（topology / network type）**
 2️⃣ **距离向量协议的防环机制（loop prevention）**

它们是 **不同维度**。

------

# 一、接口 / 网络类型（Topology）

这些决定 **邻居怎么建立、广播怎么传播**。

| 类型                | 说明             | 典型协议          |
| ------------------- | ---------------- | ----------------- |
| point-to-point      | 两个节点         | OSPF / EIGRP      |
| multipoint          | 一个接口多个邻居 | Frame Relay       |
| point-to-multipoint | 逻辑多个 p2p     | OSPF              |
| NBMA                | 不支持广播       | Frame Relay / ATM |
| broadcast           | 支持广播         | Ethernet          |

这些影响：

- 邻居发现
- DR/BDR
- split horizon 是否产生问题

例如：

```
multipoint hub-spoke
```

可能出现

```
split horizon 阻止 spoke-spoke
```

------

# 二、距离向量协议的防环机制

这是 **RIP / EIGRP 的算法机制**。

| 机制             | 作用              |
| ---------------- | ----------------- |
| Split Horizon    | 不发回同接口      |
| Poison Reverse   | 发回但 metric = ∞ |
| Route Poisoning  | 宣布路由死亡      |
| Hold-down Timer  | 暂停相信更新      |
| Triggered Update | 立即更新          |

这些机制都是为了：

```
防止 routing loop
```

------

# 三、关系图（核心理解）

```
Network topology
        │
        │影响
        ▼
Interface behavior
        │
        │可能触发
        ▼
Loop prevention mechanisms
```

例如：

```
Frame Relay multipoint
        │
        ▼
Split horizon 阻止路由传播
        │
        ▼
spoke-spoke 不通
```

解决：

```
no ip split-horizon eigrp
```

------

# 四、Poison Reverse 在这里的关系

Poison reverse **不解决 multipoint 问题**。

原因：

```
Split horizon = 不发送
Poison reverse = 发送∞
```

而 multipoint 问题是：

```
hub 根本不会发
```

所以 poison reverse **不会触发**。

------

# 五、ENARSI 

### 如果题目出现

```
EIGRP
multipoint
hub-spoke
spokes cannot communicate
```

答案几乎一定是

```
no ip split-horizon eigrp
```

------

# 六、最容易混淆的一点

很多人误以为：

```
Poison reverse 是 split horizon 的反义
```

其实不是。

关系更像：

```
Split horizon
      │
      ├── 不发
      │
      └── Poison reverse
           发∞
```

这个 Cisco 文档讲的是 **EIGRP ↔ 其他协议双向重分发（mutual redistribution）时如何避免 routing loop**。我给你把它拆成考试和实际网络最重要的几个点。

------

# 1 为什么会出现 Routing Loop

当两个协议 **互相重分发** 时就会出现 **route feedback**。

例子：

```
RIP → EIGRP → RIP
```

流程：

```
RIP 网络 192.168.1.0
      ↓
R2 redistribute → EIGRP
      ↓
EIGRP domain
      ↓
R5 redistribute → RIP
      ↓
RIP 又学到这条路由
```

结果：

```
RIP → EIGRP → RIP → EIGRP
无限循环
```

Cisco 文档称为：

```
route feedback
```

------

# 2 Cisco 官方推荐的解决方案

文档给了 **3种方法**。

------

# 方法1 Route Tag（最推荐）

这是 Cisco **最推荐**的方法。

流程：

```
Redistribute 时给 route 打 tag
再根据 tag 过滤
```

例子：

### R2

```
redistribute rip route-map RIP-TO-EIGRP
route-map RIP-TO-EIGRP permit 10
 set tag 100
```

------

### R5

当 redistributing back：

```
route-map EIGRP-TO-RIP deny 10
 match tag 100

route-map EIGRP-TO-RIP permit 20
```

意思：

```
如果 route tag = 100
说明来自 RIP
不允许再回到 RIP
```

------

# 方法2 distribute-list（考试常见）

用 ACL 过滤。

例如：

```
access-list 1 deny 192.168.1.0
access-list 1 permit any
```

然后：

```
router eigrp 10
 distribute-list 1 in s1
```

意思：

```
阻止 192.168.1.0 从 EIGRP 再进入 ASBR
```

这就是你刚那题用的方法。

------

# 方法3 route-map + prefix-list

更精细的过滤。

例如：

```
ip prefix-list BLOCK seq 5 deny 192.168.1.0/24
ip prefix-list BLOCK seq 10 permit 0.0.0.0/0 le 32
```

然后：

```
route-map FILTER permit 10
 match ip address prefix-list BLOCK
```

------

# 3 Cisco 文档里的核心原则

Cisco 原文思想其实一句话：

```
Never allow a redistributed route
to be redistributed back
into its original protocol
```

翻译：

```
不要让一个被重分发的路由
再回到原来的协议
```

------

# 4 网络结构理解图

```
        EIGRP
   R3 -------- R4
    |            |
    |            |
   R2 -------- R5
        RIP
```

R2、R5 是：

```
ASBR
```

他们同时运行：

```
EIGRP + RIP
```

所以必须：

```
过滤回流路由
```

------

# 5 ENARSI考试记忆口诀

看到：

```
mutual redistribution
```

先想：

```
routing loop
```

解决方法：

优先级：

```
1 Route Tag ⭐⭐⭐⭐⭐
2 distribute-list ⭐⭐⭐
3 prefix-list / route-map ⭐⭐⭐
```

------

# 6 为什么 Cisco 推荐 Route Tag

因为 prefix / ACL 有缺陷：

如果网络变化：

```
192.168.1.0
192.168.2.0
192.168.3.0
```

ACL 会很复杂。

但 **tag 不会变化**。

------

# 7 真实网络工程做法

工程里通常：

```
redistribute → set tag
route-map permit 10
 set tag 100
```

然后：

```
match tag 100
deny
```

------

# 8 一句话总结

**双向重分发防环**

```
标记（tag）来源
再根据 tag 阻止回流
```

------

很好，这张图是 **CCNP ENARSI 最核心的一张 —— Redistribution Loop 秒杀图**。
理解这一张，**RIP / EIGRP / OSPF 双向重分发题基本秒杀**。

------

# 🔁 Redistribution Loop 秒杀图

```
           EIGRP domain
        ┌───────────────┐
        │               │
       R3──────────────R4
        │               │
        │               │
       R2──────────────R5
        └───────────────┘
             RIP domain
```

关键角色：

```
R2 和 R5 = ASBR
(Autonomous System Boundary Router)
```

它们同时运行：

```
RIP + EIGRP
```

并且：

```
mutual redistribution
```

------

# ⚠️ Routing Loop 如何产生

假设：

```
192.168.1.0/24
originated in RIP
```

流程：

```
Step1
R1 → RIP → R2

Step2
R2 redistribute RIP → EIGRP

Step3
EIGRP domain learns route

Step4
R5 redistribute EIGRP → RIP

Step5
RIP learns route again
```

结果：

```
RIP → EIGRP → RIP → EIGRP
```

Cisco 叫：

```
Route Feedback
```

------

# 🧠 Cisco官方原则

Cisco 文档核心一句话：

```
Never allow a redistributed route
to be redistributed back
into its original protocol
```

翻译：

```
不要让一个路由
回到它原来的协议
```

------

# 🛠️ 三种解决方案

## ⭐ 方法1 Route Tag（最标准）

Redistribute 时 **打标签**

```
route-map RIP-TO-EIGRP permit 10
 set tag 100
```

当回流时：

```
route-map EIGRP-TO-RIP deny 10
 match tag 100
```

意思：

```
如果是 RIP 来的
不允许回 RIP
```

------

## ⭐ 方法2 distribute-list（考试常见）

过滤 prefix。

```
access-list 1 deny 192.168.1.0
access-list 1 permit any
```

然后：

```
router eigrp 10
 distribute-list 1 in s1
```

意思：

```
阻止 192.168.1.0
从 EIGRP 再进入 ASBR
```

------

## ⭐ 方法3 prefix-list + route-map

```
ip prefix-list BLOCK seq 5 deny 192.168.1.0/24
ip prefix-list BLOCK seq 10 permit 0.0.0.0/0 le 32
route-map FILTER permit 10
 match ip address prefix-list BLOCK
```

------

# 🚨 ENARSI考试秒杀思路

看到题目：

```
Mutual Redistribution
```

脑子立刻想：

```
Routing Loop
```

然后找答案：

```
tag
distribute-list
prefix-list
```

------

# 🧩 这题 (#235) 的关键

网络：

```
RIP
R1
 ↓
R2 —— EIGRP —— R5
 ↓               ↓
R3               R4
```

问题：

```
192.168.1.0 originated in RIP
```

必须：

```
阻止它从 EIGRP 再进入 RIP
```

所以：

```
distribute-list deny 192.168.1.0
```

------

# 🧠 记忆口诀

```
双向重分发三件事

1 标记来源 (tag)
2 阻止回流 (filter)
3 ASBR 上做
```

------

# ⭐ ENARSI 真正秒杀口诀

看到：

```
RIP ↔ EIGRP
OSPF ↔ EIGRP
OSPF ↔ BGP
```

第一反应：

```
Loop prevention
```

找：

```
tag
distribute-list
prefix-list
```

------

