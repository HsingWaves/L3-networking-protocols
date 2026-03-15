**Interface-Level Control:** By using `ip ospf authentication message-digest` on the interface, you are enabling MD5 only for that specific link.

**Encrypted Packets:** The `message-digest-key 1 md5 exam` command ensures that the OSPF packets are hashed with MD5, meeting the "encrypted" requirement.

**Self-Contained:** It doesn't rely on global area settings, making it the most precise way to target a single neighborship.

This uses **Area-based authentication**. While it would work, the command `area 0 authentication message-digest` enables MD5 for *every* interface in Area 0 on that router. The question specifically asks to configure it for a *single* neighborship.



针对 OSPF 认证（你提到的“加密”），在 Cisco IOS 中主要分为 **明文 (Type 1)** 和 **MD5 密文 (Type 2)**。为了满足你“加密”的需求，我们重点讨论 MD5 认证。

OSPF 认证的配置逻辑可以总结为：**“开启开关” + “设置密钥”**。

------

## 1. OSPF 认证思路与格式

| **认证类型**             | **数据包表现**                                 | **安全性**         |
| ------------------------ | ---------------------------------------------- | ------------------ |
| **明文 (Clear Text)**    | 密码以明文形式包含在 OSPF 报文头部             | 低（易被抓包窃取） |
| **MD5 (Message Digest)** | 密码不直接发送，发送的是根据密码计算出的哈希值 | 中/高（抗篡改）    |

------

## 2. 命令与范围 (Area vs. Single Interface)

在 OSPF 中，你可以在 **进程模式（影响整个 Area）** 或 **接口模式（仅影响该特定链路）** 下开启认证。

### A. 接口范围 (Single Interface) - 优先级更高

如果你只想针对特定的邻居（如题目中的 Single Router Neighborship）进行配置，直接在接口下操作。

- **思路：** 接口开启 MD5 开关 -> 接口配置密钥。

- **命令格式：**

  Bash

  ```
  interface Ethernet0/2
   ip ospf authentication message-digest            # 开启 MD5 开关
   ip ospf message-digest-key 1 md5 [PASSWORD]      # 设置 Key ID 和密码
  ```

### B. 区域范围 (Area-wide)

如果你希望某个区域内的所有路由器接口都启用认证，可以在 OSPF 进程下配置。但注意：**密钥仍然必须在每个接口下单独设置**。

- **思路：** 进程内开启区域开关 -> 每个接口配置密钥。

- **命令格式：**

  Bash

  ```
  router ospf 100
   area 0 authentication message-digest             # 该区域所有接口开启 MD5 开关
  !
  interface Ethernet0/2
   ip ospf message-digest-key 1 md5 [PASSWORD]      # 接口下依然需要密钥
  ```

------

## 3. 核心注意点（避坑指南）

1. **优先级：** 接口级配置会覆盖区域级配置。
2. **Key ID 必须一致：** 邻居两端的 `message-digest-key [ID]` 中的 ID 数字必须匹配，否则邻居关系无法建立（Down）。
3. **MTU 与认证：** 开启认证会占用 OSPF 报文头部的空间，虽然通常不影响 MTU，但在极少数严苛环境下需留意。
4. **不匹配的后果：** 如果一端配置了认证而另一端没配，或者密码错误，你会看到 `OSPF-5-ADJCHG: ... Neighbor Down: Adjacency forced to reset` 报错。

------

## 4. 总结对比表

| **维度**           | **区域配置 (Area)**    | **接口配置 (Interface)**       |
| ------------------ | ---------------------- | ------------------------------ |
| **配置位置**       | `router ospf X`        | `interface X`                  |
| **适用场景**       | 整个区域标准化安全策略 | 针对特定邻居、特定链路         |
| **灵活性**         | 较低（一开全开）       | 极高（可为不同邻居设不同密码） |
| **是否需接口密钥** | **是**                 | **是**                         |

 **Message-Digest** (specifically MD5 in this context) refers to a cryptographic hash function used to verify the **integrity** and **authenticity** of routing updates.

Instead of sending your password in "clear text" (where anyone with a packet sniffer like Wireshark could read it), OSPF uses a mathematical algorithm to create a unique "fingerprint" of the data.

------

### 1. How it Works (The Concept)

Think of a message-digest as a **digital seal** on an envelope.

1. **The Input:** The router takes the OSPF packet content + your secret password (the key).
2. **The Algorithm:** It runs these through the MD5 algorithm.
3. **The Digest:** The result is a fixed-length 128-bit hexadecimal string (the "digest").
4. **The Transmission:** The router sends the OSPF packet and the *digest*, but **never** the actual password.
5. **The Verification:** The receiving router performs the same calculation using its own copy of the password. If its calculated digest matches the one in the packet, the update is accepted.

------

### 2. Why use Message-Digest?

- **Anti-Spoofing:** An attacker cannot pretend to be a legitimate router because they don't know the secret key required to generate a valid digest.
- **Anti-Tampering:** If a "man-in-the-middle" changes even a single bit in the OSPF LSA (Link State Advertisement) while it's in transit, the digest will no longer match, and the receiving router will drop the packet.
- **Security:** Unlike "Plain Text" authentication (Type 1), where the password `exam` is visible in the packet header, MD5 (Type 2) keeps the password hidden.



Cisco Privileged Exce Mode:

**故障现象**：管理员尝试切换到 R1 的特权模式（即从 `R1>` 进入 `R1#`），但操作失败。

**触发点**：在 Cisco IOS 中，如果你没有配置 `enable password` 或 `enable secret`，且通过远程登录（如 Telnet/SSH）访问设备，系统通常会拒绝你进入特权模式，并提示 "No password set"。

**核心需求**：需要一条正确的命令来设置进入特权模式的密码。

 `enable password Cisco@123`