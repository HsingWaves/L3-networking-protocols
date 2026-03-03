# Control Plane Policing

作用：

> 保护路由器 CPU 不被打爆

------

# 🚨 为什么需要 CoPP？

因为：

数据流 → 目的 IP 是路由器自身
 → 必须进入 CPU

攻击场景：

- BGP flood
- SSH brute force
- ICMP flood
- SNMP flood

如果不限制：

CPU 100%
 邻居 down
 SSH 卡死
 网络瘫痪

------

# 🏗 正常数据流 vs CPU 流量

## 数据平面流量

```
PC1 → Router → Router → PC2
(ASIC 转发)
```

不进 CPU

------

## 控制平面流量

```
PC → Router (目的IP=Router)
        ↓
      CPU
```

必须进 CPU

------

# 🔥 CoPP 工作流程

```
流量进入接口
        ↓
判断目的IP是否为本机
        ↓
是 → 进入 control-plane
        ↓
匹配 class-map
        ↓
执行 police
        ↓
transmit / drop
```

------

# 🧨 CoPP 典型配置结构

```
class-map match-any BGP
 match protocol bgp

policy-map COPP
 class BGP
  police cir 1000k bc 1500
   conform-action transmit
   exceed-action drop

control-plane
 service-policy input COPP
```

------

# 📊 CoPP vs QoS 区别

| 项目     | 数据平面QoS | CoPP          |
| -------- | ----------- | ------------- |
| 作用对象 | 用户流量    | 发往CPU的流量 |
| 目的     | 控制带宽    | 保护CPU       |
| 常用技术 | shaping     | policing      |
| 是否必须 | 不是        | 企业强烈建议  |

### 1️⃣ 数据正常 + 管理异常

👉 查 CoPP

------

### 2️⃣ BGP/OSPF 邻居 flap

👉 可能 CoPP 限速过低

------

### 3️⃣ show policy-map vs show policy-map control-plane

- 前者：接口 QoS
- 后者：CPU QoS

------

### 4️⃣ exceed transmit vs exceed drop

| exceed transmit | 不限速   |
| --------------- | -------- |
| exceed drop     | 真正限速 |

# 

> 数据平面是“帮别人转发”
>  控制平面是“决定怎么转发”
>  CoPP 是“保护大脑不被打爆”