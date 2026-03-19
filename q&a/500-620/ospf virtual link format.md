**OSPF Virtual-Link 的命令格式是：** `area [transit-area-id] virtual-link [对端的 Router-ID]`

> **注意：** 必须是对端的 **Router-ID**，绝对不能是对端的接口 IP 地址（除非接口 IP 碰巧和 Router-ID 一样，但本题中显然不一样）。

- R2 应该指向 R4 的 Router-ID：`0.0.0.44`
- R4 应该指向 R2 的 Router-ID：`0.0.0.22`

由于它们配错了目标，`Virtual Link Status` 显示为 **DOWN**，导致 Area 250 无法连接到骨干区域（Area 0）。

- **对应选项 A (R2)：**
  - `no area 234 virtual-link 10.34.34.4`（删掉错的）
  - `area 234 virtual-link 0.0.0.44`（换成对端真正的 RID）
- **对应选项 E (R4)：**
  - `no area 234 virtual-link 10.23.23.2`（删掉错的）
  - `area 234 virtual-link 0.0.0.22`（换成对端真正的 RID）

------

### **OSPF 虚链路排错口诀**

1. **看 Transit Area：** 必须是相同的非骨干区域（如 Area 234），且不能是 Stub/NSSA 区域。
2. **看 Target：** 必须是**对端 Router-ID**。
3. **看可达性：** Transit Area 内 ABR 之间的物理接口必须能互相 Ping 通（通过 OSPF 学习到路由）。

**避坑指南：** 考试中如果看到 `show ip ospf virtual-links` 结果里 `State` 是 **DOWN**，第一眼就去对比 `router-id` 和 `virtual-link` 后面跟的那个 IP 是不是同一个。

既然你已经掌握了虚链路的 Router-ID 坑，要不要顺便了解一下为什么 **Stub Area（末梢区域）** 不能作为虚链路的 Transit Area？（这涉及到 OSPF 这种链路状态协议的一个底层设计逻辑）