很好 👍 我给你做一个 **完整总结 + 高频考试速记表**，这一页你记住，VRF 和接口清 IP 的坑基本通关。

------

# 🧠 一、VRF 题目核心总结

### 1️⃣ 为什么要用 VRF？

因为：

- Math 和 Science 使用 **相同网段**
- 同一个 routing table 里不能存在重复前缀
- 又要求两边隔离

所以必须：

```bash
vrf definition Science
interface e0/2
 vrf forwarding Science
 ip address ...
```

本质：

> 一台物理路由器 = 多台逻辑路由器
> 每个 VRF = 一张独立 routing table

------

### 2️⃣ 真正让它们互通的是什么？

不是 IP
不是接口

而是：

```bash
vrf forwarding Science
```

因为：

- e0/2 和 e0/3 被放进同一个 routing table
- 同一张 routing table 内天然互通

------

### 3️⃣ 为什么 C 错？

如果顺序写成：

```bash
ip address ...
vrf forwarding Science
```

IOS 会：

🔥 清掉 IP

接口最终没有 IP
路由不存在
配置逻辑错误

------

# 🚨 二、Cisco IOS 5 大“清 IP”场景速记表

这是考试非常喜欢考的。

------

## ① 改 VRF（最经典）

```bash
vrf forwarding XXX
```

✔ 100% 清 IP
✔ 必须重新配置

------

## ② Routed ↔ Switchport 切换

```bash
no switchport
switchport
```

改变接口类型时：

✔ 清 IP
✔ 清子接口

------

## ③ 改 encapsulation（子接口）

```bash
encapsulation dot1q 10
```

改变封装：

✔ 清 IP
✔ 需要重新配置

因为二层身份改变

------

## ④ 改 tunnel mode（隧道接口）

```bash
interface tunnel0
 tunnel mode gre ip
```

改变 tunnel 类型时：

✔ 会清 IP

------

## ⑤ 改 interface type（某些平台）

例如：

- 改 port-channel 成员
- 改某些模块接口模式

在部分 IOS-XE / NX-OS 平台：

✔ 可能清 IP

------

# 🎯 统一理解模型

所有“清 IP”的本质是：

> 接口的“身份”发生了改变

包括：

- routing table 改变（VRF）
- 二层 / 三层角色改变
- 封装方式改变
- 隧道类型改变

IOS 的设计哲学是：

> 为避免逻辑冲突 → 自动删除 IP → 让你重新配置

------

# 🧠 超级记忆口诀

看到下面这些词就提高警惕：

```
vrf
switchport
encapsulation
tunnel mode
port-channel
```

= 小心 IP 被清

------

# 🏆 三、考试答题思路总结

看到题目出现：

- IP 重复
- 不同部门
- 同一核心路由器
- 要隔离

直接联想：

```
VRF
```

看到：

- 顺序错误
- 先配 IP 再 vrf forwarding

直接判断：

```
IP 会被清 → 错
```

------

