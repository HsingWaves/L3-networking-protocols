------

# 🚀 DMVPN 全面总结

------

# 一、DMVPN 是什么？

**Dynamic Multipoint VPN**

一句话：

> 用 mGRE + NHRP + IPsec 构建可动态建立的 Hub-and-Spoke VPN

------

# 二、DMVPN 三大核心组件

## 1️⃣ mGRE（Multipoint GRE）

作用：

- 允许一个接口支持多个对端
- 不需要为每个 spoke 建单独 tunnel

传统 GRE：

```
1 tunnel = 1 peer
```

mGRE：

```
1 tunnel = many peers
```

------

## 2️⃣ NHRP（Next Hop Resolution Protocol）

作用：

```
Overlay IP  →  NBMA 真实地址
```

就像：

```
ARP = IP → MAC
NHRP = Tunnel IP → Public IP
```

没有 NHRP，spoke-to-spoke 永远不知道对方公网地址。

------

## 3️⃣ IPsec（可选但几乎必用）

作用：

- 加密 GRE 流量
- 常用 transport mode

DMVPN 标准写法：

```bash
interface tunnel0
 tunnel mode gre multipoint
 tunnel protection ipsec profile DMVPN
```

------

# 三、DMVPN 三个阶段（考试必考）

------

## 🔹 Phase 1

```
Spoke → Hub → Spoke
```

特点：

- 所有流量必须经过 Hub
- 简单
- 不支持 spoke-to-spoke 直连

------

## 🔹 Phase 2

```
Spoke → Spoke（直连）
```

特点：

- 支持动态 shortcut
- 需要：
  - routing protocol 允许 next-hop 不改
  - NHRP redirect

------

## 🔹 Phase 3 ⭐（主流）

改进点：

- 支持大规模
- 使用：
  - NHRP redirect
  - NHRP shortcut

工作流程：

```
Spoke A → Hub
Hub 告诉 A：直接去找 B
A 建立直连
```

------

# 四、DMVPN 工作流程（Phase 3）

1. Spoke 注册到 Hub
2. Hub 维护 NHRP mapping
3. Spoke 访问其他 Spoke
4. Hub 返回 redirect
5. 建立 spoke-to-spoke IPsec SA

------

# 五、OSPF 在 DMVPN 中

DMVPN Tunnel 默认：

```
OSPF network type = broadcast
```

所以：

会选 DR / BDR

最佳实践：

```
Hub:
 ip ospf priority 255

Spoke:
 ip ospf priority 0
```

------

# 六、常见考试坑

| 坑点                  | 正解                |
| --------------------- | ------------------- |
| NBMA 地址解析靠什么？ | NHRP                |
| IPsec 用什么模式？    | transport           |
| Hub 如何成为 DR？     | 提高 OSPF priority  |
| spoke-to-spoke 不起？ | NHRP 问题           |
| Phase 3 必须什么？    | redirect + shortcut |

------

# 七、DMVPN 结构图

```
              HUB
        10.0.0.1
         /   |   \
        /    |    \
   SpokeA  SpokeB  SpokeC
```

Overlay：

```
10.0.0.0/24
```

Underlay：

```
192.x.x.x 公网
```

------

# 八、核心记忆口诀

```
DMVPN = mGRE + NHRP + IPsec

mGRE = 多点隧道
NHRP = 地址解析
IPsec = 加密
```

------

# 九、DMVPN vs 传统 GRE

| 对比       | GRE       | DMVPN         |
| ---------- | --------- | ------------- |
| 隧道数量   | N*(N-1)/2 | 只需要 Hub    |
| 可扩展性   | 差        | 强            |
| 配置复杂度 | 高        | 低            |
| 支持直连   | 不支持    | 支持（P2/P3） |

------

# 十、你现在应该掌握的 5 个关键点

1. DMVPN 三大组件
2. Phase 1/2/3 区别
3. NHRP 的作用
4. OSPF 在 DMVPN 的配置
5. IPsec transport 模式

------



------

# 🧠 一页总览表

| 特性           | Phase 1       | Phase 2          | Phase 3 ⭐（主流）   |
| -------------- | ------------- | ---------------- | ------------------- |
| 拓扑           | Hub-and-Spoke | Hub + 动态 Spoke | Hub + 智能 Shortcut |
| Spoke-to-Spoke | ❌ 不支持      | ✅ 支持           | ✅ 支持（优化版）    |
| 流量路径       | 全部走 Hub    | 直连             | 直连                |
| 路由 Next-hop  | 指向 Hub      | 保持真实         | 保持真实            |
| NHRP Redirect  | ❌             | ❌                | ✅ 必须              |
| NHRP Shortcut  | ❌             | ❌                | ✅ 必须              |
| 可扩展性       | 小规模        | 中等             | ⭐ 大规模最佳        |
| 考试频率       | 低            | 中               | ⭐ 最高              |

------

# 🚀 Phase 1

## 📌 流量路径

```
Spoke A  →  Hub  →  Spoke B
```

所有流量必须经过 Hub。

------

## 📌 特点

- 简单
- 不支持直连
- 适合小型网络

------

## 📌 典型场景图

```
        HUB
       /   \
      /     \
  SpokeA   SpokeB
```

无 spoke-to-spoke 隧道。

------

# 🚀 Phase 2

## 📌 流量路径

```
Spoke A  →  Spoke B（直连）
```

但前提：

- 路由协议必须保持 next-hop 不改
- Hub 充当 NHRP server

------

## 📌 特点

- 支持直连
- 不支持大规模扩展（路由复杂）
- 无 redirect 机制

------

## 📌 直连建立流程

```
1. Spoke A 查路由
2. 发现 next-hop 是 Spoke B
3. 通过 NHRP 查询 NBMA
4. 建立直连隧道
```

------

# 🚀 Phase 3 ⭐（主流架构）

最重要。

------

## 📌 流量路径（初始）

```
Spoke A → Hub → Spoke B
```

Hub 发现可以 shortcut。

------

## 📌 Redirect 机制

Hub 返回：

```
NHRP Redirect:
“直接去找 Spoke B”
```

------

## 📌 Shortcut 建立

Spoke A：

```
发送 NHRP Resolution
建立 IPsec SA
形成直连
```

------

## 📌 完整流程图

```
① Spoke A → Hub
② Hub → A 发送 redirect
③ A 发送 NHRP resolution
④ A ↔ B 建立直连 IPsec
⑤ 后续流量直连
```

------

# 🎯 Phase 2 vs Phase 3 核心区别

| 项目            | Phase 2    | Phase 3  |
| --------------- | ---------- | -------- |
| 谁触发直连      | Spoke 自己 | Hub 通知 |
| 是否有 redirect | ❌          | ✅        |
| 是否支持大规模  | 不好       | ⭐ 非常好 |
| 企业推荐        | 较少       | ⭐ 主流   |

------

# 🔥 OSPF 在 DMVPN

默认 Tunnel 是 broadcast：

必须控制 DR：

```
Hub:
 ip ospf priority 255

Spoke:
 ip ospf priority 0
```

------

# 🔥 IPsec 模式

DMVPN 使用：

```
mode transport
```

因为：

GRE 已经封装，IPsec 只需保护外层。

------

# 🎯 一句话记忆口诀

```
Phase 1 = 只走 Hub
Phase 2 = 直连（自己发现）
Phase 3 = Hub 告诉你直连
```

------

# 🧩 DMVPN 三组件再复习一次

```
mGRE  → 多点隧道
NHRP  → 地址解析
IPsec → 加密
```

------

# 🧠 考试高频问法

- NBMA 地址解析？ → NHRP
- Phase 3 需要什么？ → redirect + shortcut
- Hub 如何成为 DR？ → OSPF priority 高
- IPsec 用什么模式？ → transport
- 谁做 label？ → DMVPN 不涉及 MPLS label

------



```
Hub Tunnel IP:    10.0.0.1
Spoke1 Tunnel IP: 10.0.0.2
Spoke2 Tunnel IP: 10.0.0.3
Underlay: 公网接口
```

------

# 🚀 一、基础结构（所有 Phase 都一样）

------

# 🔹 HUB 基础配置

```bash
interface Tunnel0
 ip address 10.0.0.1 255.255.255.0
 no ip redirects
 ip nhrp authentication DMVPN
 ip nhrp map multicast dynamic
 ip nhrp network-id 1
 tunnel source GigabitEthernet0/0
 tunnel mode gre multipoint
```

------

# 🔹 SPOKE 基础配置

```bash
interface Tunnel0
 ip address 10.0.0.2 255.255.255.0
 no ip redirects
 ip nhrp authentication DMVPN
 ip nhrp network-id 1
 ip nhrp map 10.0.0.1 <HUB_NBMA_IP>
 ip nhrp map multicast <HUB_NBMA_IP>
 ip nhrp nhs 10.0.0.1
 tunnel source GigabitEthernet0/0
 tunnel mode gre multipoint
```

------

# 🔐 IPsec 配置（所有 Phase 通用）

```bash
crypto isakmp policy 10
 encr aes
 hash sha
 authentication pre-share
 group 2

crypto isakmp key CISCO address 0.0.0.0

crypto ipsec transform-set DMVPN esp-aes esp-sha-hmac
 mode transport

crypto ipsec profile DMVPN-PROFILE
 set transform-set DMVPN

interface Tunnel0
 tunnel protection ipsec profile DMVPN-PROFILE
```

⚠ 注意：**mode transport 必须**

------

# 🚀 二、Phase 1 配置

特点：不支持 spoke-to-spoke

------

## HUB

无需特殊 NHRP 命令

------

## SPOKE

保持默认即可

------

## 路由协议（例如 OSPF）

Hub：

```bash
router ospf 1
 network 10.0.0.0 0.0.0.255 area 0

interface Tunnel0
 ip ospf network broadcast
 ip ospf priority 255
```

Spoke：

```bash
router ospf 1
 network 10.0.0.0 0.0.0.255 area 0

interface Tunnel0
 ip ospf priority 0
```

------

# 🚀 三、Phase 2 配置

支持 spoke-to-spoke，但无 redirect

------

## 关键点：

Hub 不改 next-hop

如果用 EIGRP：

```bash
router eigrp 100
 no ip next-hop-self
```

（BGP 场景更常见）

------

无需 redirect 命令。

------

# 🚀 四、Phase 3 ⭐（主流）

这是考试最常考。

------

## 🔹 HUB 关键命令

```bash
interface Tunnel0
 ip nhrp redirect
```

------

## 🔹 SPOKE 关键命令

```bash
interface Tunnel0
 ip nhrp shortcut
```

------

# 📌 Phase 3 完整工作流

```
1️⃣ Spoke A 发流量 → Hub
2️⃣ Hub 发现可以 shortcut
3️⃣ Hub 发 NHRP redirect
4️⃣ Spoke A 发送 NHRP resolution
5️⃣ A ↔ B 建立直连 IPsec
```

------

# 🎯 最终结构对比

| Phase | Hub 命令         | Spoke 命令       |
| ----- | ---------------- | ---------------- |
| 1     | 无特殊           | 无特殊           |
| 2     | 无特殊           | 无特殊           |
| 3     | ip nhrp redirect | ip nhrp shortcut |

------

# 🔥 高频考试陷阱

1. IPsec 必须 transport mode
2. NHRP 用于 NBMA 解析
3. OSPF Hub 要 priority 高
4. Spoke priority 0
5. crypto key 推荐 0.0.0.0

------

# 🧠 一句话记忆

```
Phase 1 = Hub-only
Phase 2 = Direct
Phase 3 = Redirect + Shortcut
```

------

