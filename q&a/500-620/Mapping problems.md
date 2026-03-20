OSPF

**Down**: 初始状态。

**Init**: 收到对方的 Hello，但里面还没看到“我”的名字（单向通信）。

**2-way**: 双方都在对方的 Hello 列表中看到了彼此（双向通信）。**DR/BDR 选举在此完成**。

**ExStart**: 决定谁先发送数据（Router ID 大的为 Master）。

**Exchange**: 交换 LSA 头部信息（类似发“目录”）。

**Loading**: 对比目录后发现自己缺少的 LSA，发送 **LSR** 请求更新。

**Full**: 数据库完全同步（图中未列出，但这是最终目标）。

| **State**    | **Description**                                              |
| ------------ | ------------------------------------------------------------ |
| **Exchange** | Each router compares the DBD (Database Descriptor) packets that were received from the other router. |
| **2-way**    | Routers exchange information with other routers in the multiaccess network. |
| **Loading**  | The neighboring router requests the other routers to send missing entries (using LSRs). |
| **ExStart**  | The network has already elected a DR and a backup BDR (the Master/Slave relationship is also established here). |
| **Init**     | The OSPF router ID of the receiving router was not contained in the hello message. |
| **Down**     | No hellos have been received from a neighbor router.         |



| **DHCP Message Type** | **Description**                                              |
| --------------------- | ------------------------------------------------------------ |
| **DHCPNAK**           | server-to-client communication, **refusing the request** for configuration parameters |
| **DHCPDECLINE**       | client-to-server communication, indicating that the **network address is already in use** |
| **DHCPACK**           | server-to-client communication with configuration parameters, including **committed network address** |
| **DHCPINFORM**        | client-to-server communication, asking for **only local configuration parameters** that the client has already externally configured as an address |

DORA

**DHCPACK vs DHCPNAK**:

- **ACK** (Acknowledge) 是“确认”，即分配成功。
- **NAK** (Negative Acknowledge) 是“否定”，即分配失败或请求的地址无效。

**DHCPDECLINE vs DHCPRELEASE**:

- **DECLINE**: 客户端通过 ARP 检测发现地址冲突了，于是“婉拒”这个地址（Already in use）。
- **RELEASE**: 客户端正常下线，主动“释放”地址。

**DHCPINFORM**:

- 这种情况下客户端**已经有 IP 地址了**（可能是静态配的），它发这个包只是为了问 DHCP 服务器要一下 DNS 或是网关信息。

**Discover (Client $\to$ Server)**:

- **目的**：客户端广播寻找网络中可用的 DHCP 服务器。
- **关键点**：源 IP 为 `0.0.0.0`，目的 IP 为 `255.255.255.255`。

**Offer (Server $\to$ Client)**:

- **目的**：服务器预留一个 IP 并告知客户端。
- **关键点**：包含拟分配的 IP、掩码、租期等。

**Request (Client $\to$ Server)**:

- **目的**：客户端正式请求使用该 IP。
- **关键点**：即使此时已有拟分配地址，这步依然是**广播**，目的是告诉其他潜在的 DHCP 服务器：“我已经选好一家了，你们可以把预留给我的地址收回去了。”

**Acknowledgment (Server $\to$ Client)**:

- **目的**：服务器确认分配，交互完成。



| **左侧角色 (Role)** | **右侧描述 (Description)**                                   | **是否匹配**                               |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------ |
| **host**            | Receives router advertisements from valid routers, and no router solicitation are received. | **正确** (主机只收 RA，不发也不收 RS)      |
| **router**          | Receives router solicitation and sends router advertisements. | **正确** (路由器的基本职能)                |
| **monitor**         | Receives valid and rogue router advertisements and all router solicitation. | **正确** (监控模式会分析所有合法/非法报文) |
| **switch**          | Received router advertisements are trusted and are flooded to synchronize states. | **正确** (交换机作为信任中继或同步节点)    |



### **配置解析 (Exhibit Analysis)**

1. **全局配置**:

   - `aaa authentication login default none`: 默认登录列表为“无认证”（即不需要用户名/密码）。
   - `aaa authentication login telnet local`: 定义了一个名为 **"telnet"** 的自定义列表，强制使用 **本地数据库 (local)** 认证。
   - `username cisco password 0 ocsic`: 本地数据库中存有用户名 `cisco` 和密码 `ocsic`。

2. **Line vty 0**:

   - 配置了 `login authentication telnet`。这表示 **vty 0** 强制使用自定义列表 "telnet"，即要求输入 **本地用户名和密码**。

3. **Line vty 1**:

   - **没有**配置任何 `login authentication` 命令。根据 AAA 逻辑，它将回退到使用 **default** 列表。由于 default 列表被设为 `none`，所以它**不需要用户名和密码**。

   

4. | **登录位置** | **用户名 (Username)** | **密码 (Password)** |
   | ------------ | --------------------- | ------------------- |
   | **vty0**     | **cisco**             | **ocsic**           |
   | **vty1**     | **no username**       | **no password**     |





| **术语**                          | **描述 (Description)**                                       |
| --------------------------------- | ------------------------------------------------------------ |
| **multiprotocol BGP**             | **propagates VPN reachability information** (传播 VPN 可达性信息/VPNv4 路由)。 |
| **Resource Reservation Protocol** | **distributes labels for traffic engineering** (为流量工程分配标签，即 RSVP-TE)。 |
| **route distinguisher**           | **uniquely identifies a customer prefix** (使客户前缀具有唯一性，解决 IP 重叠问题)。 |
| **route target**                  | **controls the import/export of customer prefixes** (控制客户前缀的导入和导出逻辑)。 |



| **缩写** | **描述 (Description)**                                       |
| -------- | ------------------------------------------------------------ |
| **P**    | **device that forwards traffic based on labels** (P 路由器仅根据外层标签进行交换，不看 IP)。 |
| **LSP**  | **path that the labeled packet takes** (Label Switched Path，即标签交换路径)。 |
| **CE**   | **device that is unaware of MPLS labeling** (Customer Edge，客户边缘设备，通常只运行普通 IP 路由)。 |
| **PE**   | **device that removes and adds the MPLS labeling** (Provider Edge，负责标签的压入 Push 和弹出 Pop)。 |



MPLS

| **标签/技术**                       | **描述 (Description)**                                       |
| ----------------------------------- | ------------------------------------------------------------ |
| **entropy label**                   | **provides ways of improving load balancing by eliminating the need for DPI at transit LSRs** (通过在标签栈中引入熵标签，避免中间路由器进行深层包检测以实现均衡)。 |
| **implicit null label**             | **LSR receives an MPLS header with the label set to 3** (隐式空标签，对应标签值 3，用于触发 PHP 次末跳弹出)。 |
| **explicit null label**             | **packet is encapsulated in MPLS with the option of copying the IP precedence to EXP bits** (显式空标签，对应标签值 0，用于保留 QoS/EXP 位信息直到最后一跳)。 |
| **inbound label binding filtering** | **controls the amount of memory used to store LDP label bindings advertised by other devices** (通过过滤入站标签绑定来节省内存)。 |

LSR

| **角色 (Role)**               | **描述 (Description)**                                       |
| ----------------------------- | ------------------------------------------------------------ |
| **Label Switch Router (LSR)** | **reads the labels and forwards the packet based on the labels** (读取标签并根据标签转发，即中间节点的标签交换)。 |
| **Label Switch Router (LSR)** | **performs penultimate hop popping** (执行次末跳弹出 PHP，通常由倒数第二台 LSR 完成)。 |
| **Label Edge Router (LER)**   | **assigns labels to unlabeled packets** (为未标记的数据包分配标签，即在入站时执行 Push 操作)。 |
| **Label Edge Router (LER)**   | **handles traffic between multiple VPNs** (处理多个 VPN 之间的流量，LER 作为 PE 设备连接不同的 VRF)。 |



