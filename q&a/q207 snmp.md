------

# 为什么会出现这个问题

SNMP 监控接口流量时，用的是 **ifIndex**：

```
IF-MIB::ifIndex
```

例如：

```
Gi0/1 → ifIndex 5
Gi0/2 → ifIndex 6
```

监控系统（NMS）通常保存的是：

```
device + ifIndex
```

而不是接口名字。

------

# 升级 Compact Flash 后发生什么

当设备：

- reload
- 升级 flash
- 硬件变化

Cisco 设备会 **重新生成 ifIndex**。

例如：

```
以前
Gi0/1 → ifIndex 5
Gi0/2 → ifIndex 6

升级后
Gi0/1 → ifIndex 12
Gi0/2 → ifIndex 13
```

于是：

```
NMS 查询 ifIndex 5
→ 不存在
→ 监控断掉
```

------

# 解决方法

让 **ifIndex 永久保持不变**：

```
snmp-server ifindex persist
```

作用：

```
保存 interface index
在 reload 后保持一致
```

这样：

```
Gi0/1 永远是同一个 ifIndex
```

NMS 就不会乱。

------

# 为什么其他选项错

### B ❌

```
snmp-server ifindex permanent
```

Cisco 没这个命令。

------

### C ❌

```
snmp ifindex persist
```

缺少 `server`，语法错误。

------

### D ❌

```
snmp ifindex permanent
```

同样不存在。

------

# ENARSI 必记一句话

看到题目出现：

```
SNMP monitoring broken
interface index changed
```

立刻想到：

```
snmp-server ifindex persist
```

很好，这里给你一份 **ENARSI SNMP 8个高频坑总结（300-410 必考点）**。
基本上 SNMP 题目 80% 都在这几个坑里。

------

# ENARSI SNMP 8 个高频坑总结

------

# 1️⃣ ifIndex 改变导致监控失效

最经典题。

症状：

```
SNMP working
but monitoring graphs broken
interface indexes changed
```

原因：

```
设备 reload / upgrade
→ interface index 重新分配
```

解决：

```cisco
snmp-server ifindex persist
```

记忆：

```
ifIndex changed → persist
```

------

# 2️⃣ SNMP ACL 没绑定

题目会给：

```cisco
access-list 11 permit 10.1.1.10
```

但 SNMP 配置：

```cisco
snmp-server community public RO 20
```

真正生效的是：

```
ACL 20
```

不是 ACL 11。

口诀：

```
Feature → reference ACL → entries
```

------

# 3️⃣ SNMPv3 Engine ID mismatch

日志：

```
Unknown Engine ID
authentication failure
```

说明：

```
NMS engineID ≠ device engineID
```

排查：

```cisco
show snmp user
debug snmp packet
```

------

# 4️⃣ SNMP community 权限错误

常见：

```cisco
snmp-server community public RO
```

但 NMS 要 **set OID**。

结果：

```
SNMP GET OK
SNMP SET FAIL
```

解决：

```
RW community
```

------

# 5️⃣ SNMP View 限制 OID

SNMPv3 结构：

```
view
group
user
```

如果 view 限制：

```cisco
snmp-server view LIMITED iso included
```

但某些 OID 不在 view 里：

```
NMS query fail
```

------

# 6️⃣ SNMP Trap 不发送

常见原因：

缺少：

```cisco
snmp-server enable traps
```

或者：

```cisco
snmp-server host 10.1.1.10 version 2c public
```

没配置。

结构必须完整：

```
enable traps
+
snmp-server host
```

------

# 7️⃣ SNMP version mismatch

设备：

```
SNMPv3
```

NMS：

```
SNMPv2c
```

结果：

```
authentication failure
```

或者：

```
no response
```

------

# 8️⃣ SNMP source interface 错误

设备有多个 IP。

NMS 允许：

```
10.1.1.1
```

但设备发送 trap 使用：

```
192.168.1.1
```

解决：

```cisco
snmp-server trap-source Loopback0
```

或

```cisco
snmp-server source-interface informs Loopback0
```

------

# SNMPv3 架构（考试必须理解）

```
SNMP
│
├── View
│
├── Group
│
├── User
│
└── EngineID
```

关系：

```
user → group → view
```

------

# ENARSI SNMP 最重要口诀

看到 SNMP 故障：

```
1 看 ACL
2 看 version
3 看 view
4 看 engineID
5 看 ifIndex
```

# 

ENARSI SNMP 题 **几乎一定会考其中 2 个**：

最常见的是：

```
1 ifindex persist
2 ACL mismatch
3 EngineID mismatch
```

------

