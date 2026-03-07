**SCP = Secure Copy Protocol**

是一个 **基于 SSH 的安全文件传输协议**，用于在设备之间 **安全复制文件**。📦🔐

------

# 1 一句话理解

**SCP = 用 SSH 加密的 copy**

相当于：

```
copy file over SSH
```

所以它比这些协议更安全：

| 协议    | 是否加密   |
| ------- | ---------- |
| TFTP    | ❌ 不加密   |
| FTP     | ❌ 明文     |
| **SCP** | ✅ SSH 加密 |
| SFTP    | ✅ SSH 加密 |

------

# 2 工作原理

SCP 实际上 **跑在 SSH 之上**：

```
SCP
  ↓
SSH (TCP 22)
  ↓
IP
```

所以必须先有：

```
SSH enabled
RSA key
VTY login
```

------

# 3 常见使用场景

### Linux → Cisco

上传文件：

```bash
scp script.py admin@192.168.1.1:script.py
```

------

### Cisco → Linux

```bash
copy flash:test.py scp:
```

------

### Linux → Linux

```bash
scp file.txt user@server:/home/user/
```

------

# 4 Cisco 必须开启 SCP server

默认 **Cisco 不允许 SCP 上传**。

需要开启：

```bash
conf t
ip scp server enable
```

否则就会看到题目里的错误：

```
Administratively disabled
```

------

# 5 SCP vs SFTP

| 特性       | SCP       | SFTP         |
| ---------- | --------- | ------------ |
| 基于 SSH   | ✅         | ✅            |
| 功能       | 简单 copy | 完整文件系统 |
| Cisco 支持 | 很常见    | 新设备支持   |

------

# 6 CCNP ENARSI 考试口诀

看到：

```
scp
Administratively disabled
```

马上想到：

```
ip scp server enable
```

------

✅ **一句话记忆**

> **SCP = SSH 加密文件复制协议**

------

下面是一张 **Cisco 文件传输协议速查表（ENARSI / CCNP 常考）** 📡📦

| 协议     | 端口   | 是否加密   | 认证         | 常见用途                | Cisco 是否默认开启 |
| -------- | ------ | ---------- | ------------ | ----------------------- | ------------------ |
| **TFTP** | UDP 69 | ❌ 不加密   | ❌ 无认证     | IOS image / config 传输 | 不需要启用         |
| **FTP**  | TCP 21 | ❌ 明文     | 用户名密码   | 文件传输                | 需要 FTP server    |
| **SCP**  | TCP 22 | ✅ SSH 加密 | SSH 用户认证 | 安全文件复制            | ❌ 默认关闭         |
| **SFTP** | TCP 22 | ✅ SSH 加密 | SSH 用户认证 | 完整文件系统操作        | IOS XE 支持        |

------

# Cisco 常用命令

## 1 TFTP（最常见）

复制 IOS 或 config

```bash
copy flash: tftp:
copy tftp: flash:
copy running-config tftp:
```

优点

- 简单
- 几乎所有设备支持

缺点

- **完全不安全**

------

# 2 FTP

```bash
copy flash: ftp:
copy ftp: flash:
```

需要：

```bash
ip ftp username USER
ip ftp password PASS
```

缺点

- 明文密码

------

# 3 SCP（安全）

必须先启用：

```bash
conf t
ip scp server enable
```

Linux 上传：

```bash
scp script.py admin@192.168.1.1:script.py
```

Cisco 下载：

```bash
copy scp: flash:
```

------

# 4 SFTP（较新）

IOS XE 支持：

```bash
copy sftp: flash:
```

例如：

```bash
copy sftp://user@server/file flash:
```

------

# 常见考试陷阱

| 题目提示                            | 答案                      |
| ----------------------------------- | ------------------------- |
| **Administratively disabled (scp)** | `ip scp server enable`    |
| SSH 不工作                          | `crypto key generate rsa` |
| SSH 登录失败                        | `transport input ssh`     |
| TFTP timeout                        | ACL / firewall            |

------

# ENARSI 一句话记忆

```
TFTP  = 快但不安全
FTP   = 有认证但明文
SCP   = SSH copy
SFTP  = SSH file system
```

------

#  如果要加密怎么办

有三个替代方案：

| 协议     | 加密方式 | 端口     |
| -------- | -------- | -------- |
| **FTPS** | SSL/TLS  | 990 / 21 |
| **SFTP** | SSH      | 22       |
| **SCP**  | SSH      | 22       |

------

# 4 Cisco 网络设备常见安全替代

| 不安全 | 安全替代 |
| ------ | -------- |
| TFTP   | SCP      |
| FTP    | SFTP     |
| Telnet | SSH      |

------

# 5 ENARSI 记忆表

| 协议 | 加密  |
| ---- | ----- |
| TFTP | ❌     |
| FTP  | ❌     |
| SCP  | ✅ SSH |
| SFTP | ✅ SSH |
| FTPS | ✅ TLS |