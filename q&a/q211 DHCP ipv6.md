下面是一张 **IPv6 Address Assignment（SLAAC / Stateless DHCPv6 / Stateful DHCPv6）

------

# IPv6 Address Assignment 一页图

```
                IPv6 地址获取方式
                       │
        ┌──────────────┼──────────────┐
        │              │              │
      SLAAC      Stateless DHCPv6   Stateful DHCPv6
   (RA only)       (RA + DHCP)        (DHCP only)
```

------

# 1 SLAAC（Stateless Address Autoconfiguration）

### 工作方式

```
Router Advertisement (RA)
      ↓
Prefix: 2001:db8:1::/64
      ↓
Client 自动生成地址
```

地址组成：

```
Prefix + Interface ID
```

例如：

```
2001:db8:1:: + EUI-64
```

------

### Router 配置

```cisco
interface g0/0
 ipv6 address 2001:db8:1::1/64
```

没有 DHCP。

------

### Client 配置

通常不需要配置：

```
IPv6 enable
```

或：

```cisco
ipv6 address autoconfig
```

------

### 特点

| 项目            | 来源  |
| --------------- | ----- |
| IPv6 address    | SLAAC |
| default gateway | RA    |
| DNS             | 无    |

------

# 2 Stateless DHCPv6

最常考模式。

### 工作方式

```
RA 提供 prefix
DHCPv6 提供 DNS
```

流程：

```
RA
↓
Client 生成 IPv6 address
↓
DHCPv6 请求 DNS
```

------

### Router 配置

```
interface g0/0
 ipv6 address 2001:db8:1::1/64
 ipv6 nd other-config-flag
 ipv6 dhcp server IPV6POOL
```

DHCP pool：

```cisco
ipv6 dhcp pool IPV6POOL
 dns-server 2001:db8::53
 domain-name lab.local
```

------

### Client 配置

```cisco
ipv6 address autoconfig
```

------

### 特点

| 项目            | 来源   |
| --------------- | ------ |
| IPv6 address    | SLAAC  |
| DNS             | DHCPv6 |
| default gateway | RA     |

------

# 3 Stateful DHCPv6

类似 **IPv4 DHCP**。

### 工作方式

```
DHCPv6 server
↓
分配 IPv6 地址
```

客户端不会自己生成地址。

------

### Router 配置

```
interface g0/0
 ipv6 address 2001:db8:1::1/64
 ipv6 nd managed-config-flag
 ipv6 dhcp server IPV6POOL
```

DHCP pool：

```cisco
ipv6 dhcp pool IPV6POOL
 address prefix 2001:db8:1::/64
 dns-server 2001:db8::53
```

------

### Client 配置

```cisco
ipv6 address dhcp
```

------

### 特点

| 项目            | 来源   |
| --------------- | ------ |
| IPv6 address    | DHCPv6 |
| DNS             | DHCPv6 |
| default gateway | RA     |

------

# 4 RA Flags（最重要考试点）

RA 中有两个关键标志：

| Flag       | 含义                         |
| ---------- | ---------------------------- |
| **M flag** | Managed (DHCPv6 分配地址)    |
| **O flag** | Other info (DHCPv6 提供 DNS) |

------

### 组合关系

| M    | O    | 模式             |
| ---- | ---- | ---------------- |
| 0    | 0    | SLAAC            |
| 0    | 1    | Stateless DHCPv6 |
| 1    | X    | Stateful DHCPv6  |

------

# 5 Cisco 对应命令

| 模式             | Router 命令                   |
| ---------------- | ----------------------------- |
| SLAAC            | 默认                          |
| Stateless DHCPv6 | `ipv6 nd other-config-flag`   |
| Stateful DHCPv6  | `ipv6 nd managed-config-flag` |

------

# 6 客户端命令总结

| 命令                      | 用途            |
| ------------------------- | --------------- |
| `ipv6 address autoconfig` | SLAAC           |
| `ipv6 address dhcp`       | Stateful DHCPv6 |

------

# 7 ENARSI 常见题型

### 看到

```
ipv6 nd other-config-flag
```

答案通常：

```
ipv6 address autoconfig
```

------

### 看到

```
ipv6 nd managed-config-flag
```

答案通常：

```
ipv6 address dhcp
```

------

# 8 一句话记忆

```
M flag = DHCP address
O flag = DHCP DNS
```

------

