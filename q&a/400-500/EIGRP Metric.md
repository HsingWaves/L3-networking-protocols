Even if you set a `variance` high enough to include a backup route, EIGRP will **never** install that route into the routing table unless it first satisfies the **Feasibility Condition (FC)**.

### 1. The Core Problem: The Feasibility Condition

For a backup route to be considered for load balancing, its **Reported Distance (RD)**—the metric advertised by the neighbor—must be strictly less than the **Feasible Distance (FD)** of the current best route.

Looking at your image:

- **Successor (Best Route):** Via 10.4.1.5. Its **FD is 409,600**.
- **Potential Feasible Successor:** Via 10.3.1.6. Its **RD is 409,600** (this is the second number in the brackets: `435200/409600`).

**The math check:**

Is the RD of the backup route ($409,600$) < the FD of the successor ($409,600$)?

**No.** It is equal, not less than.

Because $409,600$ is not $< 409,600$, the second route fails the Feasibility Condition. It is not even considered a "Feasible Successor," so the `variance` command (Options B and D) has no effect.

Since the second route is not a Feasible Successor, **changing the metric (delay) to meet the feasibility condition** is the only way to make it eligible for load balancing.

### 具体的 Metric 参数

在实际操作修改时（即选项 C 的执行），EIGRP 默认公式主要看这两项：

1. **带宽 (Bandwidth)：** 取路径上所有链路的**最小值**（最窄瓶颈）。
2. **延迟 (Delay)：** 取路径上所有链路的**总和**。

**为什么通常改 Delay 而不改 Bandwidth？**

- **累计性：** Delay 是累加的，微调它可以非常精确地控制总 Metric。
- **稳定性：** 修改带宽可能会影响 OSPF 等其他协议的选路或 QoS 策略，而修改 Delay 对设备负载影响最小，是调整 EIGRP 选路的最推荐做法。