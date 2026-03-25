| **特性 (Feature)**     | **定义关键动作**                                   | **英文原文**                                                 |
| ---------------------- | -------------------------------------------------- | ------------------------------------------------------------ |
| **IPv6 RA Guard**      | **拦截伪造路由公告**，只允许合法路由。             | Block a malicious host and permit the router from a legitimate route. |
| **IPv6 DHCPv6 Guard**  | **拦截伪造 DHCP 服务器**的消息。                   | Block reply and advertisement messages from unauthorized DHCP servers and relay agents. |
| **IPv6 ND Inspection** | **检查邻居发现报文**，根据 NDP Snooping 创建绑定。 | Create a binding table that is based on NS and NA messages.  |
| **IPv6 Source Guard**  | **过滤非法源地址**，不在表里的包不让过。           | Filter inbound traffic on Layer 2 switch ports that are not in the IPv6 binding table. |
| **IPv6 Binding Table** | **基础数据库**，汇总所有地址绑定信息。             | Create IPv6 neighbors connected to the device from information sources such as NDP snooping. |

#### **1. RA Guard (路标守卫) —— 针对路由器 (RA)**

- **记法：** 只有保安指定的“主路”才是正路。如果有个坏蛋在路边乱立路牌（伪造 RA 报文）诱导大家走错路，保安直接把牌子拆了。
- **关键词：** **RA** (Router Advertisement), **Legitimate route**。

#### **2. DHCPv6 Guard (快递站守卫) —— 针对服务器**

- **记法：** 小区只允许“顺丰”（合法 DHCP 服务器）发快递。如果有不明人员自称是派件员（伪造 DHCP Reply/Adv），保安直接轰走。
- **关键词：** **DHCP reply/advertisement**, **Unauthorized servers**。

#### **3. ND Inspection (访客登记) —— 针对邻居交换**

- **记法：** 当邻居们互相串门打招呼（NS/NA 报文）时，保安会偷听并记录谁住哪一户。
- **关键词：** **NS and NA messages** (Neighbor Solicitation/Advertisement)。

#### **4. Source Guard (工卡校验) —— 针对所有入站包**

- **记法：** 每个出门的人都要刷脸。如果你刷的脸和保安登记簿（Binding Table）上的对不上，门就不开。
- **关键词：** **Filter inbound traffic**, **Not in the binding table**。

#### **5. Binding Table (保安登记簿) —— 核心数据库**

- **记法：** 这是保安手里最重要的小册子。它是通过偷听 NDP Snooping 等手段汇总出来的“住户通讯录”。
- **关键词：** **Information sources such as NDP snooping**。

| **功能**          | **防范对象**   | **核心协议**        | **关键联想** |
| ----------------- | -------------- | ------------------- | ------------ |
| **RA Guard**      | 假路由器       | ICMPv6 Type 134     | 别乱指路     |
| **DHCPv6 Guard**  | 假 DHCP 服务器 | UDP 547             | 别乱发 IP    |
| **ND Inspection** | 假邻居         | ICMPv6 Type 135/136 | 别乱认领地址 |
| **Source Guard**  | 身份伪造       | IP Layer            | 脸不对不放行 |





这组题目是 MPLS 基础架构术语的进一步细化，主要区分了不同类型的**路由器（Router）\**和\**流量路径**。

------

### **正确对应关系**

| **术语** | **定义特征**                           | **英文原文**                                                 |
| -------- | -------------------------------------- | ------------------------------------------------------------ |
| **LSR**  | **P 路由器**，处于服务商核心。         | routers in the core of the provider network known as P routers |
| **FEC**  | **同路径、同标签**转发的所有流量集合。 | all traffic to be forwarded using the same path and same label |
| **LER**  | **PE 路由器**，连接客户侧。            | routers that connect to the customer routers known as PE routers |
| **LDP**  | 路由器之间**交换标签映射信息**。       | used for exchanging label mapping information between MPLS enabled routers |
| **LSP**  | 流量穿越 MPLS 网络的**路径**。         | path along which the traffic flows across an MPLS network    |

------

### **“名字拆解”记忆法**

这组词里最容易混淆的是 **LSR**、**LER** 和 **LSP**。你可以通过最后一个字母来区分：

#### **1. LSR (Label Switch Router) —— “交换者 (R)”**

- **理解：** 它是核心里的“P 路由器”。它只负责拿到一个标签，换成另一个标签，然后传出去。
- **关键词：** **Core**, **P routers**。

#### **2. LER (Label Edge Router) —— “边缘者 (R)”**

- **理解：** 它是“PE 路由器”。它是 MPLS 网络的边界（Edge），一头连着客户，一头连着 MPLS 核心。
- **关键词：** **Connect to customer**, **PE routers**。

#### **3. LSP (Label Switched Path) —— “路径 (P)”**

- **理解：** 这是一个**虚拟的概念**，不是设备。它是数据包从起点到终点走过的“整条路”。
- **关键词：** **Path**, **Flows across**。

------

### **“逻辑闭环”记忆法**

为了让网络通起来，这几个术语是这样串联的：

1. **LDP (协议)** 像是在“对台词”：路由器们通过它互相交换标签。
2. **FEC (分类)** 像是在“分货”：把要去往同一个地方的流量归为一类。
3. **LSP (路径)** 像是在“铺路”：确定这批货怎么走。
4. **LER (入口)** 像是在“贴标”：PE 路由器给货打上标签。
5. **LSR (中转)** 像是在“换标”：核心 P 路由器快速换标转发。

------

### **快速对比总结表**

| **缩写** | **全称核心词**      | **身份类比** | **对应设备**  |
| -------- | ------------------- | ------------ | ------------- |
| **LSR**  | **Router** (Switch) | 核心中转站   | **P** Router  |
| **LER**  | **Router** (Edge)   | 边界门户     | **PE** Router |
| **LDP**  | **Protocol**        | 沟通语言     | -             |
| **LSP**  | **Path**            | 运输单行线   | -             |
| **FEC**  | **Class**           | 货物品类     | -             |

------

