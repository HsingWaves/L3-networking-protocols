# 

```
                EIGRP Troubleshooting
                      |
     ------------------------------------------------
     |                    |              |           |
  Auto Summary       Variance       Stuck-in-Active  K-values
  (最常见)           Load Balance      (SIA)          mismatch
```

------

# 1️⃣ Auto Summary（最常见）

### 出现场景

```
172.16.1.0/24
      |
      R1
       \
        10.0.0.0
         \
          R2
         /
        10.0.1.0
       /
      R3
      |
172.16.2.0/24
```

### 问题

EIGRP默认会汇总：

```
172.16.1.0/24 → 172.16.0.0/16
172.16.2.0/24 → 172.16.0.0/16
```

R2 routing table：

```
172.16.0.0/16 via R1
172.16.0.0/16 via R3
```

结果：

```
172.16.2.x
可能被送到 R1
→ packet drop
```

### 解决

```
router eigrp 100
 no auto-summary
```

### 关键词

题目如果出现：

```
discontiguous network
unexpected load balancing
packet drop
172.16.x.x summary
```

**90% 就是这个。**

------

# 2️⃣ Variance（不等价负载均衡）

EIGRP 支持：

```
unequal cost load balancing
```

例如：

```
path1 metric 1000
path2 metric 2000
```

默认：

```
只用最优路径
```

开启 variance：

```
router eigrp 100
 variance 2
```

允许：

```
1000 × 2 = 2000
```

所以两条路都可用。

### 常见考点

问：

```
How to allow unequal load balancing?
```

答案：

```
variance
```

------

# 3️⃣ Stuck in Active (SIA)

EIGRP 查询机制：

当 route 消失：

```
router sends QUERY
neighbors must reply
```

如果某个 router 不回复：

```
SIA (Stuck In Active)
```

邻居关系会 reset。

### 常见原因

```
slow link
CPU high
query boundary missing
```

### 解决

使用：

```
EIGRP stub
router eigrp 100
 eigrp stub
```

这样 query 不会扩散。

------

# 4️⃣ K-values mismatch

EIGRP metric：

```
bandwidth
delay
reliability
load
MTU
```

由 **K-values** 控制。

默认：

```
K1 = 1
K3 = 1
```

如果两台路由器不同：

```
邻居不会建立
```

show log：

```
K-value mismatch
```

### 解决

```
metric weights 0 1 0 1 0 0
```

两边必须一样。

------

# ENARSI EIGRP 秒杀口诀

```
丢包 + 172.16 + 10.x 中间
= auto-summary
不等价负载
= variance
邻居突然 reset
= SIA
邻居起不来
= K-values
```

------

