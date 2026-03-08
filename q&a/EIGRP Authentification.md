 **EIGRP + authentication + key chain**

先检查三件事：

1️⃣ authentication mode 是否一致

```
ip authentication mode eigrp 100 md5
```

2️⃣ key-chain name 是否一致

```
ip authentication key-chain eigrp 100 cisco
```

3️⃣ **lowest key-id 是否一致**

| Protocol  | Authentication Type | Key Command                             | 特点                  | 常见故障                  |
| --------- | ------------------- | --------------------------------------- | --------------------- | ------------------------- |
| **EIGRP** | MD5                 | `ip authentication mode eigrp AS md5`   | 使用 **key-chain**    | key-id 不一致             |
| **OSPF**  | Simple / MD5        | `ip ospf authentication message-digest` | 在 **接口上配置**     | key mismatch              |
| **BGP**   | TCP MD5             | `neighbor x password`                   | 在 **neighbor上配置** | password mismatchProtocol |

给你一张 **CCNP ENARSI 路由认证秒杀表**（考试非常高频）。
记住这张基本所有题都能秒。

------

# 🔐 CCNP ENARSI Routing Authentication Cheat Sheet

| Protocol  | Authentication Type | Key Command                             | 特点                  | 常见故障          |
| --------- | ------------------- | --------------------------------------- | --------------------- | ----------------- |
| **EIGRP** | MD5                 | `ip authentication mode eigrp AS md5`   | 使用 **key-chain**    | key-id 不一致     |
| **OSPF**  | Simple / MD5        | `ip ospf authentication message-digest` | 在 **接口上配置**     | key mismatch      |
| **BGP**   | TCP MD5             | `neighbor x password`                   | 在 **neighbor上配置** | password mismatch |

------

# 1️⃣ EIGRP Authentication

配置：

```bash
key chain cisco
 key 1
  key-string 123

interface s0/0
 ip authentication mode eigrp 100 md5
 ip authentication key-chain eigrp 100 cisco
```

关键点：

| 项目           | 必须一致 |
| -------------- | -------- |
| AS number      | ✔        |
| key-chain name | ✔        |
| key-string     | ✔        |

⚠️ **考试重点**

```text
EIGRP uses the LOWEST key-id
```

如果：

```
R1
key 2 abc

R2
key 1 123
key 2 abc
```

结果：

```
R1 send → key2
R2 expect → key1
❌ authentication failure
```

------

# 2️⃣ OSPF Authentication

三种模式：

| 类型            | 命令                                    |
| --------------- | --------------------------------------- |
| None            | 默认                                    |
| Simple password | `ip ospf authentication`                |
| MD5             | `ip ospf authentication message-digest` |

示例：

```bash
interface g0/0
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 CISCO
```

要求：

| 必须一致 | 说明 |
| -------- | ---- |
| Area     | ✔    |
| Key ID   | ✔    |
| Password | ✔    |

------

# 3️⃣ BGP Authentication

最简单：

```bash
router bgp 65001
 neighbor 10.1.1.1 password CISCO
```

特点：

| 特点             | 说明              |
| ---------------- | ----------------- |
| TCP MD5          | 保护 BGP session  |
| 不需要 key-chain | 直接 password     |
| 双方必须一致     | 否则 session down |

------

# 🚨 ENARSI 高频陷阱

| 现象                    | 原因              |
| ----------------------- | ----------------- |
| EIGRP neighbors flap    | key-id mismatch   |
| OSPF stuck INIT/EXSTART | auth mismatch     |
| BGP Idle/Active         | password mismatch |

------

# 🧠 ENARSI 秒杀口诀

**EIGRP**

```
AS
key-chain
lowest key-id
```

**OSPF**

```
area
auth type
key-id
password
```

**BGP**

```
neighbor password
```

------

# 

很多 EIGRP 题目日志会出现：

```
DUAL-5-NBRCHANGE
retry limit exceeded
```

99% 是：

```
EIGRP authentication mismatch
```

------

