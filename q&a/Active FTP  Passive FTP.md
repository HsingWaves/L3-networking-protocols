

------

# Active FTP vs Passive FTP + ACL 判定表

| 项目           | Active FTP        | Passive FTP           |
| -------------- | ----------------- | --------------------- |
| 控制连接       | Client → Server   | Client → Server       |
| 控制端口       | TCP 21            | TCP 21                |
| 数据连接谁发起 | **Server 发起**   | **Client 发起**       |
| 数据端口       | Server **TCP 20** | Server **随机 >1023** |
| 数据连接方向   | Server → Client   | Client → Server       |
| NAT 友好性     | ❌ 差              | ✅ 好                  |
| 现代网络常用   | ❌ 很少            | ✅ 默认                |

------

# 连接流程对比

## Active FTP

```
Control connection
Client -----> Server
random port  TCP 21

Data connection
Server -----> Client
TCP 20       random port
```

特点：

- 数据连接 **服务器主动**
- 使用 **port 20**

------

## Passive FTP

```
Control connection
Client -----> Server
random port  TCP 21

Server reply
227 Entering Passive Mode (port 50000)

Data connection
Client -----> Server
random port  TCP 50000
```

特点：

- 数据连接 **客户端主动**
- **服务器随机高端口**

------

# ACL 配置关键差异（考试重点）

## Active FTP 需要允许

```
Client -> Server TCP 21
Server -> Client TCP 20
```

示例

```
permit tcp client any server eq 21
permit tcp server eq 20 client any
```

------

## Passive FTP 需要允许

```
Client -> Server TCP 21
Client -> Server TCP >1023
```

示例

```
permit tcp client any server eq 21
permit tcp client any server range 1024 65535
```

------

# 本题为什么错

现在 ACL 只允许：

```
ftp      (21)
ftp-data (20)
```

但 **Passive FTP data port 是随机高端口**。

所以：

```
FTP client → server high port
```

被 ACL **拒绝了**。

------

# ENARSI 秒杀口诀

```
Active FTP
server -> client
port 20

Passive FTP
client -> server
random port (>1023)
```

------

# Cisco 常见考试陷阱

| 陷阱                         | 实际原因                 |
| ---------------------------- | ------------------------ |
| FTP login 成功但无法传输文件 | data port 被 ACL block   |
| passive FTP fail             | 没允许 server high ports |
| active FTP fail              | 没允许 server port 20    |

------

