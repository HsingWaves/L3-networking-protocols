

------

# 1️⃣ DMVPN（Dynamic Multipoint VPN）

**DMVPN = 动态多点 VPN**

作用：

```
一个 Hub
多个 Spoke
通过 Internet 建立 VPN
```

传统 VPN：

```
Spoke1 → Hub → Spoke2
```

DMVPN 的目标：

```
Spoke1 → Spoke2
```

直接通信，不绕 Hub。

------

# 2️⃣ Hub 和 Spoke

这是 **Hub-and-Spoke 架构**

```
        Spoke1
           \
            \
Spoke2 ---- HUB ---- Spoke3
            /
           /
        Spoke4
```

### Hub

中心节点

特点：

- 固定公网 IP
- 所有 spoke 都先连接 hub

------

### Spoke

分支站点

特点：

- 动态 IP 也可以
- 先连接 hub，再建立 spoke-to-spoke

------

# 3️⃣ GRE Tunnel

DMVPN 的基础技术是：

```
mGRE (Multipoint GRE)
```

普通 GRE：

```
Tunnel → 只能 point-to-point
```

mGRE：

```
Tunnel → 可以多个 endpoint
```

示例：

```
Hub tunnel0
 destination dynamic
```

这就是 **multipoint GRE**。

------

# 4️⃣ NHRP（Next Hop Resolution Protocol）

DMVPN 最核心协议。

作用：

```
把 tunnel IP → 解析成 NBMA IP
```

类似：

```
ARP for DMVPN
```

对比：

| 协议 | 功能                  |
| ---- | --------------------- |
| ARP  | IP → MAC              |
| NHRP | Tunnel IP → Public IP |

例子：

```
Tunnel IP      Public IP
10.0.0.2  →   203.0.113.2
```

------

# 5️⃣ NBMA

NBMA = **Non-Broadcast Multi-Access**

意思：

```
不支持广播的网络
```

Internet 就是 NBMA。

因此需要：

```
NHRP
```

去解析地址。

------

# 6️⃣ NHRP NHS

NHS = **Next Hop Server**

就是：

```
Hub
```

Spoke 要注册到 Hub：

```
ip nhrp nhs 10.0.0.1
```

意思：

```
Hub 是我的 NHRP Server
```

------

# 7️⃣ NHRP Mapping

作用：

```
Tunnel IP → NBMA IP
```

例如：

```
ip nhrp map 10.0.0.1 203.0.113.1
```

意思：

```
Hub tunnel IP → Hub public IP
```

------

# 8️⃣ NHRP Redirect

作用：

```
Hub 告诉 Spoke：

“不要走我，直接走对方”
```

流程：

```
Spoke1 → Hub → Spoke2
```

Hub 说：

```
Use direct tunnel
```

配置：

```
ip nhrp redirect
```

------

# 9️⃣ NHRP Shortcut

作用：

```
允许 Spoke 建立 direct tunnel
```

配置：

```
ip nhrp shortcut
```

如果没有这个：

```
Spoke 不能建立 shortcut
```

------

# 🔟 NHRP Resolution

当 spoke 要找另一个 spoke 时：

发送：

```
NHRP Resolution Request
```

Hub 回复：

```
NBMA address
```

然后建立：

```
direct GRE tunnel
```

------

# 1️⃣1️⃣ 完整 DMVPN Phase3 流程

```
1  Spoke1 → Hub
2  Hub → redirect
3  Spoke1 → NHRP resolution
4  Hub → NBMA address
5  Spoke1 → Spoke2 direct tunnel
```

------

# 1️⃣2️⃣ 一张总结图

```
                (NHRP Server)
                    HUB
                     |
         redirect ---|
                     |
Spoke1 ---- shortcut | ---- shortcut ---- Spoke2
```

Hub：

```
ip nhrp redirect
```

Spoke：

```
ip nhrp shortcut
```

------

# 🧠 ENARSI 记忆口诀

看到题目：

```
spoke traffic via hub
```

想到：

```
DMVPN Phase3
```

记住：

```
Hub    redirect
Spoke  shortcut
```

------









> **Spoke → Spoke 直连 (bypass hub)**

题目说：

> voice traffic is coming via the hub router

说明现在是：

```
Spoke1 → Hub → Spoke2
```

而不是：

```
Spoke1 → Spoke2
```

所以要让 **Spoke 之间直接通信**。

------

# DMVPN 三个 Phase（考试重点）

### Phase 1

```
Spoke → Hub → Spoke
```

不能 spoke-to-spoke

------

### Phase 2

```
Spoke → Spoke
```

但需要复杂 routing

------

### Phase 3（考试最常见）

流程：

```
Spoke1 → Hub
Hub → Redirect
Spoke1 learns Spoke2 NBMA
Spoke1 → Spoke2
```

涉及两个关键命令：

| 设备  | 命令               |
| ----- | ------------------ |
| Hub   | `ip nhrp redirect` |
| Spoke | `ip nhrp shortcut` |

------

# 题目问

> Which command is required **on both spoke routers**

所以必须是 **Spoke 上的命令**

答案是：

```
B. ip nhrp shortcut
```

------

# 每个选项解释

### A

```
ip nhrp nhs multicast
```

错误

这是 **Hub 地址配置**

------

### B

```
ip nhrp shortcut
```

✅ 正确

作用：

允许 spoke 使用 **NHRP shortcut**

即：

```
Spoke → Spoke direct tunnel
```

------

### C

```
ip nhrp map dynamic
```

错误

这是 **Hub 配置**

------

### D

```
ip nhrp redirect
```

错误

这是 **Hub 才需要**

------

# DMVPN Phase3 一张图

```
1. Spoke1 → Hub → Spoke2
2. Hub sends NHRP redirect
3. Spoke1 sends NHRP resolution
4. Spoke1 learns Spoke2 NBMA
5. Spoke1 → Spoke2 direct tunnel
```

配置：

Hub

```
ip nhrp redirect
```

Spoke

```
ip nhrp shortcut
```

------

# ENARSI 秒杀口诀

看到题目：

```
spoke traffic via hub
```

立即想到：

```
DMVPN Phase 3
```

记忆：

```
Hub    → redirect
Spoke  → shortcut
```

------



------

# DMVPN 

## 1️⃣ DMVPN 三个 Phase

| Phase         | 拓扑                           | 特点                  | 典型命令              |
| ------------- | ------------------------------ | --------------------- | --------------------- |
| **Phase 1**   | Spoke → Hub → Spoke            | 只允许通过 Hub        | 无 shortcut           |
| **Phase 2**   | Spoke → Spoke                  | 直连，但 routing 复杂 | NHRP resolution       |
| **Phase 3** ⭐ | Spoke → Hub → Redirect → Spoke | **最常见**            | `redirect + shortcut` |

------

# 2️⃣ DMVPN 拓扑图

## Phase 1

```
Spoke1 ----\
             HUB
Spoke2 ----/
```

通信路径：

```
Spoke1 → Hub → Spoke2
```

特点：

- 不支持 spoke-to-spoke
- routing 最简单

------

## Phase 2

```
Spoke1 -------- Spoke2
      \        /
        HUB
```

通信路径：

```
Spoke1 → Spoke2
```

但 routing 必须知道 spoke route。

问题：

- routing design复杂
- summary困难

------

## Phase 3（考试最爱）

```
Step1

Spoke1 → Hub → Spoke2

Step2

Hub sends redirect

Step3

Spoke1 builds shortcut

Spoke1 → Spoke2
```

------

# 3️⃣ DMVPN Phase 3 工作流程

```
1  Spoke1 sends packet
2  Hub forwards
3  Hub sends NHRP redirect
4  Spoke1 sends NHRP resolution
5  Spoke1 learns NBMA of Spoke2
6  Direct tunnel created
```

------

# 4️⃣ 四个 NHRP 核心命令（必考）

| 命令               | 设备  | 作用          | 是否常考 |
| ------------------ | ----- | ------------- | -------- |
| `ip nhrp nhs`      | Spoke | 指定 Hub      | ⭐        |
| `ip nhrp map`      | Spoke | mapping NBMA  | ⭐        |
| `ip nhrp redirect` | Hub   | 通知 shortcut | ⭐⭐⭐      |
| `ip nhrp shortcut` | Spoke | 允许直连      | ⭐⭐⭐      |

------

# 5️⃣ Phase 3 标准配置

## Hub

```
interface Tunnel0

ip nhrp network-id 1
ip nhrp redirect
```

------

## Spoke

```
interface Tunnel0

ip nhrp network-id 1
ip nhrp nhs 10.1.1.1
ip nhrp shortcut
```

------

# 6️⃣ 逻辑

### 题目说

```
spoke traffic goes via hub
```

说明：

```
需要 spoke-to-spoke
```

想到：

```
DMVPN Phase3
```

答案组合：

```
Hub   → ip nhrp redirect
Spoke → ip nhrp shortcut
```

------

# 7️⃣ ENARSI 

| 选项                           | 错误原因 |
| ------------------------------ | -------- |
| `ip nhrp redirect` 在 spoke    | 错       |
| `ip nhrp shortcut` 在 hub      | 错       |
| `ip nhrp map dynamic` 在 spoke | 错       |
| `ip nhrp nhs multicast`        | 假命令   |

------

# 8️⃣ DMVPN + Routing 关系

| Phase  | routing       |
| ------ | ------------- |
| Phase1 | 最简单        |
| Phase2 | routing复杂   |
| Phase3 | routing简单 ⭐ |

Phase3 支持：

```
EIGRP summary
OSPF
BGP
```

------

# 9️⃣ DMVPN 记忆口诀

```
Phase1
只走 Hub

Phase2
直接走

Phase3
Hub redirect
Spoke shortcut
```

再压缩版：

```
Hub    → redirect
Spoke  → shortcut
```

------

