**uRPF（单播反向路径转发）** 的两种工作模式：**Strict Mode（严格模式）** 和 **Loose Mode（松散模式）**，以及在多归属（Multihomed）网络环境中的应用。

### 题目分析

1. **现状：**
   - **Fa0/0 (ISP1):** 配置了 `ip verify unicast source reachable-via any allow-default`。这是 **Loose Mode**（因为使用了 `any` 关键字），并且允许默认路由通过验证。
   - **Fa0/1 (ISP2):** 配置了 `ip verify unicast source reachable-via rx`。这是 **Strict Mode**（因为使用了 `rx` 关键字）。
2. **问题：** * 流量从 ISP1 出去，但从 ISP2 返回。
   - 在 Fa0/1（ISP2）上，由于开启了 **Strict Mode**，路由器会检查：*“这个包的源 IP 按照我的路由表，是否应该从 Fa0/1 进来？”*
   - 由于路由表可能指示该路径应通过 ISP1，或者存在异步路由（Asymmetric Routing），Strict Mode 校验失败，导致合法返回流量被丢弃。

------

### 解题核心：uRPF 模式对比

- **Strict Mode (`reachable-via rx`)：** 要求源 IP 不仅在路由表中存在，而且其最佳路径的 **出接口** 必须与流量的 **入接口** 完全一致。
  - *适用场景：* 只有一条上行链路（单归属）的情况。
  - *局限性：* 在异步路由（如本题）中会误杀流量。
- **Loose Mode (`reachable-via any`)：** 只要路由表中存在该源 IP 的条目（无论哪个接口），流量就允许通过。
  - *适用场景：* 多归属网络、异步路由环境。