 OSPF 在 VRF 环境下的一个**防环机制**。

### 1. 现象描述：消失的 LSA

工程师配置了 VRF-lite，但发现 Site A 的路由器收不到 **LSA 3, 4, 5, 7**。这意味着除了本区域（Area 1）内的路由，其他外部路由和区域间路由都没能进入路由表。

### 2. 核心考点：OSPF 下的“DN 位”防环

在标准的 MPLS VPN 架构中，PE 路由器会将路由从 BGP 注入到 OSPF。为了防止路由在 PE 之间形成环路，OSPF 会在 LSA 中设置一个特殊的标志位——**DN 位 (Down Bit)**。

- **设计初衷：** 当一台 PE 路由器收到一条带 DN 位的 LSA 时，它会认为：“这条路由是从 MPLS 骨干网传出来的，我不能再把它传回骨干网，也不能把它装进路由表。”
- **本题的坑：** 这里的 Head Office 和 Site A 路由器都运行了 `router ospf 1 vrf abc`。在 OSPF 看来，只要开启了 VRF，这台路由器就可能扮演“PE”的角色。
- 因此，Site A 路由器收到了带 DN 位的 LSA，触发了防环机制，由于它认为自己是 PE，于是**拒绝将这些 LSA 放入路由表**。

------

### 3. 解决方案：`capability vrf-lite`

我们要告诉 Site A 和 Head Office 路由器：“我只是个普通的客户路由器（CE），我并不是连接骨干网的 PE，请关掉那个烦人的 DN 位检测。”

这条命令就是：`capability vrf-lite`。

- **在哪里配？** 在 OSPF 进程下。
- **谁需要配？** **Head Office 和 Site A** 这两台运行 VRF 的客户路由器。