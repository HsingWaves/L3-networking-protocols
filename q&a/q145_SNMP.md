------

# 一、问题本质

现象：

> R1 用 SNMP 监控，但监控系统只能拿到部分信息

根因：

> CoPP 对 SNMP 做了过低的限速（CIR 太小）

------

# 二、SNMP 在这里的角色

SNMP 是：

- 监控系统（NMS）轮询设备
- 设备从 CPU（control-plane）返回数据

关键点：

> SNMP 是控制平面流量

所以：

它会被 **CoPP（Control Plane Policing）限制**

------

# 三、CoPP 在干什么？

当前配置：

```plaintext
Class-map: SNMP-Out
police:
  cir 8000 bps
  bc 1500 bytes
  conform → transmit
  exceed  → drop
```

意思是：

- 每秒最多允许 8000 bit
- 超过的流量直接丢弃

------

# 四、为什么会“只收到部分数据”？

因为：

SNMP walk / bulk 会一次性返回大量数据。

但：

```plaintext
8000 bps ≈ 1 KB/s
```

如果一次返回 20 KB：

- 前 1 KB 通过
- 剩下 19 KB 被 drop

于是：

监控系统数据不完整

------

# 五、核心概念总结

## 1️⃣ CIR（Committed Information Rate）

- 平均允许速率
- 单位 bps
- 控制长期带宽

------

## 2️⃣ BC（Burst Size）

- 允许的突发大小
- 单位 bytes
- 控制瞬间流量

------

## 3️⃣ conform / exceed

| 状态    | 含义   | 动作     |
| ------- | ------ | -------- |
| conform | 没超速 | transmit |
| exceed  | 超速   | drop     |

------

# 六、正确解决方案

提高 CIR。

例如改成 64 kbps：

```bash
conf t
policy-map CoPP
 class SNMP-Out
  police cir 64000 bc 8000
```

然后验证：

```bash
show policy-map control-plane
```

观察：

```plaintext
exceeded packets
```

是否还在增长。

------

# 七、为什么不是改 ACL？

ACL 只负责：

> 匹配什么流量进入这个 class

问题不是匹配错了。

问题是：

> 匹配到了，但被限速丢弃。

------

# 八、这题考点总结（考试视角）

这题同时考：

✔ SNMP 工作机制
✔ Control Plane vs Data Plane
✔ CoPP 作用
✔ Policing 机制
✔ CIR / BC 含义
✔ exceed drop 的后果

------

# 九、一句话记忆

> SNMP 数据不完整 + CoPP 存在 = 90% 是 CIR 太小

------

# 十、最终总结图（逻辑链）

```
NMS 发 SNMP 请求
        ↓
设备 CPU 处理
        ↓
SNMP 响应发出
        ↓
经过 CoPP policing
        ↓
CIR 太小 → exceed → drop
        ↓
监控系统只拿到部分数据
```

解决：

```
提高 CIR
```

------

RADIUS 的**默认端口**是：

| 功能                   | 端口         | 协议 |
| ---------------------- | ------------ | ---- |
| 认证（Authentication） | **UDP 1812** | UDP  |
| 计费（Accounting）     | **UDP 1813** | UDP  |

------

## 为什么有时看到 1645 / 1646？

历史原因：

| 功能 | 旧端口   |
| ---- | -------- |
| 认证 | UDP 1645 |
| 计费 | UDP 1646 |

早期厂商使用 1645/1646，后来 IANA 正式标准化为：

- **1812（auth）**
- **1813（acct）**

现代设备默认都是 1812/1813。

------

## CCNP 考试记忆点

如果题目说：

> AAA server 使用默认端口

那就是：

```plaintext
Authentication → 1812
Accounting → 1813
```

如果设备配置成 1814、1815 等 → 一定不匹配。

------

一句话记忆：

> RADIUS = 1812 / 1813
> TACACS+ = TCP 49

