在这个具体的拓扑结构中，我们需要先计算出每一条路径的 **可行距离 (Feasible Distance, FD)** 和 **报告距离 (Advertised Distance, AD)**。

在 EIGRP 中，要实现非等价负载均衡（Unequal Cost Load Balancing），必须满足 **可行性条件 (Feasibility Condition)**：

![image-20260327033909811](C:\Users\lihai\AppData\Roaming\Typora\typora-user-images\image-20260327033909811.png)

> **AD < 当前最短路径的 FD**

让我们从 **Router E** 的视角来看到达 **Network X** 的三条路径：

### 1. 计算各路径度量值

| **路径**            | **路径组成** | **全程度量值 (FD)** | **邻居通告的度量值 (AD)** |
| ------------------- | ------------ | ------------------- | ------------------------- |
| **路径 1 (通过 C)** | E -> C -> A  | **20** (10+10)      | 10 (C 到 A 的距离)        |
| **路径 2 (通过 B)** | E -> B -> A  | **30** (20+10)      | 10 (B 到 A 的距离)        |
| **路径 3 (通过 D)** | E -> D -> A  | **45** (20+25)      | 25 (D 到 A 的距离)        |

------

### 2. 确定 Successor 和 Feasibility Condition

1. **Successor (最短路径)**：路径 1（通过 Router C），**FD = 20**。
2. **检查 Feasible Successor (备份路径)**：
   - **路径 2 (Router B)**：其 AD (10) < 当前最短 FD (20)。**满足条件**，它是 Feasible Successor。
   - **路径 3 (Router D)**：其 AD (25) > 当前最短 FD (20)。**不满足条件**，它不是 Feasible Successor，无论你怎么调 variance，这条路都不会被放入路由表。

------

### 3. 为什么是 `variance 2`？

你的疑问点在于“公式方向”。`variance` 的作用是告诉路由器：允许把那些 **度量值小于（最短 FD × variance）** 且 **满足可行性条件** 的路径加入路由表。

公式应该是：

$$FD_{feasible\_path} < FD_{successor} \times variance$$

我们要让 **路径 2 (FD=30)** 参与负载均衡：

- 最短 $FD = 20$
- 目标 $FD = 30$
- $20 \times 1.5 = 30$

因为 `variance` 只能填整数，所以填 **`variance 2`** 就能覆盖到 40 以内的所有满足条件的路径。

### 总结

- **路径 2**：$30 < 20 \times 2$，且满足 AD(10) < FD(20)，**可以**负载均衡。
- **路径 3**：虽然 $45 > 20 \times 2$（不满足），但更重要的是它的 AD(25) > FD(20)，它连入选资格都没有。

所以，通过 `variance 2`，Router E 就可以同时利用路径 1 和路径 2 来转发流量，从而缓解“应用缓慢”的问题。

需要我帮你再找几道关于 **Feasibility Condition** 的练习题巩固一下吗？

