在 Route-map 的配置中，`permit 10`、`permit 20` 这种写法类似于编程里的 **`if-else if`** 语句或者 **`switch-case`** 分支。

### 1. 基本定义：Sequence Number (序列号)

- **`CCNP`**: 这是 Route-map 的名字。
- **`permit`**: 这是策略动作。如果是 `permit`，匹配成功就执行 `set` 语句；如果是 `deny`，匹配成功就跳过此策略，去走常规路由。
- **`10` / `20`**: 这是**序列号 (Sequence Number)**。Route-map 会按照序列号**从小到大**依次匹配。一旦匹配成功，就不再看后面的条目。

------

### 2. 为什么需要多个序列号？（应用场景）

如果没有序列号，你只能对所有流量执行同一种简单的操作。有了序列号，你可以实现**复杂的逻辑分流**：

#### 场景 A：分流处理 (Traffic Steering)

想象你有一个公司网络，你想实现：

- **10号员工**走电信线路 (序列号 10)。
- **20号员工**走联通线路 (序列号 20)。
- **其他人**走默认网关。

#### 场景 B：排除特定流量（对应你上一题的逻辑）

如果你想对一个网段做处理，但要**排除其中某一台主机**：

1. **序列号 10**: 先写一个 ACL 把那台主机 `deny` 掉。这样这台主机在匹配序列号 10 时会失败（不符合条件）。
2. **序列号 20 / 隐式拒绝**: 如果没有后续匹配，这台主机就会脱离 Route-map 的控制，去走正常的路由表。

------

### 3. 为什么 Option A 和 B 会在这里栽跟头？

结合你发的那道 PBR 题目，我们来看看多余的序列号或错误的匹配会发生什么：

- **在 Option A 中：** 它多写了一个 `route-map CCNP permit 20`。 当 `192.168.1.1` 进来时，它在 `permit 10` 里因为 ACL 的 `deny` 匹配失败了。由于有 `permit 20` 存在，它会接着去匹配 20。如果 20 后面什么都没写（默认匹配所有），那 `1.1` 最终还是会被抓住并执行 `set` 动作。这就好比你本想放走他，结果在后门又把他抓回来了。
- **在 Option D（正确答案）中：** 它只有 `permit 10`。 当 `1.1` 在 `permit 10` 匹配失败后，后面没有 `permit 20` 了。Route-map 结束匹配，将 `1.1` 踢回给 CPU 按照**普通路由表 (Normal Routing Table)** 转发。这正是题目想要的效果：不标记，正常走。

------

### 核心口诀

> **Route-map 像滤网，序号从小排到大。** **匹配成功就执行，匹配失败看下一家。** **全都失败怎么办？剔除策略回原话（走普通路由）。**





在 **PBR (策略路由)** 的上下文中，`route-map` 语句本身的 `deny` 动作非常容易让人产生误解。

简单来说：**Route-map 的 `deny` 不是“丢弃数据包”，而是“拒绝执行策略”。**

------

### 1. `route-map CCNP deny 10` 的真正含义

当你配置 `route-map [名字] deny [序列号]` 时，路由器的逻辑是：

1. **检查 Match 条件：** 如果数据包匹配了 `match` 语句（比如匹配了 ACL 里的 `permit`）。
2. **执行 Deny 动作：** 路由器说：“这个包我**不归我管**，我不对它进行任何 `set` 操作（不改下一跳、不改优先级）”。
3. **后续处理：** 数据包被“踢出” Route-map，直接交给 **普通的路由表 (Routing Table)** 进行转发。

------

### 2. 重点对比：`deny` 出现在不同位置的区别

这是 CCNP 考试中最阴险的考点。请看下面两个配置的区别：

#### 情况 A：`deny` 在 ACL 里 (Option D 的逻辑)

Plaintext

```
access-list 1 deny 192.168.1.1
access-list 1 permit any
route-map CCNP permit 10
  match ip address 1
  set ip precedence critical
```

- **逻辑：** 1.1 进来，ACL 说 `deny`（不匹配）。Route-map 发现 `permit 10` 没匹配上，且后面没序列号了，所以 1.1 **走普通路由**。

#### 情况 B：`deny` 在 Route-map 语句里 (Option C 的逻辑)

Plaintext

```
access-list 1 permit any
route-map CCNP deny 10
  match ip address 1
  set ip precedence critical  <-- 注意：这行在 deny 模式下会被忽略！
```

- **逻辑：** 所有的包进来，ACL 都说 `permit`（匹配）。但 Route-map 这一行开头是 `deny`。
- **结果：** 路由器直接说：“匹配成功了，但我拒绝执行策略”。结果就是**所有包都走普通路由**，没有任何包会被标记。

------

### 3. 总结表：PBR 中的匹配结果

| **Route-map 语句** | **ACL 条目** | **最终结果**                                       |
| ------------------ | ------------ | -------------------------------------------------- |
| **permit**         | **permit**   | **执行 `set` 动作** (策略路由成功)                 |
| **permit**         | **deny**     | 不匹配此行，看下一个序列号；若无，则**走普通路由** |
| **deny**           | **permit**   | 匹配成功，但**拒绝执行策略**，直接**走普通路由**   |
| **deny**           | **deny**     | 不匹配此行，看下一个序列号；若无，则**走普通路由** |

------

### 4. 为什么会有 `route-map deny`？

既然 `deny` 是让它走普通路由，那为什么不直接不写这行呢？

- **用途：** 它可以用来快速“排除”某些流量。
- **例子：** “我想让财务部的流量走专线，但财务部的**经理**必须走普通宽带（因为专线贵）”。
  - `route-map TO_SPECIAL deny 10` (match 经理 IP) —— 经理直接被踢回普通路由。
  - `route-map TO_SPECIAL permit 20` (match 财务部全员) —— 剩下的人被 `set` 到专线。

**避坑指南：**

在 PBR 中，如果你想让一个包被**彻底丢弃 (Drop)**，你不能靠 `route-map deny`。你必须在 `route-map permit` 里 `set ip next-hop` 到一个不存在的地址，或者直接用普通的 ACL 在接口入站方向 `deny` 掉。



| **配置方式**                   | **程序员视角的逻辑 (C++ 伪代码)**                            |
| ------------------------------ | ------------------------------------------------------------ |
| **ACL permit** + **RM permit** | `if (packet == target) { modify_packet(); return; }`         |
| **ACL deny** + **RM permit**   | `if (packet == target) { /* do nothing, go to next sequence */ }` |
| **RM deny**                    | `if (packet == target) { use_standard_routing(); return; }`  |