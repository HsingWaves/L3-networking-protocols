7 个陷阱

------

# CCNP ACL Cheat Sheet (7 大陷阱)

```
ACL = Top → Down matching
First match wins
Implicit deny at the end
```

------

# 1️⃣ First Match Rule（最重要）

ACL **从上往下匹配，匹配后立即停止**

```
10 permit tcp host A host B eq 80
20 deny tcp any host B eq 80
```

如果：

```
A → B port 80
```

匹配：

```
rule 10 → permit
rule 20 不再检查
```

**考试陷阱**

```
10 deny tcp any host B eq 23
20 permit tcp host A host B eq 23
```

A → B telnet

❌ **被 rule 10 拦截**

------

# 2️⃣ Implicit Deny（隐藏 deny）

所有 ACL **最后都有一条**

```
deny ip any any
```

例：

```
10 permit tcp host A host B eq 80
```

只有 HTTP 可以通过，其它全部：

```
❌ drop
```

解决：

```
permit ip any any
```

------

# 3️⃣ Sequence Number 插入规则

Cisco ACL 可以 **插入规则**

原 ACL

```
10 permit ip A any
20 deny ip any any
```

插入：

```
15 permit tcp B C eq 22
```

命令

```
ip access-list extended ACL1
sequence 15 permit tcp B C eq 22
```

**考试经常问**

哪个 sequence 可以让 ACL 生效。

------

# 4️⃣ Standard ACL 位置

Standard ACL **只看 source**

```
access-list 10 deny 10.1.1.0 0.0.0.255
access-list 10 permit any
```

如果放在靠近 source：

```
source ---- R1 ---- R2 ---- destination
```

会把 **所有去任何地方的流量都拦截**。

最佳实践：

```
Standard ACL
= close to destination
```

------

# 5️⃣ Extended ACL 位置

Extended ACL 可以匹配：

```
source
destination
protocol
port
```

最佳实践：

```
Extended ACL
= close to source
```

原因：

减少无用流量。

------

# 6️⃣ Wildcard Mask（很多人错）

Wildcard mask 是 **反掩码**

```
Subnet mask    255.255.255.0
Wildcard mask  0.0.0.255
```

例：

```
access-list 10 permit 10.1.1.0 0.0.0.255
```

匹配：

```
10.1.1.0 - 10.1.1.255
```

------

### 常见 wildcard

| Subnet | Wildcard      |
| ------ | ------------- |
| /24    | 0.0.0.255     |
| /16    | 0.0.255.255   |
| /8     | 0.255.255.255 |

------

# 7️⃣ ACL Direction（in / out）

ACL 方向非常重要。

```
interface Gi0/0
 ip access-group ACL1 in
```

意思：

```
ACL applied to traffic
ENTERING interface
```

------

### 示例

```
Client → Router → Server
```

如果 ACL 在

```
Gi0/0 inbound
```

匹配：

```
Client → Router
```

------

### outbound

```
interface Gi0/0
 ip access-group ACL1 out
```

匹配：

```
Router → Server
```

------

# Bonus（考试超级爱考）

## Telnet vs SSH port

| Protocol | Port |
| -------- | ---- |
| Telnet   | 23   |
| SSH      | 22   |
| HTTP     | 80   |
| HTTPS    | 443  |
| FTP      | 21   |

------

# ACL 一张图理解

```
packet →
        rule 10
        ↓
     match?
      yes → action
      no
        ↓
        rule 20
        ↓
     match?
      yes → action
      no
        ↓
   implicit deny
```

------

# CCNP ACL 解题 4 步法

考试直接这样看：

```
1 看 source
2 看 destination
3 看 port
4 看顺序
```

再判断：

```
in / out
```

------

# ENARSI 最容易丢分的 3 类 ACL 题

1️⃣ **deny 在 permit 前面**

2️⃣ **implicit deny**

3️⃣ **ACL direction 错**

------

如果你愿意，我可以再给你一份 **ENARSI 300-410 Security 一页 cheat sheet**，里面包含：

```
DHCP Snooping
DAI
IP Source Guard
RA Guard
IPv6 ND inspection
CoPP
```

下面是一页 **CCNP ENARSI Security Cheat Sheet**（考试非常高频的 L2 / Control-Plane 安全功能）。
基本上 **300-410 的 Security 题目大部分都来自这里**。

------

# CCNP ENARSI Security Cheat Sheet

```text
Focus areas:
DHCP Snooping
DAI
IP Source Guard
RA Guard
IPv6 ND Inspection
CoPP
```

------

# 1️⃣ DHCP Snooping

### 作用

防止 **Rogue DHCP Server**

```text
Client ---- Switch ---- Legit DHCP Server
               |
               |---- Fake DHCP Server (blocked)
```

------

### 工作方式

端口类型：

| Port      | Function             |
| --------- | -------------------- |
| Trusted   | DHCP server / uplink |
| Untrusted | Client ports         |

默认：

```text
ALL ports = untrusted
```

------

### 过滤规则

| DHCP message | Allowed          |
| ------------ | ---------------- |
| Discover     | ✔                |
| Request      | ✔                |
| Offer        | ❌ from untrusted |
| ACK          | ❌ from untrusted |

------

### 核心配置

```cisco
ip dhcp snooping
ip dhcp snooping vlan 10

interface Gi1/0/1
ip dhcp snooping trust
```

------

### 常见问题

| 问题               | 原因                     |
| ------------------ | ------------------------ |
| Client 无法获取 IP | uplink 没设 trusted      |
| DHCP 不工作        | Option 82 被 server 拒绝 |

解决：

```cisco
no ip dhcp snooping information option
```

------

# 2️⃣ Dynamic ARP Inspection (DAI)

### 作用

防止 **ARP Spoofing**

攻击：

```text
Attacker sends fake ARP
Gateway IP → attacker MAC
```

------

### 工作原理

DAI 会检查：

```text
IP
MAC
Port
```

对比 **DHCP Snooping binding table**

------

### 依赖

```text
DAI requires DHCP Snooping
```

------

### 配置

```cisco
ip arp inspection vlan 10
```

uplink：

```cisco
interface Gi1/0/1
ip arp inspection trust
```

------

# 3️⃣ IP Source Guard

### 作用

防止 **IP Spoofing**

检查：

```text
IP
MAC
Port
```

来自：

```text
DHCP Snooping binding table
```

------

### 配置

```cisco
interface Gi1/0/10
ip verify source
```

------

### 检查逻辑

```text
packet arrives
↓
lookup binding table
↓
IP + MAC + Port match?
    yes → forward
    no  → drop
```

------

# 4️⃣ IPv6 RA Guard

### 作用

防止 **Rogue IPv6 Router Advertisement**

攻击：

```text
Fake router advertises itself as default gateway
```

------

### 结果

客户端会使用：

```text
Attacker → default gateway
```

MITM attack。

------

### 配置

```cisco
ipv6 nd raguard policy BLOCK_RA

interface Gi1/0/10
ipv6 nd raguard attach-policy BLOCK_RA
```

------

# 5️⃣ IPv6 ND Inspection

### 作用

保护 **IPv6 Neighbor Discovery**

防止：

```text
Neighbor spoofing
Duplicate Address Detection attack
```

------

### 工作方式

建立 **IPv6 binding table**

```text
IPv6 address
MAC address
Port
```

------

### 配置

```cisco
ipv6 nd inspection vlan 10
```

------

# 6️⃣ IPv6 Device Tracking

### 作用

记录 IPv6 host 信息

用于：

```text
ND inspection
Source guard
Security tracking
```

------

### 记录

```text
IPv6 address
MAC
Interface
```

------

# 7️⃣ CoPP (Control Plane Policing)

### 作用

保护 **router CPU**

防止：

```text
DoS attack
control-plane flooding
```

攻击例子：

```text
大量 ICMP
大量 SNMP
大量 routing packets
```

------

### 流量分类

```text
class-map
```

------

### 策略

```text
policy-map
```

------

### 应用

```cisco
control-plane
service-policy input COPP_POLICY
```

------

### 示例

```cisco
class-map match-any SNMP
 match protocol snmp

policy-map COPP_POLICY
 class SNMP
  police 32000
```

------

# ENARSI Security 功能关系

```text
                DHCP Snooping
                     |
        -------------------------------
        |                             |
        v                             v
   IP Source Guard              Dynamic ARP Inspection
```

IPv6:

```text
RA Guard
ND Inspection
Device Tracking
```

------

# ENARSI 高频考点

| Topic           | Frequency |
| --------------- | --------- |
| DHCP Snooping   | VERY HIGH |
| DAI             | HIGH      |
| IP Source Guard | HIGH      |
| RA Guard        | HIGH      |
| ND Inspection   | MEDIUM    |
| CoPP            | HIGH      |

------

# 最常见考试陷阱

### 1️⃣ uplink 未 trust

```text
DHCP reply dropped
```

------

### 2️⃣ Option82

DHCP server 不支持：

```text
clients fail to obtain IP
```

解决：

```cisco
no ip dhcp snooping information option
```

------

### 3️⃣ DAI 未信任 uplink

ARP reply 被丢弃。

------

### 4️⃣ CoPP rate 太低

SNMP / routing protocol 被限速。

------

# ENARSI Security 一句话记忆

```text
DHCP Snooping → stop rogue DHCP
DAI → stop ARP spoof
IP Source Guard → stop IP spoof
RA Guard → stop fake IPv6 router
ND inspection → secure IPv6 neighbor discovery
CoPP → protect router CPU
```

------

