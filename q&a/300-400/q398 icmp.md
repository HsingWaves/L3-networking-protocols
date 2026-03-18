**指定源接口 (Option A)：** `icmp-echo 8.8.8.8 source-interface GigabitEthernet0/1`

### 深入解析 IP SLA 中的 ICMP 探测

在网络运维中，**IP SLA (Service Level Agreement)** 配合 **ICMP Echo** 是最常用的“活体检测”技术。它的核心逻辑是：**“如果我Ping不通目的地，我就认为这条线路挂了，立刻切换路由。”**

------

### 1. ICMP 在这里的作用

在 IP SLA 配置中，ICMP Echo 充当了**探测针**。

- **主动探测：** 路由器定期主动发送 ICMP 请求包（类似 Ping）。
- **性能监控：** 它不仅看通不通，还记录 **RTT（往返时间）**、**抖动（Jitter）** 和 **丢包率**。
- **联动路由：** 配合 `track` 命令，它可以动态地把静态路由从路由表中“踢出去”或“拉回来”。

------

### 2. 具体命令拆解

我们拿你刚才题目中的配置来逐行解剖：

Bash

```
ip sla 1                         # 创建一个编号为 1 的探测实例
 icmp-echo 8.8.8.8               # 探测类型：向 8.8.8.8 发送 ICMP Echo (Ping)
 threshold 1000                  # 阈值：如果 RTT 超过 1000ms，判定为“不达标”
 timeout 2000                    # 超时：等了 2000ms 还没回包，判定为“彻底断开”
 frequency 5                     # 频率：每 5 秒探测一次
ip sla schedule 1 life forever start-time now  # 激活计划：从现在开始，永久运行
```

**关键补充（解决 Flapping 的命令）：**

- `icmp-echo 8.8.8.8 source-interface G0/1`
  - **作用：** 强制这个 Ping 包必须从 G0/1 接口发出去。这样即使默认路由切到了 ISP2，这个探测包也不会“绕路”去走 ISP2，从而避免了我们刚才讨论的震荡（Flapping）问题。

------

### 3. 如何配置（完整联动流程）

要把这个探测应用到实际的 ISP 切换中，需要三步走：

#### 第一步：配置 SLA 探测器

Bash

```
ip sla 1
 icmp-echo 8.8.8.8 source-interface GigabitEthernet0/1
 frequency 5
ip sla schedule 1 life forever start-time now
```

#### 第二步：配置 Track 跟踪器

SLA 只是在后台默默地 Ping，它不能直接修改路由表。我们需要一个“中间人”——**Track**。

Bash

```
track 1 ip sla 1 state
# 作用：如果 IP SLA 1 探测成功，Track 1 就是 Up；否则就是 Down。
```

#### 第三步：联动静态路由

这是最后一步，把路由和 Track 绑定。

Bash

```
# 主线路：关联 track 1。如果 track 1 为 Down，这条路由会从路由表中消失。
ip route 0.0.0.0 0.0.0.0 203.0.113.1 track 1

# 备线路：设置更高的管理距离（AD=10）。平时它在数据库里待命，主路由消失后它才上位。
ip route 0.0.0.0 0.0.0.0 198.51.100.1 10
```

------

### 4. 常用检查命令

当你配置完后，用这两条命令来排错：

1. **`show ip sla statistics`**
   - 查看最近一次探测是 `Success` 还是 `Timeout`。
2. **`show track`**
   - 查看跟踪器的状态。如果是 `Down`，你的主路由应该已经从 `show ip route` 中消失了。

------

### 总结

ICMP SLA 就像是一个**“心跳监测仪”**。如果没有 `source-interface`，这个监测仪可能会因为“心脏（ISP1）”停跳后通过“人工肺（ISP2）”学到的信号而误以为心脏还在跳，导致频繁在手术台（路由表）上切换方案。

