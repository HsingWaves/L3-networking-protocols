| **NDP Message**                 | **ICMPv6 Type** | **Purpose**                                                  |
| ------------------------------- | --------------- | ------------------------------------------------------------ |
| **Router Solicitation (RS)**    | **Type 133**    | Sent by hosts to locate routers on the link.                 |
| **Router Advertisement (RA)**   | **Type 134**    | Sent by routers to advertise their presence and link parameters. |
| **Neighbor Solicitation (NS)**  | **Type 135**    | Used for address resolution (like ARP) and Duplicate Address Detection (DAD). |
| **Neighbor Advertisement (NA)** | **Type 136**    | Used to respond to an NS or to notify neighbors of a link-layer address change. |
| **Redirect Message**            | **Type 137**    | Used by routers to inform a host of a better first-hop router. |

NDP 协议本质上是“请求”与“响应”的对话。你可以把它们分成两组：**路由器组**和**邻居组**。

- **路由器组 (Router Group):** 先发生（主机上线先找网关）。
  - **133 (RS)**: Router **S**olicitation (请求)
  - **134 (RA)**: Router **A**dvertisement (通告)
- **邻居组 (Neighbor Group):** 后发生（类似 IPv4 的 ARP）。
  - **135 (NS)**: Neighbor **S**olicitation (请求)
  - **136 (NA)**: Neighbor **A**dvertisement (通告)
- **重定向 (Redirect):**
  - **137**: Redirect Message (告诉主机有更好的路径)

------

### 2. 首字母顺口溜法

按照 133 到 137 的顺序，提取首字母：**R-R-N-N-R**。

- **133/134**: **R**outer 相关。
- **135/136**: **N**eighbor 相关。
- **137**: 剩下的那个就是 **R**edirect。



| **健康分数 (Health Score)** | **颜色 (Color)**  | **状态含义**            |
| --------------------------- | ----------------- | ----------------------- |
| **8 到 10 (8-10)**          | **Green (绿色)**  | 健康状况良好            |
| **4 到 7 (4-7)**            | **Orange (橙色)** | 存在中度问题 (Warning)  |
| **1 到 3 (1-3)**            | **Red (红色)**    | 存在严重问题 (Critical) |
| **0**                       | **Gray (灰色)**   | 无数据 (No Data)        |





| **步骤**   | **描述 (Description)**                           | **对应命令示例**                   |
| ---------- | ------------------------------------------------ | ---------------------------------- |
| **STEP 1** | **Create the BGP Routing Process**               | `router bgp 65001`                 |
| **STEP 2** | **Identify the BGP Neighbor's IP and AS Number** | `neighbor 1.1.1.1 remote-as 65002` |
| **STEP 3** | **Initialize the address-family (AFI/SAFI)**     | `address-family ipv4 unicast`      |
| **STEP 4** | **Activate the address-family for the neighbor** | `neighbor 1.1.1.1 activate`        |

你可以把这个过程想象成**“进屋 -> 找人 -> 谈事”**：

1. **进屋 (Step 1)**：先输入 `router bgp` 进入 BGP 进程的大门。
2. **找人 (Step 2)**：告诉路由器你的邻居是谁 (`neighbor remote-as`)，建立基础联系。
3. **确定话题 (Step 3)**：进入具体的地址族 (`address-family`)。BGP 可以传 IPv4, IPv6, VPNv4 等，你得告诉它现在要谈哪种业务。
4. **开始聊天 (Step 4)**：最后一步必须激活 (`activate`)，否则这个邻居虽然认识，但不会在这个特定的话题（地址族）下交换路由。





| **步骤**   | **描述 (Description)**                                | **核心动作**                                                 |
| ---------- | ----------------------------------------------------- | ------------------------------------------------------------ |
| **STEP 1** | **Download the Cisco IOS image to the TFTP Server**   | **准备阶段**：先确保 TFTP 服务器上有你需要的目标镜像文件。   |
| **STEP 2** | **Copy the IOS image in the file system**             | **传输阶段**：使用 `copy tftp: flash:` 将镜像从服务器下载到路由器的闪存中。 |
| **STEP 3** | **Verify the configuration register & boot variable** | **检查阶段**：确认寄存器值（如 `0x2102`）并设置 `boot system` 变量指向新镜像。 |
| **STEP 4** | **Save the configuration & Reload the router.**       | **生效阶段**：`write` 保存配置后 `reload`，路由器才会加载新镜像。 |

**第一步在外 (Step 1)**：镜像最初肯定不在路由器里，是在外部的 **TFTP Server** 上。

**第二步进内 (Step 2)**：把镜像 **Copy** 进路由器的文件系统（Flash）。

**第三步指路 (Step 3)**：文件进来了，得告诉路由器下次启动走哪条路（**Boot Variable**）。

**最后重启 (Step 4)**：万事俱备，**Reload** 见真章。