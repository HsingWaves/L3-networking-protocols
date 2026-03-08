这题其实是 **OSPF External Type1 vs Type2 的经典考点**。
关键句是：

> **external metric is fixed and does not add at each hop**

这句话直接说明：

**现在是 OSPF E2 路由。**

------

# 1 OSPF 外部路由两种类型

| 类型   | 计算方式                            |
| ------ | ----------------------------------- |
| **E1** | metric + OSPF path cost（每跳增加） |
| **E2** | metric 固定，不随 hop 增加          |

------

# 2 题目要求

题目说：

> metric must be added when traversing through multiple hops

意思是：

**每跳都要累加 cost**

这正是：

```
OSPF E1
```

------

# 3 当前问题

现在 metric **不会增加**

说明当前是：

```
metric-type 2
```

要改成：

```
metric-type 1
```

------

# 4 再看选项

关键判断两件事：

1️⃣ **配置必须在 R2**
因为 R2 是 **OSPF ↔ EIGRP 的 redistribution router**

```
R1 —— OSPF —— R2 —— EIGRP —— R3
```

所以：

```
redistribute eigrp → ospf
```

必须在 **R2**。

------

2️⃣ metric-type 必须是 **type 1**

------

### A

```
set metric-type type-2
```

❌ 错，因为 E2。

------

### B

```
R2
set metric-type type-1
```

✔ 正确。

------

### C

配置在 **R1**

❌ 错，因为 R1 不连 EIGRP。

------

### D

配置在 R1 + type2

❌ 更错。

------

# 5 正确配置

在 **R2**

```
access-list 10 permit 192.168.10.0 0.0.0.255
access-list 10 permit 192.168.20.0 0.0.0.255

route-map RD permit 10
 match ip address 10
 set metric 20
 set metric-type type-1

router ospf 10
 redistribute eigrp 10 subnets route-map RD
```

------

# 6 一句话秒杀口诀

看到题目：

```
metric increases each hop
```

答案：

```
OSPF E1
```

看到：

```
metric fixed
```

答案：

```
OSPF E2
```

------

# 7 ENARSI 超高频总结

| 类型   | 公式                        |
| ------ | --------------------------- |
| **E1** | external metric + OSPF cost |
| **E2** | external metric only        |

------

下面这张是 **ENARSI OSPF External LSA 秒杀表**。
基本所有 OSPF 重分发题都绕不开这几个点。看题时直接套表即可。

------

# ENARSI OSPF External LSA 秒杀表

## 1️⃣ External LSA 类型

| 类型   | 名称            | Metric 计算                          | 特点          | 常见用途       |
| ------ | --------------- | ------------------------------------ | ------------- | -------------- |
| **E1** | External Type 1 | **external metric + OSPF path cost** | 每跳增加      | 更精确路径选择 |
| **E2** | External Type 2 | **external metric only**             | 不随 hop 变化 | 默认类型       |

------

## 2️⃣ 计算示例

假设：

```
External metric = 20
OSPF path cost = 10
```

### E1

```
total metric = 20 + 10 = 30
```

如果再多一跳：

```
20 + 20 = 40
```

➡ **每跳增加**

------

### E2

```
total metric = 20
```

无论多少跳：

```
20
```

➡ **metric 不变**

------

# 3️⃣ Cisco 默认行为

如果你没有指定：

```
set metric-type
```

默认是：

```
metric-type 2
```

也就是：

```
E2
```

------

# 4️⃣ Redistribute 时配置

### 默认（E2）

```
router ospf 1
 redistribute eigrp 10 subnets
```

------

### 指定 E1

```
router ospf 1
 redistribute eigrp 10 subnets metric-type 1
```

或者 route-map：

```
route-map REDIST permit 10
 set metric 20
 set metric-type type-1
```

------

# 5️⃣ ENARSI 题目关键句

看到这些句子直接判断：

| 题目关键词                             | 答案   |
| -------------------------------------- | ------ |
| metric **adds per hop**                | **E1** |
| metric **increases across hops**       | **E1** |
| metric **includes internal OSPF cost** | **E1** |
| metric **remains constant**            | **E2** |
| metric **does not change per hop**     | **E2** |
| **default redistribution**             | **E2** |

------

# 6️⃣ 决策流程（考试秒杀）

看到题目：

```
redistribute → OSPF
```

问 metric 行为：

```
metric per hop ?
```

判断：

```
YES → E1
NO  → E2
```

------

# 7️⃣ ENARSI 常见陷阱

### 陷阱1

题目说：

```
metric fixed
```

答案：

```
E2
```

------

### 陷阱2

题目说：

```
metric accumulates
```

答案：

```
E1
```

------

### 陷阱3

题目说：

```
best path depends on internal cost
```

答案：

```
E1
```

因为 E1 会加内部 OSPF cost。

------

# 8️⃣ 真实网络设计建议

| 场景        | 推荐          |
| ----------- | ------------- |
| 小网络      | E2            |
| 大网络      | **E1**        |
| 多 ABR/ASBR | **E1 更准确** |

原因：

```
E2 不考虑内部 cost
可能选错路径
```

------

# 9️⃣ LSA 类型顺便复习

| LSA       | 名称             |
| --------- | ---------------- |
| LSA 1     | Router LSA       |
| LSA 2     | Network LSA      |
| LSA 3     | Summary LSA      |
| LSA 4     | ASBR Summary     |
| **LSA 5** | **External LSA** |
| LSA 7     | NSSA External    |

------

# 10️⃣ 一句话口诀

```
E1 = 每跳累加
E2 = metric 固定
默认 = E2
```

------

如果你愿意，我可以再给你做一张 **OSPF 30秒排错图（ENARSI 高频）**：

基本所有 OSPF 题都逃不过这 **6个问题**：

```
邻居
LSA
Area
Cost
External
Redistribute
```



------

# OSPF 30 秒排错图（ENARSI）

```
OSPF问题
   │
   ▼
1️⃣ 邻居建立了吗？
   │
   ├─ NO → 看 Area / Network / MTU / Auth
   │
   └─ YES
        │
        ▼
2️⃣ 路由存在 LSDB 吗？
        │
        ├─ NO → 看 LSA 类型 / ABR / Area type
        │
        └─ YES
             │
             ▼
3️⃣ 路由进 routing table 吗？
             │
             ├─ NO → 看 AD / 更优路由
             │
             └─ YES
                  │
                  ▼
4️⃣ 路径正确吗？
                  │
                  ├─ NO → 看 Cost
                  │
                  └─ YES
                       │
                       ▼
5️⃣ External route 吗？
                       │
                       ├─ YES → 看 E1 / E2
                       │
                       └─ NO
                            │
                            ▼
6️⃣ Redistribute 正确吗？
                            │
                            ├─ route-map
                            ├─ subnets
                            └─ metric
```

------

# 1️⃣ 邻居建立了吗

先看：

```
show ip ospf neighbor
```

如果 **没有邻居**：

检查四个东西（考试最常见）：

| 问题             | 现象             |
| ---------------- | ---------------- |
| Area 不一致      | stuck in EXSTART |
| Hello/dead timer | stuck in INIT    |
| MTU mismatch     | stuck in EXSTART |
| Authentication   | 无邻居           |

口诀：

```
OSPF 邻居 = Area + Hello + MTU + Auth
```

------

# 2️⃣ LSDB 有路由吗

```
show ip ospf database
```

如果 LSDB 没有：

说明 **LSA 没传播**。

常见原因：

| 问题        | 原因            |
| ----------- | --------------- |
| stub area   | 不接受 external |
| NSSA        | LSA7            |
| ABR summary | summary 过滤    |

------

# 3️⃣ LSDB 有但路由表没有

```
show ip route
```

原因通常是：

| 原因          | 说明              |
| ------------- | ----------------- |
| AD 更高       | 被别的协议抢走    |
| better metric | OSPF 路径不是最优 |

例：

```
EIGRP AD = 90
OSPF AD = 110
```

OSPF 不会进表。

------

# 4️⃣ 路径选错

看：

```
show ip ospf interface
```

检查：

```
cost
```

公式：

```
Cost = 10^8 / bandwidth
```

例：

| bandwidth | cost |
| --------- | ---- |
| 100M      | 1    |
| 10M       | 10   |

------

# 5️⃣ External route 问题

看：

```
show ip route ospf
```

会看到：

```
O E1
O E2
```

判断：

| 类型 | metric   |
| ---- | -------- |
| E1   | 每跳增加 |
| E2   | 固定     |

题目关键词：

| 题目说           | 答案 |
| ---------------- | ---- |
| metric increases | E1   |
| metric fixed     | E2   |

------

# 6️⃣ Redistribute 问题

最常见三件事：

### ① 忘记 subnets

```
redistribute static
```

错

```
redistribute static subnets
```

对

------

### ② metric 没设置

OSPF 默认 metric：

```
20
```

------

### ③ route-map 过滤

看：

```
match ip address
```

ACL 是否 permit。

------

# ENARSI 高频排错顺序（记住这 6 个）

```
邻居
LSA
路由表
Cost
External
Redistribute
```

口诀：

```
Neighbor
LSA
Route
Cost
External
Redistribute
```

------

# 最重要的一条考试经验

**OSPF 排错永远按这个顺序：**

```
邻居
↓
LSDB
↓
Routing table
↓
Metric
```

Cisco 官方 TAC 也是这个流程。

------

