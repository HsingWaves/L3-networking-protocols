好，我们画一个 **双 ISP 出口流量决策逻辑图**，你一看就彻底通透。

------

# 🌍 场景拓扑

```
            ISP1 (高延迟)
                |
              R1
               \ 
                \____ 你的 AS (公司)
                /
               /
              R2
                |
            ISP2 (正常延迟)
```

- R1 连接 ISP1（高 delay）
- R2 连接 ISP2（正常）
- 你希望：只要 ISP2 在，就走 ISP2

------

# 🧠 BGP 出站选路决策流程（关键部分）

当两条路都能到 8.8.8.0/24 时：

```
        收到两条 eBGP 路由
                ↓
        比较 Weight
                ↓
        比较 LOCAL_PREF   ← ★ 这里决定
                ↓
        比较 AS-PATH
                ↓
        比较 MED
                ↓
        选最佳路径
```

------

# 🎯 解决方案：提高 ISP2 的 LOCAL_PREF

默认：

```
LOCAL_PREF = 100
```

配置：

```
ISP2 → LOCAL_PREF 200
ISP1 → LOCAL_PREF 100
```

------

# 🔥 选路结果

| ISP  | LOCAL_PREF | 结果 |
| ---- | ---------- | ---- |
| ISP1 | 100        | 备份 |
| ISP2 | 200        | 主用 |

只要 ISP2 up：

→ 所有流量都会走 ISP2
→ ISP1 自动成为 backup

------

# 🚨 如果 ISP2 down 会怎样？

BGP 会撤销 ISP2 的路由：

```
只有 ISP1 路由存在
```

→ 自动切换到 ISP1
→ 不需要额外配置

------

# 🆚 为什么不用其他属性？

| 属性    | 问题               |
| ------- | ------------------ |
| Weight  | 只在单台路由器有效 |
| AS-PATH | 影响别人怎么进来   |
| MED     | 不适合不同 ISP     |

------

# 🎓 一句话黄金记忆法

```
控制自己出口 → LOCAL_PREF
控制别人入口 → AS-PATH prepend
```

------

好，给你做一份 **考试冲刺版** 的 ——
看完基本 BGP 选择题不会错。

------

# 🔥 一页速记图：BGP 选路顺序（Exam Version）

记忆口诀：

> **We Love AS Oranges More Than People Like Ripe Apples**

对应顺序如下 👇

```
1️⃣  Weight              （Cisco 私有，越大越优）
2️⃣  LOCAL_PREF          （越大越优）
3️⃣  Locally originated  （network / redistribute）
4️⃣  AS-PATH             （越短越优）
5️⃣  ORIGIN              （IGP < EGP < Incomplete）
6️⃣  MED                 （越小越优）
7️⃣  eBGP 优于 iBGP
8️⃣  IGP metric 到 next-hop （越小越优）
9️⃣  最早收到的
🔟  Router ID 最小
```

------

## 🧠 结构理解图

```
           控制自己出口
         ───────────────
         Weight
         LOCAL_PREF
         ↓
     --------------------
         影响路径结构
         AS-PATH
         ORIGIN
         MED
         ↓
     --------------------
         技术细节比较
         eBGP vs iBGP
         IGP cost
         Router-ID
```

------

# 🎯 考试高频 5 大坑点

------

## ⚠️ 坑 1：LOCAL_PREF vs MED 搞混

很多人会选 MED。

记住：

| 场景                 | 用什么       |
| -------------------- | ------------ |
| 控制自己整个 AS 出口 | ✅ LOCAL_PREF |
| 同一 ISP 多条链路    | MED          |

不同 ISP 之间默认不比较 MED。

------

## ⚠️ 坑 2：Weight 只本地有效

Weight：

- 只在本路由器有效
- 不传播
- Cisco 私有

如果题目有 **两台边界路由器**

→ 不能只用 Weight

必须用 LOCAL_PREF。

------

## ⚠️ 坑 3：AS-PATH 是控制“别人”怎么进来

- prepend = 让自己路径变长
- 影响 inbound traffic

很多人误以为可以控制自己出口。

❌ 不行。

------

## ⚠️ 坑 4：eBGP 一定优先于 iBGP

即使：

- AS-PATH 一样
- MED 一样

只要一个是 eBGP，一个是 iBGP

→ eBGP 赢

------

## ⚠️ 坑 5：Next-hop 不可达 = 路由直接丢弃

很多题目会给你：

```
BGP table 有路由
Routing table 没有
```

原因通常是：

- next-hop 不可达
- 没有 IGP 支持

BGP 选路之前：

> next-hop 必须可达

------

# 🚀 考试最重要 3 条（浓缩版）

```
出口控制 → LOCAL_PREF
入口控制 → AS-PATH prepend
不同 ISP 不比 MED
```

------

好，这个是 **CCNP 级别必杀图**。
你之前已经做过不少 redistribution 环路题，这张图会把逻辑彻底打通。

------

# 🧠 BGP + OSPF + EIGRP 互相重分发一页脑图

------

## 🌍 三协议角色定位

```text
          🌐 BGP
     （对外 Internet）

        ↑       ↓

    🟢 OSPF     🔵 EIGRP
   （企业核心） （分支/园区）
```

------

# 🔁 重分发总结构

```text
BGP  → IGP  → BGP
  ↑               ↓
  └────── 可能产生环路 ──────┘
```

核心问题：

> 一条路进来后，又被送回原协议。

这就是 **re-injection loop**

------

# 📌 每个方向要点速记

------

## 1️⃣ BGP → OSPF

### 典型配置

```bash
router ospf 1
 redistribute bgp 65000 subnets
```

### 结果

- 变成 OSPF External
- 类型默认 E2

------

### ⚠️ 关键点

| 类型 | 特征                    |
| ---- | ----------------------- |
| E1   | 计算内部 cost           |
| E2   | 不计算内部 cost（默认） |

考试常考：

> E1 和 E2 区别

------

------

## 2️⃣ OSPF → BGP

```bash
router bgp 65000
 redistribute ospf 1
```

### 结果

- 进入 BGP table
- origin 变成 ? (incomplete)

------

### ⚠️ 关键点

如果同时：

```text
BGP → OSPF
OSPF → BGP
```

你就会得到：

🔥 环路

------

------

## 3️⃣ EIGRP ↔ OSPF

这是最容易炸的组合。

```text
EIGRP → OSPF
OSPF → EIGRP
```

如果不加过滤：

→ 路由来回传
→ metric 被重算
→ 产生回灌

------

# 🚨 三协议互相重分发大环路图

```text
       Internet
          |
        BGP
          |
      redistribute
          ↓
        OSPF
          |
      redistribute
          ↓
        EIGRP
          |
      redistribute
          ↓
        OSPF
```

如果没有 tag 过滤：

🔥 无限循环

------

# 🏷 正确做法：Route Tag 防环

------

## 原理

当你重分发时：

```bash
redistribute eigrp 100 route-map TAG_EIGRP
```

route-map:

```bash
set tag 100
```

然后在反方向：

```bash
match tag 100
deny
```

------

## 逻辑图

```text
EIGRP → OSPF
         tag 100

OSPF → EIGRP
         如果 tag 100
         → 不允许
```

🔥 环路断开

------

# 🎯 互相重分发黄金规则

```text
永远不要双向无脑 redistribute
必须加 route-map + tag
```

------

# 📌 三协议默认管理距离

| 协议           | AD   |
| -------------- | ---- |
| eBGP           | 20   |
| EIGRP internal | 90   |
| OSPF           | 110  |
| EIGRP external | 170  |
| iBGP           | 200  |

考试喜欢考：

> 为什么 EIGRP 赢 OSPF？

因为 AD 90 < 110

------

# 🧨 典型考试陷阱

1. 双向 redistribute 无过滤
2. 忘记 subnets（OSPF）
3. 忘记 metric（EIGRP）
4. 忘记 next-hop-self（BGP）
5. 忘记 tag 防环

------

# 🧠 最终浓缩脑图（终极记忆版）

```text
出口控制：LOCAL_PREF
入口控制：AS-PATH

重分发原则：
1️⃣ 单向优先
2️⃣ 双向必须打 tag
3️⃣ EIGRP 必须给 metric
4️⃣ OSPF 记得 subnets
5️⃣ 注意 AD 优先级
```

------

