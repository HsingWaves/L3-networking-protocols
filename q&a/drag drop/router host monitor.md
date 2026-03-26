这道题目考察的是 **IPv6 First-Hop Security (FHS)** 中的 **IPv6 Guard**（也叫 RA Guard）。

它通过在交换机端口上配置不同的角色（Role），来过滤和监控 RA（路由器通告）和 RS（路由器请求）报文，防止伪造路由器攻击。

### 1. 正确的对应关系

| **角色 (Role)** | **描述 (Description)**                                       | **逻辑理解**                                                 |
| --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **host**        | **Receives router advertisements from valid routers, and no router solicitation are received.** | **主机端**：只接收合法的 RA（下行），不应该收到 RS（因为主机不找主机）。 |
| **router**      | **Receives router solicitation and sends router advertisements.** | **服务端**：接收主机的请求（RS），并发送自己的通告（RA）。   |
| **monitor**     | **Receives valid and rogue router advertisements and all router solicitation.** | **监控端**：通常用于流量分析或 IDS，它会接收所有的 RA（合法或伪造）和 RS。 |
| **switch**      | **Received router advertisements are trusted and are flooded to synchronize states.** | **交换端**：在这种 FHS 环境下，它负责转发经过验证的、可信的报文。 |

------

### 2. 深度解析：RA Guard 的工作原理

在 CCNP 考试中，**RA Guard** 是保护网络不被非法路由器劫持的关键技术。它的配置核心在于区分 **Trusted (可信)** 和 **Untrusted (不可信)** 端口：

- **配置在 Host 端口**：如果这个接口收到了 RA（通告），交换机会直接丢弃。因为 Host 不应该发 RA，只有伪造的网关才会发。
- **配置在 Router 端口**：允许 RA 进入，因为它连接的是真实的合法路由器。

------

### 3. 记忆技巧：看报文的“方向”

- **Host = "Consumer" (消费者)**：它消费 RA，产生 RS。所以它所在的端口如果不小心收到了来自别人的 RS，那肯定不对劲。
- **Router = "Provider" (提供者)**：它产生 RA，消费 RS。
- **Monitor = "Vigilant" (警惕者)**：作为旁路设备，它要看全局，所以好的坏的都要。

------

### 考试考点提醒

在 Cisco 命令行中，配置步骤通常是：

1. 创建一个策略：`ipv6 nd inspection policy POLICY_NAME`
2. 在接口下应用：`ipv6 nd guard attach policy POLICY_NAME` (或 `device-role host/router`)

在计算机网络，尤其是你正在复习的 **IPv6 (NDP)** 和 **DHCP** 协议中，**Solicitation** 是一个核心动作词。

你可以把它最直观地理解为：**“主动打听”** 或 **“全网呼叫”**。

------

### 1. 词源与本质

在英语中，*Solicit* 的意思是“请求、恳求、征求”。

在网络协议里，它代表一种**主动触发的请求机制**。通常是因为一个节点（Host 或 Router）**不知道**某些信息，所以发报文去“打听”。

它通常和 **Advertisement (通告/广告)** 成对出现：

- **Solicitation**: “请问有人知道...吗？”（问）
- **Advertisement**: “我知道，我是... / 信息在这里。”（答）

------

### 2. IPv6 中的两大 Solicitation

在 CCNP 的 IPv6 邻居发现协议 (NDP) 中，你会反复看到这两个词：

#### A. Router Solicitation (RS) —— “找网关”

- **情景**：一台主机（比如你的电脑）刚刚接入网络，它没有 IPv6 地址，也不知道网关是谁。
- **动作**：它等不及路由器的定期通告了，直接发一个 RS 报文（Type 133）。
- **翻译**：*“喂！这地方有路由器吗？快给我发个前缀，我要生成地址！”*

#### B. Neighbor Solicitation (NS) —— “找邻居”

- **情景**：主机已经知道邻居的 IPv6 地址，但不知道它的 **MAC 地址**（类似 IPv4 的 ARP Request）。
- **动作**：发送 NS 报文（Type 135）。
- **翻译**：*“谁是 `2001:DB8::1`？听到请回答你的 MAC 地址。”*
- **另一个用途 (DAD)**：在自己用一个地址前，先发个 NS 问问：*“这地址有人用吗？”*

------

### 3. DHCP 里的 Solicitation (DHCPv6)

在 **DHCPv6** 中，动作名称发生了变化。IPv4 叫 *Discover*，而 IPv6 叫 **Solicit**。

- **Solicit (C → S)**：客户端寻找可用的 DHCPv6 服务器。
- **Advertise (S → C)**：服务器回应：“我在这，我可以给你分配参数。”

------

### 4. 记忆总结表

为了方便你考试，记住这个逻辑链条：

| **动作名称**              | **本质**      | **谁发起的？** | **目的**                            |
| ------------------------- | ------------- | -------------- | ----------------------------------- |
| **RS** (Router Solicit)   | **求救/咨询** | Host           | 找路由器 (Default Gateway)          |
| **NS** (Neighbor Solicit) | **打听/质询** | Host/Router    | 找邻居的 MAC 或做重复地址检测 (DAD) |
| **DHCP Solicit**          | **搜寻**      | Client         | 找 DHCP 服务器                      |

------

### 💡 一个形象的比喻

想象你进了一个黑漆漆的屋子：

- 你大喊一声：“有人吗？” —— 这就是 **Solicitation**。
- 屋里有人回了一句：“我在角落里。” —— 这就是 **Advertisement**。

如果你不喊（Solicitation），你就只能等屋里的人每隔 10 分钟自言自语一次（Periodic Advertisement），那效率就太低了。

