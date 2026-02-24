

## 一句话核心

> R1 **已经有去 Internet 的路**，
> 但 OSPF 里的其他路由器 **不知道这条默认路由**。
> 所以我们要让 R1 在 OSPF 里“宣布”：
>
> 👉 “找不到路就来我这，我带你上网。”

这条“宣布”命令就是：

```cisco
default-information originate
```

------

# 一、先看网络结构（逻辑关系）

```
R3 — R2 — R1 — ISP — Internet
            |
        0.0.0.0 → 99.3.5.1
```

- R1 连着 ISP

- R1 有一条静态默认路由：

  ```
  ip route 0.0.0.0 0.0.0.0 99.3.5.1
  ```

这条路的意思是：

> 所有不知道去哪的流量 → 丢给 ISP

------

# 二、问题出在哪里？

R1 知道怎么上网 ✅
但：

- R2 不知道
- R3 不知道

因为 **R1 没把 0.0.0.0 发布到 OSPF**

所以 R2 / R3：

```
show ip route
```

里根本没有：

```
O*E2 0.0.0.0/0
```

它们不知道默认走 R1。

------

# 三、为什么不能用 redistribute static？

很多人会想：

```cisco
redistribute static
```

表面上看：

- 静态路由是 0.0.0.0
- 重分发静态不就行了吗？

但问题是：

### 1️⃣ 这是“粗暴重分发”

它会把所有 static 都丢进 OSPF
生产环境不推荐

------

### 2️⃣ OSPF有专门发布默认路由的命令

OSPF设计时就考虑了这种场景：

> “我有一条默认路由，想告诉大家”

于是给了一个专门命令：

```cisco
default-information originate
```

意思是：

> 如果我本地有默认路由，就把它变成 OSPF 外部缺省发出去

------

# 四、你要脑子里形成这个画面

R1 内心OSPF在说：

### ❌ 现在状态：

“我自己知道上网，但我不说。”

------

### ✅ 加命令后：

“各位听好了！
不会的都给我！我带你们出去！”

------

# 五、命令到底做了什么？

当你输入：

```cisco
router ospf 1
 default-information originate
```

OSPF 会：

- 检查自己有没有 0.0.0.0
- 如果有
- 就生成一条 Type 5 External LSA
- 向整个 OSPF 泛洪

结果：

R2 / R3 学到：

```
O*E2 0.0.0.0/0 via R1
```

网络恢复。

------

# 六、考试关键记忆点

| 场景                 | 用什么                          |
| -------------------- | ------------------------------- |
| 让 OSPF 发布默认路由 | ✅ default-information originate |
| 把别的协议引进来     | redistribute                    |
| 加接口进 OSPF        | network                         |

------

# 七、给你一句考试口诀

> OSPF 要发布默认
> 永远优先想
> **default-information originate**

而不是 redistribute。

------

好，这张图你必须彻底理解 ——
因为它是 **OSPF 默认路由题的核心套路**。

我给你做一个「考试级流程脑图」👇

------

# 🔥 OSPF 默认路由传播完整流程图

```
            Internet
                |
          99.3.5.1
                |
        R1 (ASBR)
--------------------------------
Step 1️⃣
R1 有静态默认路由
ip route 0.0.0.0 0.0.0.0 99.3.5.1
--------------------------------
Step 2️⃣
在 OSPF 里执行：
router ospf 1
 default-information originate
--------------------------------
Step 3️⃣
R1 生成 LSA
Type 5 External LSA
--------------------------------
Step 4️⃣
LSA 在 OSPF 泛洪
Area 0 → Area 5 → 所有区域
--------------------------------
Step 5️⃣
内部路由器安装路由
O*E2 0.0.0.0/0 via R1
--------------------------------
Step 6️⃣
流量转发
R3 → R2 → R1 → ISP → Internet
```

------

# 🧠 每一步讲人话解释

------

## ① R1 必须先有默认路由

```
ip route 0.0.0.0 0.0.0.0 99.3.5.1
```

这一步只是：

👉 R1 自己知道怎么上网
但别人还不知道

------

## ② 执行 default-information originate

这句命令的本质是：

> “如果我有默认路由，我就把它告诉 OSPF。”

它不会自动创建默认路由
它只是发布已有的。

------

## ③ 生成 Type 5 LSA

因为：

- 默认路由是外部路由
- 来自静态
- 不是 OSPF 自己算出来的

所以它是：

```
Type 5 = External LSA
```

R1 变成：

```
ASBR (Autonomous System Boundary Router)
```

------

## ④ LSA 泛洪

OSPF 特点：

- LSA 会在整个 OSPF 域泛洪
- 所有区域都会收到

除非：

- 是 stub area（考试常考陷阱）

------

## ⑤ 内部路由器学到

R2 / R3：

```
O*E2 0.0.0.0/0 via R1
```

解释：

- O = OSPF
- - = Candidate default
- E2 = External Type 2

------

## ⑥ 实际流量走向

当 R3 访问 8.8.8.8：

```
R3 查表 → 没匹配
→ 用默认
→ R2
→ R1
→ ISP
```

恢复 Internet。

------

# 🎯 重点记忆结构

```
有默认路由
      ↓
default-information originate
      ↓
Type 5 LSA
      ↓
全网泛洪
      ↓
O*E2 出现
```

------

# ⚠️ 考试最爱问的 3 个坑

### 1️⃣ 如果没有静态默认路由？

不会发布。

要加：

```
default-information originate always
```

------

### 2️⃣ 如果是 stub area？

Stub area 不接受 Type 5
会自动生成默认路由。

------

### 3️⃣ E1 vs E2？

默认是 E2
不累加内部 cost。

------

# 🔥 你现在应该形成的脑回路

看到题目：

- 有 ISP
- 有 static 0/0
- 内部不能上网

大脑自动反射：

👉 default-information originate

------

好，这张是 **OSPF LSA 类型一页速记图（考试版）**。
你把它记熟，OSPF 题 80% 不会慌。

------

# 🔥 OSPF LSA 类型总览（速记脑图）

```
                   OSPF LSA 类型
┌──────────────────────────────────────────────┐
│ Type 1  → Router LSA        (每台路由器自己) │
│ Type 2  → Network LSA       (DR 产生)        │
│ Type 3  → Summary LSA       (ABR 产生)       │
│ Type 4  → ASBR Summary LSA  (ABR 产生)       │
│ Type 5  → External LSA      (ASBR 产生)      │
│ Type 7  → NSSA External     (NSSA 内 ASBR)   │
└──────────────────────────────────────────────┘
```

------

# 🧠 一张逻辑结构图（必须脑补）

```
          Area 0
     ┌──────────────┐
     │   ABR        │
     └──────┬───────┘
            │
        Type 3
            │
     ┌──────────────┐
     │   Area 5     │
     └──────────────┘

ASBR 在某区域
    │
    └── Type 5 (全域泛洪)
```

------

# 🔎 逐个解释（人话版）

------

## 🥇 Type 1 – Router LSA

- 每个路由器自己生成
- 描述自己的接口、邻居
- 只在本区域传播

口诀：

> 1 是“自我介绍”

------

## 🥈 Type 2 – Network LSA

- 只有 DR 生成
- 描述多访问网络（广播网络）
- 只在本区域传播

口诀：

> 2 是“DR 说大家”

------

## 🥉 Type 3 – Summary LSA

- ABR 生成
- 把一个区域的网段告诉别的区域

口诀：

> 3 是“跨区域网段”

考试最常考。

------

## 🏅 Type 4 – ASBR Summary LSA

- ABR 生成

- 告诉别的区域：

  👉 “ASBR 在哪儿”

因为：

Type 5 需要知道 ASBR 位置。

口诀：

> 4 是“找 ASBR 的地图”

------

## 🏆 Type 5 – External LSA

- ASBR 生成
- 用于：
  - redistribute static
  - redistribute BGP
  - default-information originate

全域泛洪（除了 stub）

口诀：

> 5 是“外来路由”

------

## 🧨 Type 7 – NSSA 专用

- NSSA 内部 ASBR 生成
- 出 NSSA 时转换成 Type 5

口诀：

> 7 是“假 5 号”

------

# 🔥 高频考试陷阱

------

## ① Stub Area 不允许什么？

❌ Type 5
❌ Type 4

------

## ② Totally Stub 不允许什么？

❌ 3
❌ 4
❌ 5

只留默认路由。

------

## ③ NSSA 不允许什么？

❌ Type 5

用 Type 7 代替。

------

# 🎯 E1 / E2 属于谁？

只属于：

```
Type 5
Type 7
```

- E2 默认
- E1 累加内部 cost

------

# 🔥 终极记忆口诀

```
1 自我介绍
2 DR 广播
3 跨区网段
4 找 ASBR
5 外部路由
7 NSSA 外部
```

------

# 💡 考试秒解技巧

看到：

- redisribute → 想 Type 5
- ABR → 想 Type 3/4
- Stub 问题 → 想 5 被挡
- Default-information originate → Type 5

------

如果你愿意，我可以再给你做一张：

