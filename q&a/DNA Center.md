下面是一张 **Cisco DNA Center Architecture（ENARSI 考试版速查图）**，把考试里最容易混淆的几个概念一次整理清楚。

------

# Cisco DNA Center Architecture（Cluster）

```text
                 Cisco DNA Center Cluster
        ┌─────────────────────────────────────┐
        │                                     │
        │        Service Distribution          │
        │                                     │
        │   ┌───────────┐   ┌───────────┐     │
        │   │  Node 1   │   │  Node 2   │     │
        │   │ (Primary) │   │ (Secondary)│    │
        │   │           │   │           │     │
        │   │Automation │   │Assurance  │     │
        │   │Inventory  │   │Analytics  │     │
        │   └───────────┘   └───────────┘     │
        │           │           │              │
        │           │           │              │
        │        ┌───────────┐                │
        │        │  Node 3   │                │
        │        │ (Worker)  │                │
        │        │Telemetry  │                │
        │        │AI/ML      │                │
        │        └───────────┘                │
        │                                     │
        └─────────────────────────────────────┘
```

------

# 1 Cluster Node 类型

| Node               | 作用                 |
| ------------------ | -------------------- |
| **Primary node**   | 管理 cluster         |
| **Secondary node** | HA / service hosting |
| **Worker node**    | 额外计算资源         |

一个 cluster 通常：

```text
3 nodes
```

------

# 2 Service Distribution

默认情况下：

```text
所有服务运行在 Primary node
```

启用后：

```text
服务自动分布到不同节点
```

例如：

| Node  | Service    |
| ----- | ---------- |
| Node1 | Automation |
| Node2 | Assurance  |
| Node3 | Telemetry  |

启用位置：

```text
System → System 360 → Enable Service Distribution
```

考试常见题：

```text
3 node cluster
services on one node
```

答案：

```text
Enable service distribution
```

------

# 3 DNA Center 两种网络链路

DNA Center 有两个重要网络：

| Network             | 作用         |
| ------------------- | ------------ |
| **Enterprise link** | 管理网络设备 |
| **Cluster link**    | 节点之间通信 |

------

## Enterprise link

```text
DNA Center → Switch / Router / AP
```

用途：

- discovery
- provisioning
- telemetry

------

## Cluster link

```text
Node1 ↔ Node2 ↔ Node3
```

用途：

- service sync
- database replication
- cluster heartbeat

如果 cluster link down：

```text
node communication failure
```

------

# 4 System 360 页面

System 360 是 **DNA Center 运维页面**。

可以看到：

| 项目                 | 说明     |
| -------------------- | -------- |
| cluster health       | 集群健康 |
| node health          | 节点状态 |
| service distribution | 服务分布 |
| disk / CPU           | 资源使用 |

------

# 5 常见 ENARSI 题型

### 题型 1

```text
3 node cluster
services on primary node
```

答案：

```text
Enable Service Distribution
```

------

### 题型 2

```text
cluster node unreachable
```

原因：

```text
cluster link failure
```

------

### 题型 3

```text
device discovery fail
```

原因：

```text
enterprise link problem
```

------

# 6 DNA Center 功能模块

最常见的服务：

| Service        | 作用                |
| -------------- | ------------------- |
| **Automation** | config push         |
| **Assurance**  | monitoring          |
| **Inventory**  | device database     |
| **Telemetry**  | streaming telemetry |
| **Analytics**  | network analytics   |

------

# 7 一句话记忆（考试版）

```text
Enterprise link = 管设备
Cluster link    = 节点通信
Service Distribution = 服务负载均衡
System 360 = 运维监控
```

------



- 很好，我们把 **Cisco DNA Center + SDA (Software-Defined Access) 架构**整理成一张 **ENARSI / CCNP 高频理解图**。这部分很多题是围绕这些组件出题的。

  ------

  # Cisco DNA Center + SDA Architecture

  ```text
                             +--------------------+
                             |   Cisco DNA Center |
                             |  (Automation Brain)|
                             +----------+---------+
                                        |
                                        | API / Policy
                                        |
                             +----------v----------+
                             |       Cisco ISE      |
                             | (Identity / Policy)  |
                             +----------+-----------+
                                        |
                           Authentication / Policy
                                        |
          ---------------------------------------------------------
                             Fabric Network
          ---------------------------------------------------------
  
              +------------------+
              |   Border Node     |
              | (Fabric Edge →    |
              |  External world)  |
              +--------+----------+
                       |
                       | VXLAN
                       |
          +------------+-------------+
          |                          |
  +-------v-------+          +-------v-------+
  |   Fabric Edge  |          |   Fabric Edge |
  | (Access switch)|          | (Access switch)|
  +-------+--------+          +--------+------+
          |                            |
          |                            |
       Clients                      Clients
   (PC / Phone / IoT)          (PC / Phone / IoT)
  
                  +----------------------------+
                  |   Control Plane Node       |
                  |        (LISP Map Server)   |
                  +----------------------------+
  ```

  ------

  # 1 Cisco DNA Center（大脑）

  作用：

  | 功能         | 说明         |
  | ------------ | ------------ |
  | Automation   | 自动部署网络 |
  | Assurance    | 网络监控     |
  | Provisioning | 配置设备     |
  | Policy       | 推送策略     |

  DNA Center 控制：

  ```text
  Fabric nodes
  Edge
  Border
  Control plane
  ```

  ------

  # 2 Cisco ISE（身份与策略）

  ISE 负责：

  ```text
  身份认证
  设备分类
  访问控制
  ```

  例如：

  | 用户     | VLAN          |
  | -------- | ------------- |
  | Employee | Corp network  |
  | Guest    | Guest network |
  | IoT      | IoT segment   |

  ISE 与 DNA Center 结合：

  ```text
  Policy based access
  ```

  ------

  # 3 Fabric Edge Node

  位置：

  ```text
  接入交换机
  ```

  作用：

  ```text
  用户接入点
  ```

  功能：

  | 功能                | 技术    |
  | ------------------- | ------- |
  | 用户接入            | 802.1X  |
  | VXLAN encapsulation | overlay |
  | policy enforcement  | SGT     |

  ------

  # 4 Control Plane Node

  作用：

  ```text
  LISP Map Server
  ```

  管理：

  ```text
  IP ↔ Location mapping
  ```

  例如：

  ```text
  Host 10.10.10.5
  在 Edge SW2
  ```

  记录：

  ```text
  Endpoint → Edge node
  ```

  ------

  # 5 Border Node

  作用：

  ```text
  Fabric → 外部网络
  ```

  连接：

  ```text
  Internet
  Data center
  WAN
  ```

  功能：

  | 功能                 | 说明              |
  | -------------------- | ----------------- |
  | VXLAN termination    | overlay exit      |
  | Route redistribution | fabric ↔ external |

  ------

  # 6 SDA Overlay 技术

  SDA 使用：

  ```text
  VXLAN + LISP
  ```

  ------

  ### VXLAN

  用途：

  ```text
  数据封装
  ```

  例：

  ```text
  Client traffic
  → VXLAN encapsulation
  → Fabric network
  ```

  ------

  ### LISP

  用途：

  ```text
  Endpoint location mapping
  ```

  功能：

  ```text
  Where is endpoint located?
  ```

  ------

  # 7 数据流流程

  用户访问服务器：

  ```text
  Client
  ↓
  Fabric Edge
  ↓ VXLAN encapsulation
  Fabric network
  ↓
  Border Node
  ↓
  External network
  ```

  ------

  # 8 SDA 的优势

  | 传统网络              | SDA                         |
  | --------------------- | --------------------------- |
  | VLAN                  | Virtual Network             |
  | 手动配置              | 自动化                      |
  | IP-based segmentation | Identity-based segmentation |

  ------

  # 9 ENARSI 高频考点

  ### 1 Fabric Edge

  ```text
  用户接入交换机
  ```

  ------

  ### 2 Border Node

  ```text
  Fabric ↔ External networks
  ```

  ------

  ### 3 Control Plane Node

  ```text
  LISP map server
  ```

  ------

  ### 4 DNA Center

  ```text
  automation + management
  ```

  ------

  # 10 一句话记忆（考试版）

  ```text
  DNA Center = Brain
  ISE        = Identity
  Edge       = Access
  Control    = LISP database
  Border     = Exit point
  ```

  ------

  

