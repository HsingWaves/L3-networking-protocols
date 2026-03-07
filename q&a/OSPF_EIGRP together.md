很好，这个问题其实是 **看 `show ip route` 的 Administrative Distance (AD)**。
我一步一步带你看这题是怎么判断的。

------

# 1️⃣ 看 routing table 的这一行

题目里：

```text
O E2 10.123.187.0/24 [110/20] via 10.1.3.2
```

解释：

| 字段 | 含义                    |
| ---- | ----------------------- |
| O    | OSPF                    |
| E2   | OSPF external type 2    |
| 110  | Administrative Distance |
| 20   | metric                  |

------

# 2️⃣ EIGRP topology 表

下面还有：

```text
P 10.123.187.0/24, 0 successors, FD is Inaccessible
```

说明：

- EIGRP **确实知道这个路由**
- 但 **没有可用 successor**
- 所以 **没有安装到 routing table**

------

# 3️⃣ routing table 只能选一个协议

Cisco 路由选择顺序：

```text
1  Longest prefix match
2  Administrative distance
3  Metric
```

这题的关键是 **AD**。

------

# 4️⃣ 各协议 AD

| 协议           | AD      |
| -------------- | ------- |
| Connected      | 0       |
| Static         | 1       |
| EIGRP internal | 90      |
| **OSPF**       | **110** |
| RIP            | 120     |
| EIGRP external | 170     |

------

# 5️⃣ 为什么说 OSPF 抢走了

因为 routing table 里：

```text
O E2 10.123.187.0/24
```

说明：

```text
EIGRP 没有被安装
OSPF 被安装
```

而题目说：

> should be learning the route via **EIGRP**

说明设计上：

```text
这条路应该来自 EIGRP
```

但现在：

```text
来自 OSPF redistribution
```

------

# 6️⃣ 从哪里还能看出来？

看 **EIGRP topology**

```text
P 10.123.187.0/24, 0 successors
```

正常应该是：

```text
1 successors
```

例如：

```text
P 10.55.13.0/24, 1 successors
```

------

# 7️⃣ 所以完整逻辑

当前情况：

```text
10.123.187.0/24
      │
      ├── EIGRP topology 有
      │      但没有 successor
      │
      └── OSPF external route 已安装
```

所以：

```text
routing table 使用 OSPF
```

而不是：

```text
EIGRP
```

------

# 8️⃣ 一眼判断口诀（ENARSI）

看到：

```text
show ip route
O E2
```

同时题目说：

```text
should be EIGRP
```

就说明：

```text
route redistribution 引入了 OSPF 路径
```

解决方案通常是：

```text
修正 redistribution
或过滤该 route
```

------

# 9️⃣ 这题最关键的两行

```text
O E2 10.123.187.0/24
```

和

```text
0 successors FD is Inaccessible
```

下面给你一张 **ENARSI 路由题秒杀表（考试版）**。
核心目标：**看到 `show ip route` / `show ip eigrp topology` / `show ip ospf database` 时 5 秒判断问题在哪。**

------

# 一、show ip route 代码速查

| 代码 | 含义                 |
| ---- | -------------------- |
| C    | Connected            |
| S    | Static               |
| R    | RIP                  |
| O    | OSPF intra-area      |
| O IA | OSPF inter-area      |
| O E1 | OSPF external type 1 |
| O E2 | OSPF external type 2 |
| D    | EIGRP internal       |
| D EX | EIGRP external       |
| B    | BGP                  |

------

# 二、Administrative Distance（必须背）

| 协议           | AD   |
| -------------- | ---- |
| Connected      | 0    |
| Static         | 1    |
| EIGRP summary  | 5    |
| EIGRP internal | 90   |
| OSPF           | 110  |
| IS-IS          | 115  |
| RIP            | 120  |
| EIGRP external | 170  |
| iBGP           | 200  |

**规则**

```text
AD 小的赢
```

------

# 三、路由选择顺序（考试常考）

```text
1 Longest Prefix Match
2 Administrative Distance
3 Metric
```

例子：

```text
10.0.0.0/8
10.1.1.0/24
```

一定选：

```text
/24
```

------

# 四、ENARSI 最常见 6 种问题

## ① Redistribution 导致错误路径

症状：

```text
show ip route
O E2 10.x.x.x
```

但题目说：

```text
should be learned via EIGRP
```

说明：

```text
OSPF redistribution 抢走了
```

解决：

```text
fix redistribution
route-map filter
```

------

## ② EIGRP 有 topology 但没有 routing entry

症状：

```text
show ip eigrp topology

0 successors
FD is Inaccessible
```

说明：

```text
EIGRP route 不可达
```

常见原因：

- next hop unreachable
- split horizon
- route filtering
- passive interface

------

## ③ OSPF 路由不出现

看：

```text
show ip ospf neighbor
```

如果没有：

```text
FULL
```

说明：

```text
OSPF adjacency 没建立
```

常见原因：

| 问题           | 命令                   |
| -------------- | ---------------------- |
| area 不一致    | show ip ospf interface |
| hello interval | show ip ospf interface |
| network type   | show ip ospf interface |

------

## ④ EIGRP neighbor 不建立

看：

```text
show ip eigrp neighbors
```

如果没有：

```text
neighbor
```

检查：

| 问题            | 命令              |
| --------------- | ----------------- |
| AS 不一致       | show run          |
| k values 不一致 | show ip protocols |
| network 不匹配  | show run          |

------

## ⑤ Route 被 ACL / distribute-list 过滤

症状：

```text
topology table 有
routing table 没有
```

检查：

```text
show ip protocols
```

看：

```text
distribute-list
prefix-list
route-map
```

------

## ⑥ Route 被更优 AD 抢走

例子：

```text
D 10.1.1.0/24
O 10.1.1.0/24
```

如果 routing table 是：

```text
D
```

说明：

```text
90 < 110
```

EIGRP 赢。

------

# 五、EIGRP topology 快速判断

看这一列：

```text
successors
```

| 情况        | 含义 |
| ----------- | ---- |
| 1 successor | 正常 |

> 1 successor | load balancing |
> 0 successor | 不可达 |

例子：

```text
P 10.1.1.0/24, 1 successors
```

正常。

------

# 六、OSPF 外部路由区别

| 类型 | 特点                     |
| ---- | ------------------------ |
| O E1 | internal cost + external |
| O E2 | 只看 external cost       |

考试套路：

```text
E1 会考虑内部路径
E2 不会
```

------

# 七、Redistribution 常见错误

看到：

```text
O E2
D EX
```

基本说明：

```text
发生了 redistribution
```

检查：

```text
show run | section router
```

例如：

```text
redistribute eigrp
redistribute ospf
redistribute static
```

------

# 八、ENARSI 秒杀流程

做题时按这个顺序看：

```text
1 看 show ip route
2 看 AD
3 看 route source (O / D / B)
4 看 topology table
5 看 redistribution
```

------

# 九、考试最经典套路

题目：

```text
Route should be learned via EIGRP
```

但你看到：

```text
O E2
```

答案几乎一定是：

```text
fix redistribution
```

------

# 十、最重要的一句口诀

```text
show ip route 先看协议
再看 AD
再看 metric
```

------

