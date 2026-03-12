**Routing Loops (A):** When you redistribute routes from OSPF into EIGRP and then back from EIGRP into OSPF at a different boundary point, a router might receive its own advertised route back with a "better" looking metric. This can create a feedback loop where packets circulate indefinitely between the two routing domains.

**Sub-optimal Routing (B):** OSPF and EIGRP use fundamentally different "math" to calculate path costs (OSPF uses **bandwidth/cost**, while EIGRP uses a composite of **bandwidth and delay**). Because these metrics don't translate directly, a router might choose a path through the redistributed domain that is actually slower or more congested than a native path, simply because the redistributed metric looks more attractive.





### Quick Comparison

| **Command**          | **Use Case**                                                 |
| -------------------- | ------------------------------------------------------------ |
| `debug ip routing`   | **Correct.** Shows when routes are installed or removed from the routing table. |
| `debug ip sla error` | Shows why a probe failed, but not the impact on the routing table. |
| `debug ip flow`      | Used for NetFlow troubleshooting (monitoring traffic flows). |
| `debug ip packet`    | Provides very detailed header info for every packet; too "noisy" for a routing table change. |

In the context of the exam question you shared earlier, the prompt said: *"The route has changed to flow through router R2. Which debug command is used to troubleshoot **this issue**?"*

1. **"This issue"** refers to the routing change itself.
2. While `debug ip sla error` tells you *why* the SLA failed, `debug ip routing` is what proves the router actually **modified the RIB** based on that failure.
3. In a real-world CCNP troubleshooting scenario, you use `ip routing` to verify that your backup floating static route (the one through R2) actually took over when the primary vanished.

### Key Takeaway for the CCNP

- Use **`debug ip sla error`** when you want to know: "Why is my probe failing?"
- Use **`debug ip routing`** when you want to know: "Is my routing table actually updating based on my tracking objects?"



Order of operations:

### Key Details to Remember for the Exam:

- **IP Removal:** When you apply the `ip vrf forwarding` command to an interface, Cisco IOS **automatically removes the IP address** from that interface. You must re-configure the IP address *after* assigning the VRF.
- **VRF Lite:** This specific configuration (VRF without MPLS) is often referred to as "VRF-Lite."
- **Routing Table Isolation:** Once configured, the interface will no longer use the global routing table; it will only look at the specific routing table for the `Inet` VRF.

### 1. VRF 接口配置的顺序 (VRF-Lite)

这是最常见的坑。

- **正确步骤：**
  1. 创建 VRF：`ip vrf Inet`
  2. 进入接口：`interface Fa0/0`
  3. **先绑定 VRF**：`ip vrf forwarding Inet`
  4. **后配 IP**：`ip address 10.1.1.1 255.255.255.0`

> **为什么？** 当你执行 `ip vrf forwarding` 时，路由器会认为该接口正在从“全局路由表”迁移到“私有路由表”。为了防止地址冲突和安全隐患，IOS 会**自动删除**该接口原有的 IP 地址。如果你先配了 IP 再绑定 VRF，你会发现 `show ip int brief` 里该接口的 IP 变成了 `unassigned`。

------

### 2. 控制层面与数据层面的顺序 (IP SLA)

回到你第一个问题，关于 IP SLA 和静态路由：

- **逻辑顺序：**
  1. **定义 SLA 探测器**（发包去检测 R3）。
  2. **定义 Track 对象**（盯着 SLA 的结果，是 OK 还是 Timeout）。
  3. **应用到路由**（在 `ip route` 后面加 `track` 编号）。

如果你先写了带 `track` 的静态路由，但还没定义 Track 对象，这条路由默认是不会生效的。

------

### 3. 数据包处理的顺序 (Inside vs Outside)

在涉及 NAT、ACL 和路由时，路由器处理数据包是有严格先后顺序的。这直接决定了你的 ACL 应该写“转换前”的地址还是“转换后”的地址。

**从 Inside 到 Outside (进入路由器)：**

1. **入站 ACL** (Input ACL) —— 先过滤。
2. **策略路由** (PBR)。
3. **路由查找** (Routing)。
4. **NAT 转换** (NAT Inside to Outside) —— **最后才转换地址**。

**从 Outside 到 Inside (进入路由器)：**

1. **入站 ACL** (Input ACL)。
2. **NAT 转换** (NAT Outside to Inside) —— **先转换地址**。
3. **路由查找** (Routing)。

------

### 4. 考试避坑小贴士

在做 CCNP 实验题或多选题时，如果发现配置都对但结果不对，检查这三点：

- **VRF：** IP 地址是不是因为绑定顺序不对被刷掉了？
- **ACL：** 它是应用在 NAT 转换前还是转换后？
- **OSPF/EIGRP：** 接口绑定 VRF 后，对应的路由进程（Network 命令）是否也移到了 VRF 模式下？





没错，你的公式抓住了 **EIGRP Variance** 的精髓！

具体到这道题的数值，计算逻辑如下：

- **最佳路径 (FD_best)** = 1,075,200
- **次优路径 (FD_sub)** = 2,611,200

按照你的公式 $FD\_best \times V > FD\_sub$：

- **如果 $V = 2$**: $1,075,200 \times 2 = 2,150,400$。这个值**小于** 2,611,200，所以次优路径被“拒之门外”。
- **如果 $V = 4$**: $1,075,200 \times 4 = 4,300,800$。这个值**大于** 2,611,200，次优路径成功上位，实现负载均衡。

### ⚠️ 必须满足的“隐藏前提”

光有公式还不够，EIGRP 有个死理：为了防环，次优路径必须先成为 **Feasible Successor (FS)**。

也就是说，在乘 $V$ 之前，次优路径必须满足：

> **它的 RD (Reported Distance) < 最佳路径的 FD**

在这道题中：

- **次优路径的 RD** = 281,600（括号里逗号后面的那个数字）
- **最佳路径的 FD** = 1,075,200
- 因为 $281,600 < 1,075,200$，它拿到了作为“备胎”的资格券，所以 $V=4$ 才能生效。

------

### 总结

1. **资格赛**：$RD_{sub} < FD_{best}$（满足此条件才能叫 Feasible Successor）。
2. **决赛圈**：$FD_{best} \times Variance > FD_{sub}$（满足此条件才能进入路由表做负载均衡）。

