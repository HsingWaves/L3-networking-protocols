好的，我按“完全小白”的方式，把**题目在问什么、图里每个东西代表什么、为什么会坏、为什么用 `no eigrp stub` 就能修**，一步一步讲清楚。

------

## 1）题目到底在说啥？

题目一句话翻译：

> 分公司（Branch）那边的用户网段 **2001:db8:0:4::/64** 说上不了互联网。
> 这套网络用了 **IPv6 + EIGRP（AS 100）**。
> 问：要在 **IPv6 EIGRP 100 的配置模式**下敲哪条命令才能解决？

所以它不是在问你“怎么配 IPv6”，而是在问：

**EIGRP 的一个配置导致 Branch 收不到能上网的路由**，你要用一条命令修复。



Branch 要能上 Internet，它必须有：

```
::/0
```

也就是：

> 🌍 默认路由（Default Route）

# 🧠 小技巧（考试实战）

看到这种题：

1️⃣ 看出口路由器（R2）有没有默认路由
 2️⃣ 看中间路由器（R1）有没有
 3️⃣ 看下游（Branch）有没有

只要某一层“断了”：

👉 问题就在那一层

------

# 🧩 再教你一个更高级的看法

在 topology 里每条路由会有：

```
P prefix, successors, FD
    via ...
```

如果一个 prefix 根本没出现：

👉 表示这个路由“从来没学到”

如果出现但没有 successor：

👉 表示在 Active 状态（正在计算）

------

# 🧠 所以这题真正看的是

> Branch topology 里有没有 ::/0？

没有 = 不通 Internet

------

# 🎯 总结一句话

从 Branch 的 EIGRP topology 判断是否连到 Internet：

👉 只需要看有没有 `::/0`

- 有 = 连上
- 没有 = 没连上

------

## 2）先读拓扑图：谁是谁？

图里有 3 台路由器：

- **R2（左边）**：连着 Internet（云那边 2001:db8:f::/??）
- **R1（中间）**：中间转发（核心/汇聚）
- **Branch（右边）**：分公司路由器，右侧挂着用户网段 **2001:db8:0:4::/64**

链路网段大概是：

- R2 ↔ R1：`2001:db8:0:12::/64`
- R1 ↔ Branch：`2001:db8:0:14::/64`
- Branch LAN：`2001:db8:0:4::/64`

**现实含义：**
Branch 的电脑要上网，流量必须走：
**PC → Branch → R1 → R2 → Internet**

所以只要其中有一段“路由信息没传过去”，Branch 就会不知道“去 Internet 应该往哪走”。

------

## 3）EIGRP 在这里干啥？

EIGRP 是一种动态路由协议，作用就是：

- 让各路由器互相“通告”自己知道的网段
- 让彼此自动学到路由表（不用你手工一条条写静态路由）

EIGRP is a dynamic routing protocol.

Its function is to:

👉 allow routers to automatically learn from each other "where to go and how to get there".

EIGRP uses the Diffusing Update Algorithm (DUAL) to compute loop-free and backup routes.

| 特点           | 含义             |
| -------------- | ---------------- |
| 快             | 收敛快           |
| 智能           | 只更新变化的路由 |
| 使用 DUAL 算法 | 计算无环路径     |
| 有 topology 表 | 维护备选路径     |

发现邻居（Hello）

交换路由信息（Update）

计算最优路径（DUAL）DUAL = **Diffusing Update Algorithm**

放进路由表：

✅ **Branch 必须学到：默认路由（::/0）或者去 Internet 的路由**
否则 Branch 不知道 Internet 在哪，就上不了网。

------

## 4）关键概念：什么是 EIGRP Stub（存根）？

把它理解成一个“声明”：

> “我是边缘路由器（小弟），我不负责帮别人转发路由信息。
> 别指望我当中转站。”

### Stub 的用途（为什么有人要开 stub）

通常在分支网络里，Branch 就是末端（leaf），开 stub 可以：

- 减少 EIGRP 查询（Query）和计算压力
- 让网络更稳定

### 但 stub 有一个致命规则

**Stub 路由器不会把它从别的邻居学到的路由再转发出去。**

换句话说：

- 你从 A 学到的路由，不会再发给 B
  （它只发自己的直连/静态/汇总等有限类别）

------

## 5）这题为什么会坏？（用“传话”比喻）

想象：

- R2 知道“Internet 在我这里”
- R1 是中间人
- Branch 需要 R1 把“去 Internet 怎么走”告诉它

正常情况应该是：

1. R2 告诉 R1：我这边有“去 Internet 的路”（比如默认路由 ::/0）
2. R1 再告诉 Branch：去 Internet 走我（R1），我再带你去 R2
3. Branch 就能上网了 ✅

但如果 **R1 被配置成了 stub**，就变成：

1. R2 告诉 R1：我有去 Internet 的路
2. **R1：我虽然听到了，但我不负责转告别人（stub）**
3. Branch 永远听不到“去 Internet 的路” ❌
4. 所以 Branch 用户上不了网

**核心矛盾：**

> R1 是“中转路由器”（transit），但却被当成“边缘路由器”（stub）来配置了。

------

## 6）为什么答案是 `no eigrp stub` 在 R1？

题目问的是：

> “在 IPv6 router EIGRP 100 配置模式下发什么命令解决？”

如果问题是 “R1 被配置成 stub 导致不转发路由”，那么修复就是：

✅ 把 R1 的 stub 取消

也就是：

```cisco
ipv6 router eigrp 100
 no eigrp stub
```

------

## 7）为什么不是对 R2 做？

你可能会想：R2 才连 Internet 啊，为什么不是改 R2？

因为问题不是“Internet 那边没路由”，而是：

- R2 → R1 这段没问题（R1 能学到）
- **R1 → Branch 这段路由没传过去**

所以动 R2 没用，瓶颈在 **R1 的“转告能力”**。

------

## 8）用一句“考试必背”总结

> **EIGRP Stub 只能用在边缘/末端路由器，不能用在中间转发（transit）路由器。**
> 否则它不会把学到的路由转发给下游，导致下游缺路由、上不了网。

在这题里：

- Branch 才像边缘
- R1 是中间转发
- 所以 **R1 不能是 stub**，要 `no eigrp stub`

------

## 9）你可以怎么验证你真的理解了？

你只要回答这个问题就行：

> “Branch 上不了网，是因为它缺了哪条最关键的路由？”

答案：**默认路由（::/0）或者去 Internet 的路由**
而它缺这条路由的原因：**R1 是 stub，不转发从 R2 学来的路由**。

------

