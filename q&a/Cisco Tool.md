| 工具                                           | 作用                     | 典型题目                   |
| ---------------------------------------------- | ------------------------ | -------------------------- |
| **Path Trace**                                 | 查看数据包在网络中的路径 | PC 无法访问服务器 / 打印机 |
| **Device Trace**                               | 查看设备健康状态         | CPU / memory / interface   |
| **ACL Trace**                                  | 检查 ACL 是否阻挡流量    | ACL policy 问题            |
| **Application Experience / Application Trace** | 查看应用性能             | SaaS / HTTP / latency      |
| **Client 360**                                 | 查看客户端连接问题       | WiFi 连接失败              |
| **Network Health**                             | 网络整体健康             | SLA / performance          |

# 1️⃣ Path Trace（最常考）

作用：

```
模拟 packet 在网络中的路径
```

会显示：

```
Client
   ↓
Access Switch
   ↓
Distribution
   ↓
Firewall
   ↓
Server
```

并标出：

- ACL drop
- policy drop
- routing problem

典型题：

```
PC cannot reach server
```

答案：

```
Path Trace
```

------

# 2️⃣ Device Trace

查看设备问题：

```
CPU high
Memory issue
Interface down
```

例如题目：

```
Switch performance issue
```

------

# 3️⃣ ACL Trace

只检查：

```
ACL rule
SGT policy
TrustSec
```

例题：

```
ACL blocking traffic
```

------

# 4️⃣ Application Trace

用于：

```
Application performance
```

例如：

```
Office365 slow
Web application latency
```

------

# 5️⃣ Client 360

查看：

```
WiFi client issue
```

例如：

```
Client cannot connect to WLAN
```

------

# 🔥 ENARSI 秒杀口诀

| 关键词           | 工具              |
| ---------------- | ----------------- |
| connectivity     | **Path Trace**    |
| ACL              | ACL Trace         |
| device health    | Device Trace      |
| WiFi client      | Client 360        |
| application slow | Application Trace |

------

# 🔥 真实考试最常见

**80% 都是：**

```
client cannot reach server
```

答案基本都是：

```
Path Trace
```