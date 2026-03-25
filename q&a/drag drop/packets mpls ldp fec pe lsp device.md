| **平面**       | **简记口诀**           | **典型协议/动作**                |
| -------------- | ---------------------- | -------------------------------- |
| **Data**       | forward                | 用户上网流量、二层交换、三层转发 |
| **Control**    | routing                | OSPF, BGP, STP, ARP              |
| **Management** | manage                 | SSH, Telnet, SNMP, HTTPS         |
| **Services**   | **Special processing** | NAT, IPsec, QoS, Firewall        |

### **1. 数据平面 (Data Plane) —— “快递员”**

- **角色：** 只负责“跑腿”搬运货物（数据包）。
- **特点：** 它们不思考，只根据手里的地址表（转发表）把包裹从 A 点送到 B 点。
- **关键词：** **User-generated**（用户产生的流量）、**Forwarding**（转发）。
- **联想：** 就像高速公路上的车辆，它们只是过客。

### **2. 控制平面 (Control Plane) —— “导航仪/大脑”**

- **角色：** 负责绘制地图，决定走哪条路最快。
- **特点：** 它生成网络运行所需的“知识”。比如 OSPF、BGP 协议在互相通气，商量怎么建立路由表。
- **关键词：** **Creation of the network**（构建网络拓扑）、**Protocol**（协议）。
- **联想：** 就像交通指挥中心，虽然不运货，但没有它，快递员就迷路了。

### **3. 管理平面 (Management Plane) —— “管理员/老板”**

- **角色：** 负责对设备进行“远程操控”。
- **特点：** 这是你（管理员）登录设备的方式。比如你通过 SSH 进去改个密码、查看一下运行状态。
- **关键词：** **Operate**（操作）、**Manage**（管理）。
- **联想：** 就像公司的 HR 或老板，他们不搬砖，也不画地图，但他们决定谁能进公司工作。

### **4. 服务平面 (Services Plane) —— “特殊处理/VIP通道”**

- **角色：** 对特定货物进行“深加工”。
- **特点：** 普通转发不够，还需要额外的处理（如防火墙过滤、加密、QoS 优先级排队）。
- **关键词：** **Higher priority**（高优先级）、**Special processing**。
- **联想：** 就像快递里的“生鲜冷链”或“武装押运”，需要特殊对待。



| **缩写** | **类比**         | **核心考点**                        |
| -------- | ---------------- | ----------------------------------- |
| **FEC**  | 货物的**目的地** | 凡是发往同一网段的包都算一个 FEC    |
| **LDP**  | 调度员的**对话** | 路由器互传“标签 <-> 路由”的对应关系 |
| **LSP**  | 固定的**航线**   | 标签包走过的单向隧道                |
| **PE**   | 码头的**海关**   | 连接客户网络和运营商核心的交界点    |

## Gemini said

MPLS 的这些概念可以用**“火车物流系统”**来类比记忆。

------

### **正确对应关系**

| 术语     | 定义 (中文翻译)                            | 定义 (英文原文)                                              |
| -------- | ------------------------------------------ | ------------------------------------------------------------ |
| **MPLS** | **与链路层无关的数据传输机制**             | data-carrying mechanism that is independent of any data link layer protocol |
| **LDP**  | **路由器之间交换标签映射信息的机制**       | mechanism by which two routers exchange label mapping information |
| **FEC**  | **具有相同特征、绑定到同一标签的一组报文** | set of packets with similar characteristics that might be bound to the same MPLS label |
| **PE**   | **作为 MPLS 域入口或出口的路由器**         | router that functions as the ingress and/or egress router to the MPLS domain |
| **LSP**  | **由 LDP 或 BGP 等信令协议定义的穿越路径** | route through an MPLS network, defined by a signaling protocol such as LDP or BGP |

------

### **“火车物流”记忆大法**

#### **1. MPLS (多协议标签交换) —— “集装箱运输”**

- **记法：** 无论下面是公路（以太网）还是铁路（帧中继），我只管在货物上打个“标签”。它让传输独立于底层协议。
- **关键词：** **Independent** (独立)、**Data-carrying** (承载数据)。

#### **2. FEC (转发等价类) —— “货物的类别”**

- **记法：** 去往同一个目的地（比如上海）的所有包裹，都被归为一类，贴上同样的“上海”标签。
- **关键词：** **Similar characteristics** (相似特征)、**Same label** (相同标签)。

#### **3. LSP (标签交换路径) —— “预定铁轨线”**

- **记法：** 火车从北京到上海预先铺好的铁轨。它是从起点到终点的**一整条路**。
- **关键词：** **Route through** (穿越路径)、**Defined by signaling** (信令定义的)。

#### **4. LDP (标签分发协议) —— “调度员之间的对讲机”**

- **记法：** A 站调度员告诉 B 站：“发往上海的货，请贴上 100 号标签”。这是路由器之间**交换情报**的过程。
- **关键词：** **Exchange label mapping** (交换标签映射)。

#### **5. PE (边缘路由器) —— “货运总站”**

- **记法：** 货物进入 MPLS 网络的“大门”。在这里，普通包裹被打包进“标签集装箱”（Ingress），或者被拆箱还原（Egress）。
- **关键词：** **Ingress/Egress** (入口/出口)、**Edge** (边缘)。



| **角色**                | **定义特征**                                 | **英文原文**                                                 |
| ----------------------- | -------------------------------------------- | ------------------------------------------------------------ |
| **Provider (P) device** | **运营商核心**，只负责快速交换标签包         | device in the **core** of the provider network that switches MPLS packets |
| **PE device**           | **运营商边缘**，负责给数据打上/撕掉 VPN 标签 | device that **attaches and detaches** the VPN labels to the packets... |
| **Customer (C) device** | **企业内网设备**，只管连接内部其他设备       | device in the **enterprise network** that connects to other customer devices |
| **CE device**           | **客户侧边缘**，负责连接运营商网络           | device at the **edge of the enterprise network** that connects to the SP network |

#### **1. C (Customer) —— “公司内的小兵”**

- **位置：** 深陷在公司内网里。
- **特点：** 它**完全不知道** MPLS 或运营商的存在。它只跟自己的同事（其他 C 设备）说话。
- **关键词：** **Enterprise network**, **other customer devices**。

#### **2. CE (Customer Edge) —— “公司传达室”**

- **位置：** 公司的边界，大门口。
- **特点：** 它是公司与外界联系的唯一窗口。
- **关键词：** **Edge of enterprise**, **connects to SP**（连接服务提供商）。

#### **3. PE (Provider Edge) —— “海关/快递分拣中心”**

- **位置：** 运营商的门口。
- **特点：** **最忙、活最全**。它是真正干“技术活”的——把普通包裹打成“MPLS 标签件”（Attach），或者拆开（Detach）。
- **关键词：** **Attaches and detaches VPN labels**。

#### **4. P (Provider) —— “高速公路收费站”**

- **位置：** 运营商网络的核心深处。
- **特点：** 它们只负责**快**。它们不看包里是什么，只根据标签无脑转发，效率极高。
- **关键词：** **Core**, **switches MPLS packets**。