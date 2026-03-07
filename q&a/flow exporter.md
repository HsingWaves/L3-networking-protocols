# 一、Flow Exporter 是干嘛的

**Flow Exporter = 定义 NetFlow 数据 “发到哪里、怎么发” 的组件。**

简单说：

> **Flow Exporter 负责把 NetFlow 数据发送到 NMS / Collector。**

它定义：

| 参数               | 作用                       |
| ------------------ | -------------------------- |
| destination IP     | 发到哪个 NetFlow Collector |
| source interface   | NetFlow 包使用哪个源 IP    |
| transport protocol | UDP                        |
| port               | 例如 2055                  |
| NetFlow version    | v5 / v9 / IPFIX            |

------

# 二、一句话记忆（非常重要）

```
Flow Exporter = NetFlow 数据发送策略
```

或者更直白：

```
Exporter = where to send flows
```

------

# 三、完整 NetFlow 架构

Cisco Flexible NetFlow 有 **四层结构**：

```
Flow Record  →  Flow Monitor  →  Flow Exporter  →  Collector
```

含义：

| 组件          | 作用                   |
| ------------- | ---------------------- |
| Flow Record   | 定义采集哪些字段       |
| Flow Monitor  | 绑定 record + exporter |
| Flow Exporter | 定义发送目标           |
| Interface     | 在接口启用监控         |

------

# 四、举个完整例子

### 1️⃣ Flow Record（收集什么）

```
flow record NETFLOW_RECORD
 match ipv4 source address
 match ipv4 destination address
 collect counter packets
 collect counter bytes
```

------

### 2️⃣ Flow Exporter（发到哪里）

```
flow exporter NETFLOW_EXPORTER
 destination 10.221.10.11
 source Loopback0
 transport udp 2055
 version 9
```

意思：

```
NetFlow → UDP 2055 → 10.221.10.11
```

------

### 3️⃣ Flow Monitor（绑定）

```
flow monitor NETFLOW_MONITOR
 record NETFLOW_RECORD
 exporter NETFLOW_EXPORTER
```

------

### 4️⃣ 在接口启用

```
interface Gi0/0
 ip flow monitor NETFLOW_MONITOR input
```

------

# 五、现实网络中的位置

```
Router/Switch
     │
     │ NetFlow UDP 2055
     ▼
NetFlow Collector
(SolarWinds / Stealthwatch / NMS)
```

Flow Exporter 就是定义：

```
UDP packet
source IP = router
destination IP = collector
```

------

# 六、ENARSI 常见考点

看到这些关键词基本就是 **Flow Exporter 问题**：

| 题目关键词            | 看哪里                    |
| --------------------- | ------------------------- |
| collector 收不到 flow | exporter destination      |
| collector 识别错误    | exporter source interface |
| UDP port 错           | exporter transport        |
| NetFlow version 错    | exporter version          |

------

# 七、考试最简单理解

记住这一句就够：

```
Flow Record  = what to collect
Flow Monitor = glue
Flow Exporter = where to send
```