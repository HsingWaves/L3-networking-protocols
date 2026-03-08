

```
(config)#logging buffered 16000 informational
```

------

# 1 题目在问什么

题目说：

> R1 does not hold enough log messages

意思是：

**日志缓存太小，保存不了足够多的日志。**

要解决：

✔ **增加 logging buffer size**

------

# 2 Cisco logging buffered 结构

命令格式：

```
logging buffered <size> <level>
```

例子：

```
logging buffered 16000 informational
```

含义：

| 参数          | 含义                 |
| ------------- | -------------------- |
| 16000         | buffer 大小（bytes） |
| informational | 记录日志级别         |

------

# 3 Logging Level

Cisco syslog level：

| Level | 名字          |
| ----- | ------------- |
| 0     | emergencies   |
| 1     | alerts        |
| 2     | critical      |
| 3     | errors        |
| 4     | warnings      |
| 5     | notifications |
| 6     | informational |
| 7     | debugging     |

------

# 4 关键理解

如果设置：

```
critical
```

只会记录：

```
0
1
2
```

日志非常少。

------

如果设置：

```
informational
```

会记录：

```
0
1
2
3
4
5
6
```

日志 **多得多**。

------

# 5 题目需求

题目要：

```
更多日志
```

所以需要：

1️⃣ **更大的 buffer**
2️⃣ **更详细的日志级别**

------

# 6 看选项

### A

```
logging buffered 4096 critical
```

buffer 小
日志级别低

❌ 不行

------

### B

```
logging buffered 16000 critical
```

buffer 大了

但日志级别 **critical**

日志仍然少

❌

------

### C

```
logging buffered 16000 informational
```

✔ buffer 更大
✔ 日志更详细

✅ 正确

------

### D

```
logging buffered 4096 informational
```

日志级别对
但 buffer 小

题目说 **not hold enough logs**

❌

------

# 7 最终配置

```
conf t
logging buffered 16000 informational
```

查看：

```
show logging
```

------

# ENARSI 秒记口诀

看到题目：

```
not enough log messages
```

直接想到：

```
logging buffered <bigger size> informational
```

------

