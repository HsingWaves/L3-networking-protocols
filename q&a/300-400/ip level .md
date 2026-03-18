在 Cisco IOS 中，`ip precedence` 是一个老牌的 QoS 标记机制，它使用 IPv4 头部 TOS 字段的前 3 位。由于 $2^3 = 8$，所以共有 **0 到 7** 这八个等级。

对于 `ip ssh precedence 6` 命令，它的可选值及对应的含义如下表：

### IP Precedence 等级表

| **级别 (Value)** | **名称 (Designation)** | **用途说明 (Common Usage)**                                  |
| ---------------- | ---------------------- | ------------------------------------------------------------ |
| **7**            | **Network**            | 预留给最核心的网络控制（如 STP、部分 Keepalive）。           |
| **6**            | **Internet**           | **网络控制流量专用**（如 BGP、OSPF、SSH、SNMP）。这是管理流量的“免死金牌”。 |
| **5**            | **Critical**           | 常用于语音流量 (Voice/VoIP)。                                |
| **4**            | **Flash-override**     | 视频会议或实时交互流量。                                     |
| **3**            | **Flash**              | 重要业务数据。                                               |
| **2**            | **Immediate**          | 普通业务数据。                                               |
| **1**            | **Priority**           | 背景流量（优先级略高于普通流量）。                           |
| **0**            | **Routine**            | **默认值 (Best Effort)**。所有普通流量默认都是 0。           |

------

### 为什么 SSH 要用 6 而不是 7？

在 Cisco 的设计哲学里：

- **级别 7 (Network Control)**：是给“如果不通，网络就会瞬间瘫痪”的流量准备的（比如防止二层环路的 BPDU）。
- **级别 6 (Internetwork Control)**：是给“路由协议和管理协议”准备的。SSH 属于管理协议，所以 6 是它的最高标准建议。

------

### 与现代 DSCP 的对应关系

现在的网络更多使用 **DSCP (Differentiated Services Code Point)**，它是 6 位宽。IP Precedence 6 对应的是 DSCP 中的 **CS6 (Class Selector 6)**。

如果你在支持 DSCP 的设备上配置，对应的数值计算如下：

- **IP Precedence 6** = 二进制 `110` (前三位) + `000` (后三位) = **48** (十进制)。

### 验证命令

如果你配置了该命令，可以通过以下方式查看 SSH 流量是否被打上了正确的标签：

Bash

```
Router# show ip interface <interface> | include IP precedence
# 或者直接抓包查看 TOS 字段
```

