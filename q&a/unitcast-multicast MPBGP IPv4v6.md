下面给你一张 **CCNP ENARSI / MP-BGP Address-Family 对比表（考试常考版）**。很多题其实就是让你区分 **unicast / multicast / vpnv4 / vpnv6**。

------

# MP-BGP Address-Family Cheat Sheet

| Address Family     | 用途            | 典型命令                        | 使用场景            |
| ------------------ | --------------- | ------------------------------- | ------------------- |
| **ipv4 unicast**   | 传统 IPv4 路由  | `address-family ipv4 unicast`   | Internet / 企业网络 |
| **ipv6 unicast**   | IPv6 路由       | `address-family ipv6 unicast`   | IPv6 网络           |
| **ipv4 multicast** | IPv4 组播路由   | `address-family ipv4 multicast` | PIM / IPTV          |
| **ipv6 multicast** | IPv6 组播路由   | `address-family ipv6 multicast` | IPv6 multicast      |
| **vpnv4**          | MPLS L3VPN IPv4 | `address-family vpnv4`          | MPLS VPN            |
| **vpnv6**          | MPLS L3VPN IPv6 | `address-family vpnv6`          | IPv6 MPLS VPN       |

------

# 每种 Address-Family 示例

## 1 IPv4 Unicast（最常见）

```cisco
router bgp 65000
 address-family ipv4 unicast
  network 10.1.1.0 mask 255.255.255.0
```

用途：

- 传统 BGP
- Internet routing

------

# 2 IPv6 Unicast

必须关闭默认 IPv4 AF：

```cisco
router bgp 65000
 no bgp default ipv4-unicast
 address-family ipv6 unicast
  network 2001:DB8::/64
```

用途：

- IPv6 routing
- MP-BGP IPv6

考试关键词：

```
advertise IPv6 routes
MP-BGP IPv6
```

------

# 3 IPv4 Multicast

用于 **PIM multicast routing**。

```cisco
router bgp 65000
 address-family ipv4 multicast
```

用途：

```
IPTV
video streaming
multicast routing
```

------

# 4 IPv6 Multicast

```cisco
router bgp 65000
 address-family ipv6 multicast
```

用于：

```
IPv6 multicast
```

考试很少考。

------

# 5 VPNv4（MPLS VPN）

Provider backbone。

```cisco
router bgp 65000
 address-family vpnv4
  neighbor 1.1.1.1 activate
  neighbor 1.1.1.1 send-community extended
```

用途：

```
MPLS L3VPN
```

特点：

```
Route Distinguisher
Route Target
```

------

# 6 VPNv6

IPv6 MPLS VPN。

```cisco
router bgp 65000
 address-family vpnv6
```

------

# ENARSI 高频考点总结

| 题目关键词            | 选什么                              |
| --------------------- | ----------------------------------- |
| advertise IPv6 routes | **ipv6 unicast**                    |
| multicast routing     | **ipv4 multicast / ipv6 multicast** |
| MPLS VPN              | **vpnv4 / vpnv6**                   |
| normal BGP routing    | **ipv4 unicast**                    |

------

# 一个最容易错的地方

很多人会忘记这条：

```cisco
no bgp default ipv4-unicast
```

原因：

默认 BGP 会自动开启：

```
address-family ipv4 unicast
```

如果要做 **MP-BGP IPv6**，通常需要关闭。

------

# ENARSI 秒杀口诀

```
普通路由 → unicast
组播 → multicast
MPLS VPN → vpnv4/vpnv6
```

------

