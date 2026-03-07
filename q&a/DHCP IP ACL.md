------

# 一、DHCP + ip helper-address 工作流程

```
Client (192.168.30.0/24)
       |
       | 1️⃣ DHCP Discover
       | src: 0.0.0.0:68
       | dst: 255.255.255.255:67
       |
       v
Gi3 interface (Router)
IP = 192.168.30.1
ACL applied: ip access-group Gi3-in in
       |
       | 2️⃣ Router converts broadcast → unicast
       | via ip helper-address
       |
       v
DHCP Server
192.168.255.3
```

------

# 二、完整 DHCP 4步流程

### ① DHCP Discover

客户端还没有 IP：

```
src IP      0.0.0.0
src port    68 (bootpc)

dst IP      255.255.255.255
dst port    67 (bootps)
```

图示：

```
Client
0.0.0.0:68
   |
   | broadcast
   v
255.255.255.255:67
```

对应 ACL 允许：

```
permit udp host 0.0.0.0 eq bootpc host 255.255.255.255 eq bootps
```

------

### ② Router Relay (ip helper-address)

Router 收到广播后：

```
ip helper-address 192.168.255.3
```

Router 会转发成：

```
src IP   192.168.30.1 (relay agent)
dst IP   192.168.255.3
Router
192.168.30.1
   |
   | unicast
   v
192.168.255.3
```

注意：

**这一步不会经过 Gi3 inbound ACL**
因为是 **router自己发出去的流量**。

------

### ③ DHCP Offer

Server 回复：

```
src 192.168.255.3:67
dst 192.168.30.1:67
```

Router 再转发给客户端：

```
src 192.168.255.3
dst 255.255.255.255
```

------

### ④ DHCP Request / ACK

客户端拿到 IP 后会出现：

```
src 192.168.30.x:68
dst 192.168.255.3:67
```

这就是 **续租 / request**。

对应 ACL：

```
permit udp 192.168.30.0 0.0.0.255 eq bootpc host 192.168.255.3 eq bootps
```

------

# 三、ACL 方向判断（考试重点）

接口配置：

```
interface Gi3
 ip access-group Gi3-in in
```

意思：

```
ACL 只过滤：
进入 Gi3 的流量
```

图示：

```
Client  --->  Router(Gi3)
         ACL check here
```

所以 ACL 只会看到：

```
Client → Router
```

不会看到：

```
Router → Server
Server → Router
Router → Client
```

------

# 四、本题真正需要允许的 DHCP流量

### 必须允许 1：DHCP Discover

```
0.0.0.0:68 → 255.255.255.255:67
```

ACL：

```
permit udp host 0.0.0.0 eq bootpc host 255.255.255.255 eq bootps
```

------

### 必须允许 2：DHCP Request / Renew

```
192.168.30.x:68 → 192.168.255.3:67
```

ACL：

```
permit udp 192.168.30.0 0.0.0.255 eq bootpc host 192.168.255.3 eq bootps
```

------

# 五、DHCP + ACL 常见考点总结

看到下面关键词基本就是这个套路：

| 关键词            | 结论                      |
| ----------------- | ------------------------- |
| ip helper-address | DHCP relay                |
| ACL inbound       | 只看 client → router      |
| DHCP 不工作       | bootpc / bootps 被 ACL 拦 |
| 0.0.0.0           | DHCP discover             |
| 255.255.255.255   | DHCP broadcast            |

------

# 六、一句话记忆（考试版）

```
DHCP client → server
68 → 67
bootpc → bootps
```

广播阶段：

```
0.0.0.0 → 255.255.255.255
```

续租阶段：

```
192.168.x.x → DHCP server
```

------

