

------

# 一、R2 → R1 的 AS-PATH 变化图

## 🟢 情况一：正常（没有 prepend）

假设：

- R2 在 AS 65000
- R1 在 AS 65100
- R2 宣告 192.168.130.0/24

------

### R2 发出去的 BGP Update：

```
Network: 192.168.130.0/24
AS-PATH: 65000
```

------

### R1 看到的：

```
65100 ← 65000
```

R1 认为：

> 这条路距离我 1 个 AS 跳

------

------

## 🔴 情况二：使用 prepend

R2 配置：

```
set as-path prepend 65000
```

它在发之前，会变成：

```
AS-PATH: 65000 65000
```

------

### 结构图变成：

```
65100 ← 65000 ← 65000
```

虽然物理上还是一个 R2

但逻辑上：

> 路径长度 = 2

------

### R1 现在认为：

> 这条路 2 AS hops away

------

# 二、为什么这样有用？

因为 BGP 选路时：

# ✅ 会优先选择 AS-PATH 短的

------

如果 R1 同时收到两条路：

| 路径   | AS-PATH     |
| ------ | ----------- |
| Path A | 65000       |
| Path B | 65000 65000 |

R1 一定选：

```
65000
```

因为更短。

------

# 三、BGP 选路顺序（最重要部分）

这个一定要背下来。

面试会问：

> BGP best path selection order?

------

## 🏆 BGP 选路完整顺序（从上到下）

------

### 1️⃣ Highest Weight（Cisco 私有）

✔ 数值大优先
默认本地路由 weight = 32768

------

### 2️⃣ Highest Local Preference

✔ 数值大优先
默认 = 100

用于控制 **整个 AS 内部出口**

------

### 3️⃣ Locally originated route

本地产生的优先
(network / redistribute)

------

### 4️⃣ Shortest AS-PATH

✔ 越短越好
这就是 prepend 生效的地方

------

### 5️⃣ Lowest Origin Type

顺序：

```
IGP < EGP < Incomplete
```

IGP 最好

------

### 6️⃣ Lowest MED

✔ 越小越好
通常用于两个 AS 之间

------

### 7️⃣ eBGP 优于 iBGP

来自外部 AS 的优先

------

### 8️⃣ Lowest IGP metric to next-hop

看内部路由距离

------

### 9️⃣ Oldest path

最老的稳定路径优先

------

### 🔟 Lowest Router ID

最后比 Router-ID

------

# 四、超简化记忆口诀（给你用）

我给你压缩成考试记忆版：

```
Weight
Local Preference
Local
AS-PATH
Origin
MED
eBGP
IGP metric
Router ID
```

------

# 五、考试常考陷阱

### ❗ prepend 不会：

- 阻止路由
- 过滤路由
- 改变下一跳
- 改变 local-pref

它只影响：

# 👉 第4步：AS-PATH 长度

------

# 六、面试会怎么问你？

> 为什么我们用 AS-PATH prepend？

标准答案：

> To make a route less preferred by increasing the AS-PATH length.

------

# 七、给你一个真实 ISP 场景

公司有两个出口：

```
出口A → ISP1
出口B → ISP2
```

想让别人优先走 ISP1

就在 ISP2 上：

```
set as-path prepend 65000 65000 65000
```

这样：

外部世界更少选 ISP2

------

# 八、你现在应该真正理解了

AS-PATH prepend 本质是：

> 人工“拉长”路径，让别人不选你

------

