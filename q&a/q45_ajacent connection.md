这题是典型 **OSPF 邻居卡在 EXSTART/EXCHANGE** 的考点。

------

## 🔎 题目关键信息

```
show ip ospf neighbor
State: EXSTART / EXCHANGE
```

问：应该采取什么措施？

选项：

A. match the passwords
B. match the hello timers
C. match the MTUs
D. match the network types

------

# ✅ 正确答案：C. match the MTUs

------

# 🧠 为什么？

## 📌 OSPF 邻接建立过程（超重要）

Down
→ Init
→ 2-Way
→ **ExStart**
→ **Exchange**
→ Loading
→ Full

------

## 🚨 EXSTART / EXCHANGE 卡住 = 99% 是 MTU 不一致

在 **ExStart 阶段**：

- 两台路由器协商 Master/Slave
- 开始交换 DBD（Database Description）包
- DBD 里包含 **接口 MTU 值**

如果两边 MTU 不一样：

```
Router A MTU 1500
Router B MTU 1400
```

👉 会拒绝对方的 DBD
👉 邻接卡在 EXSTART 或 EXCHANGE
👉 永远到不了 FULL

------

# ❌ 为什么其他选项错？

### A. 密码不匹配

认证失败 → 邻居根本起不来
通常停在：

```
DOWN
或者 INIT
```

不会到 EXSTART

------

### B. Hello timer 不匹配

Hello/Dead timer 不匹配：

邻居建立不了
通常卡在：

```
INIT
```

也到不了 EXSTART

------

### D. Network type 不匹配

比如：

- 一边是 broadcast
- 一边是 point-to-point

通常会卡在 2-Way 或邻接异常
但最经典 EXSTART 原因不是这个

------

# 🎯 考试记忆口诀

| 状态               | 问题        |
| ------------------ | ----------- |
| INIT               | Hello timer |
| 2-Way 卡住         | DR 相关     |
| EXSTART / EXCHANGE | 🚨 MTU       |
| Loading 卡住       | LSA 问题    |

------

# 🔥 现实排错命令

```bash
show interface g0/0 | include MTU
```

或者临时绕过：

```bash
ip ospf mtu-ignore
```

（考试一般选 MTU，不选 mtu-ignore）

------

# 🏁 总结

如果你在考试看到：

```
OSPF neighbor state EXSTART or EXCHANGE
```

不用想 —— 选：

> ✅ **match the MTUs**

------

好 👌
给你一张 **考试秒杀版 OSPF 邻接故障一页速记图**
（CCNP / ENCOR 高频）

------

# 🔥 OSPF 邻接故障一页速记图

------

## 🧠 一、邻接建立流程（必须背熟）

```
Down
  ↓
Init
  ↓
2-Way
  ↓
ExStart
  ↓
Exchange
  ↓
Loading
  ↓
Full
```

------

# 🚨 二、状态 → 故障定位 对照表（考试直接套）

| 卡在哪个状态 | 99% 问题               | 记忆关键词         |
| ------------ | ---------------------- | ------------------ |
| **Down**     | 对方没发 Hello         | 物理层 / IP 错     |
| **Init**     | 收到 Hello 但没双向    | ❗Hello/Dead 不匹配 |
| **2-Way**    | 不选 DR 或网络类型问题 | DR / Network Type  |
| **ExStart**  | ❗MTU 不匹配            | ⭐最经典考点        |
| **Exchange** | ❗MTU 不匹配            | ⭐同上              |
| **Loading**  | LSA 交换异常           | LSA 丢失           |
| **Full**     | 正常                   | 无问题             |

------

# 🎯 三、考试最常考 5 大坑

------

## 1️⃣ 卡在 INIT

👉 看到 INIT 就想：

- Hello timer 不一致
- Dead timer 不一致

检查：

```bash
show ip ospf interface
```

------

## 2️⃣ 卡在 2-Way

👉 多数是正常（在 Broadcast 网络）

如果题目说：

> “无法形成 FULL adjacency”

那就想：

- DR/BDR 选举
- Network type 不匹配

------

## 3️⃣ 卡在 EXSTART / EXCHANGE ⭐⭐高频

直接选：

> ✅ MTU 不匹配

检查：

```bash
show interface g0/0 | include MTU
```

解决：

```bash
ip ospf mtu-ignore
```

------

## 4️⃣ 邻居根本起不来（DOWN）

优先检查：

- IP 是否同网段
- Area 是否一致
- Authentication 是否一致

------

## 5️⃣ 认证问题

如果密码错：

邻居通常起不来

```
DOWN
或者
INIT
```

不会到 EXSTART

------

# 🧠 四、OSPF 建邻必须一致的 7 个东西

考试很爱考这个 👇

| 必须一致       | 否则现象   |
| -------------- | ---------- |
| Area ID        | 起不来     |
| Hello timer    | INIT       |
| Dead timer     | INIT       |
| Authentication | 起不来     |
| MTU            | EXSTART    |
| Network type   | 2-Way 异常 |
| Subnet         | 起不来     |

------

# 🚀 五、超级速记口诀

```
INIT 看 Timer
2-Way 看 DR
EXSTART 看 MTU
DOWN 看 IP/Area/密码
```

------

# 🎓 六、真实考试套路

题目通常这样出：

> show ip ospf neighbor
> State: EXSTART

你大脑应该自动触发：

```
EXSTART = MTU
```

不要犹豫。

------

# 🏁 七、终极一页压缩版（考前 30 秒复习）

```
INIT → Timer
2-Way → DR / Network type
EXSTART → MTU
DOWN → Area/IP/Auth
```

------

