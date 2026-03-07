snmp-server group NETVIEW v3 priv read NETVIEW access 20
snmp-server group NETADMIN v3 priv read NETVIEW write NETADMIN access 20
snmp-server community Cisc0Us3r RO 20
snmp-server community Cisc0wruS3r RW 20

是的，你理解得基本对，不过我给你 **精确一点的 Cisco 语义**，这样考试不会混淆。

**`access 20` 绑定的是 ACL 20（ACL list），不是某一条 entry。**

也就是说：

```
snmp-server community Cisc0Us3r RO 20
```

这里的 **20** 指的是：

👉 **ACL number 20**

设备会去找：

```
access-list 20 ...
```

然后用 **整个 ACL 20** 来过滤 SNMP source IP。

------

### 结构关系

```
SNMP configuration
        │
        │ uses
        ▼
ACL 20 (整个列表)
        │
        ├─ access-list 20 permit 10.221.10.10
        ├─ access-list 20 permit 10.221.10.11
        └─ implicit deny any
```

逻辑就是：

```
SNMP request arrives
        │
        ▼
Check source IP against ACL 20
        │
        ├─ match permit → allow SNMP
        └─ no match → deny
```

------

### 为什么这题选 A

原来 ACL 20 是：

```
access-list 20 permit 10.221.10.10
```

而 NMS 是：

```
10.221.10.11
```

不匹配 → **SNMP 被拒绝**

加上：

```
access-list 20 permit 10.221.10.11
```

就匹配了。

------

### 再强调一个 ENARSI 很爱考的点

很多人会被这里误导：

```
access-list 11 permit 10.221.10.11
```

但 **SNMP 根本没有引用 ACL 11**

只有：

```
access 20
```

所以：

```
ACL 存在 ≠ 功能使用
```

必须看 **谁引用了哪个 ACL**。

------

### 一句话记忆（考试非常重要）

**Feature → reference ACL number → ACL contains entries**

```
SNMP → access 20 → ACL 20 → entries
```

不是：

```
SNMP → entry
```

------

很好，这个我给你整理一张 **ENARSI 最容易混的 ACL 引用 Cheat Sheet**。
很多题就是考 **“ACL 被谁引用”**，而不是 ACL 本身。

------

# ENARSI ACL 引用 Cheat Sheet

## 1️⃣ SNMP 访问控制

限制 **哪个 NMS 可以访问设备**

配置：

```cisco
snmp-server community public RO 20
```

含义：

```
SNMP → access 20 → ACL 20
```

ACL：

```cisco
access-list 20 permit 10.221.10.11
```

逻辑：

```
SNMP request source IP
        │
        ▼
Check ACL 20
        │
permit → allow SNMP
deny   → drop
```

⚠️ 常见坑
ACL 存在但 **SNMP 没引用**

------

# 2️⃣ VTY / SSH / Telnet 限制

限制 **谁能登录设备**

配置：

```cisco
line vty 0 4
access-class 10 in
```

结构：

```
VTY → access-class 10 → ACL 10
```

ACL：

```cisco
access-list 10 permit 10.1.1.0 0.0.0.255
```

含义：

```
只有这个网段能 SSH / Telnet
```

------

# 3️⃣ NTP 访问控制

限制 **谁可以同步时间**

配置：

```cisco
ntp access-group peer 10
```

结构：

```
NTP → ACL 10
```

ACL：

```cisco
access-list 10 permit 10.1.1.5
```

只允许这个 NTP server。

------

# 4️⃣ CoPP (Control Plane Policing)

限制 **进入 CPU 的流量**

配置：

```cisco
class-map SNMP
 match access-group 100
```

结构：

```
CoPP class-map → ACL 100
```

ACL：

```cisco
access-list 100 permit udp any any eq snmp
```

------

# 5️⃣ PBR (Policy Based Routing)

配置：

```cisco
route-map PBR permit 10
 match ip address 101
 set ip next-hop 10.1.1.1
```

结构：

```
route-map → ACL 101
```

ACL：

```cisco
access-list 101 permit ip 192.168.1.0 0.0.0.255 any
```

含义：

```
匹配流量 → 改下一跳
```

------

# 6️⃣ Routing Protocol 分发控制

例如 OSPF：

```cisco
distribute-list 10 in
```

结构：

```
Routing → distribute-list → ACL 10
```

ACL：

```cisco
access-list 10 deny 10.1.1.0 0.0.0.255
access-list 10 permit any
```

------

# 最重要的考试原则

一定记住：

```
Feature
   ↓
引用 ACL number
   ↓
ACL entries
```

不是：

```
ACL 存在
   ↓
功能自动使用
```

------

# 

### ❌ 错误思维

```
access-list 11 permit 10.1.1.1
```

看到 IP 以为 **已经允许**

但如果配置是：

```
snmp-server community public RO 20
```

那 ACL 11 **完全没用**。

------

### ✔ 正确思维

先找：

```
哪个功能
引用哪个 ACL
```

再看 ACL。

------

# 

ENARSI ACL 三步：

```
1 看 feature
2 找 reference ACL
3 看 ACL entries
```

