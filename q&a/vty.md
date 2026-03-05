这个问题其实是在理解 **Cisco “line” 的作用范围**。关键点是：

> **line console、line vty、line aux 是不同的终端类型，各自独立生效。**

所以改 **VTY** 的配置，不会影响 **console**。

------

# 1️⃣ Cisco 三种登录线路

| line 类型   | 代表什么 | 怎么进入      |
| ----------- | -------- | ------------- |
| **console** | 本地串口 | 插 console 线 |
| **vty**     | 远程登录 | SSH / Telnet  |
| **aux**     | modem    | 很少用        |

------

# 2️⃣ 你的场景

题目说：

> engineer 从 **console** 登录到 R1

所以当前 session 是：

```
line console 0
```

然后在 console 上执行：

```
ssh -l cisco 192.168.12.2
```

这叫：

**outbound connection**

------

# 3️⃣ console 当前配置

```
line con 0
 transport output none
```

意思：

```
console 不允许发起任何连接
```

所以 SSH 被拒绝：

```
% ssh connections not permitted from this terminal
```

------

# 4️⃣ 为什么改 VTY 没用

VTY 只控制：

```
别人远程连进来
```

例如：

```
PC ----SSH----> Router
```

配置：

```
line vty 0 4
 transport input ssh
```

控制的是：

> 是否允许别人 SSH 到 router

而你现在是：

```
console ----SSH----> 另一台 router
```

这属于：

```
transport output
```

而且是在 **console line**

------

# 5️⃣ 一个图秒懂

### 情况1（VTY）

```
PC ----SSH----> Router
          |
       line vty
```

### 情况2（console）

```
Laptop --console--> Router ----SSH----> Router2
           |
        line console
```

所以要改：

```
line console 0
transport output ssh
```

------

# 6️⃣ Cisco 最容易考的区别

| 命令             | 含义           |
| ---------------- | -------------- |
| transport input  | 允许别人连进来 |
| transport output | 允许设备连出去 |
| line vty         | 远程登录       |
| line console     | 本地登录       |

------

# 

```
SSH / Telnet 进设备 → VTY
设备 SSH / Telnet 出去 → console / vty 的 transport output
```

------

