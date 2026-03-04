很好 👍 这两个是 AAA 的核心理解点，我给你做成 **考试级逻辑图 + 对比表**。

------

# 🔥 一、AAA 全流程决策图（Cisco 真实执行顺序）

当用户尝试登录设备时：

```
                用户尝试登录
                       │
                       ▼
            ┌─────────────────────┐
            │ 1️⃣ Authentication   │
            │ 验证“你是谁？”       │
            └─────────────────────┘
                       │
                       ▼
         成功？ ── 否 ──→ 拒绝登录
                       │
                       是
                       ▼
            ┌─────────────────────┐
            │ 2️⃣ Authorization    │
            │ 决定“你能做什么？”   │
            └─────────────────────┘
                       │
                       ▼
         成功？ ── 否 ──→ 不能进入 EXEC
                       │
                       是
                       ▼
            ┌─────────────────────┐
            │ 3️⃣ Accounting       │
            │ 记录“你做了什么？”   │
            └─────────────────────┘
                       │
                       ▼
                   进入设备
```

------

# 🔥 二、细化每个阶段

## 1️⃣ Authentication（认证）

命令：

```bash
aaa authentication login default local
aaa authentication login default group tacacs+ local
```

决定：

- 用本地？
- 用 RADIUS？
- 用 TACACS+？
- 是否 fallback？

------

## 2️⃣ Authorization（授权）

### 登录后发生

```bash
aaa authorization exec default local
aaa authorization commands 15 default group tacacs+
```

决定：

- 是否允许进入 exec
- privilege level 是多少
- 是否允许执行某条命令

------

## 3️⃣ Accounting（计费）

```bash
aaa accounting exec default start-stop group radius
```

记录：

- 谁登录了
- 执行了什么命令
- 什么时候退出

------

# 🔥 三、Console vs VTY 的关键区别

| 项目                                 | Console       | VTY      |
| ------------------------------------ | ------------- | -------- |
| 默认是否受 AAA 影响                  | ❌ 不受        | ✅ 受     |
| 是否需要 `aaa authorization console` | ✅ 需要        | ❌ 不需要 |
| 是否必须指定 login authentication    | 可选          | 必须     |
| 常见问题                             | EXEC 授权失败 | 无法认证 |

------

# 🔥 四、最重要区别（考试高频）

## Console 特殊规则

如果启用：

```bash
aaa new-model
```

但没有配置：

```bash
aaa authorization console
```

那么：

> console EXEC 授权不会生效

可能出现：

```
AAA/AUTHOR/EXEC: Authorization FAILED
```

------

# 🔥 五、VTY 的行为

VTY 必须：

```bash
line vty 0 4
 login authentication VTY_AUTH
 authorization exec VTY_AUTH
```

否则不会使用自定义 method list。

------

# 🔥 六、完整 AAA 执行顺序（带 method list）

举例：

```bash
aaa authentication login default group tacacs+ local
aaa authorization exec default group tacacs+ local
```

执行顺序：

```
登录
 ↓
尝试 TACACS+
 ↓
TACACS+ 不通？
 ↓
使用 local
 ↓
认证成功
 ↓
尝试 TACACS+ 授权
 ↓
失败？
 ↓
使用 local
 ↓
进入 exec
```

------

# 🔥 七、AAA 典型故障定位逻辑

### 情况 1：无法登录

→ 看 AUTHEN 阶段

```
debug aaa authentication
```

------

### 情况 2：能登录但不能进特权

→ 看 AUTHORIZATION 阶段

```
debug aaa authorization
```

------

### 情况 3：命令执行失败

→ 看 command authorization

```
aaa authorization commands 15 default group tacacs+
```

------

# 🔥 八、Console 常见陷阱

| 配置                                     | 结果             |
| ---------------------------------------- | ---------------- |
| 只配置 authentication                    | 可以登录         |
| 配置 exec authorization 但没启用 console | EXEC 失败        |
| 配置 tacacs 但服务器挂了                 | console 可能锁死 |

------

# 🔥 九、AAA 终极记忆口诀

```
Authentication = 你是谁
Authorization  = 你能干啥
Accounting     = 你干了啥
```

Console 记住：

> 用 AAA 一定考虑 `aaa authorization console`

------

# 🔥 十、考试一眼识别技巧

看到：

```
AAA/AUTHOR/EXEC: Authorization FAILED
```

问自己：

- 是 console 还是 vty？
- 有没有 aaa authorization console？
- method list 是否匹配？

------

很好 👍 这是 **CCNP 必考核心表格**。
我给你做一个**考试级 + 实战级完整版对比表**。

------

# 🔥 AAA vs RADIUS vs TACACS+ 全面对比

| 项目             | AAA             | RADIUS       | TACACS+      |
| ---------------- | --------------- | ------------ | ------------ |
| 是协议吗？       | ❌ 不是          | ✅ 是         | ✅ 是         |
| 是框架还是协议？ | 框架            | 协议         | 协议         |
| 主要用途         | 定义认证架构    | 网络接入认证 | 设备管理认证 |
| 工作层           | Cisco 本地机制  | 应用层       | 应用层       |
| 传输协议         | 依赖底层        | UDP          | TCP          |
| 默认端口         | N/A             | 1812 / 1813  | 49           |
| 是否加密         | N/A             | 只加密密码   | 全部加密     |
| 是否分离 AAA     | N/A             | ❌ 不完全分离 | ✅ 完全分离   |
| 命令级授权       | 通过服务器      | ❌ 不支持     | ✅ 支持       |
| 常用场景         | 路由器 AAA 框架 | 802.1X / VPN | 管理员登录   |

------

#  一、AAA 是什么？

AAA 是：

```text
Authentication
Authorization
Accounting
```

它是：

> Cisco 设备上的控制逻辑框架

AAA 自己不是认证协议。
它调用 RADIUS 或 TACACS+。

------

#  二、RADIUS 特点

## 基本信息

| 项目     | 值         |
| -------- | ---------- |
| 协议     | UDP        |
| 认证端口 | 1812       |
| 计费端口 | 1813       |
| 加密     | 仅密码加密 |
| 常用于   | 网络接入   |

------

## RADIUS 的核心特性

- 认证 + 授权打包在一起
- 不支持细粒度命令授权
- 性能好（UDP）

------

## 常见用途

- 802.1X
- WiFi
- VPN
- ISP 接入

------

#  三、TACACS+ 特点

## 基本信息

| 项目         | 值                    |
| ------------ | --------------------- |
| 协议         | TCP                   |
| 端口         | 49                    |
| 加密         | 整个 payload 全部加密 |
| 分离 AAA     | 完全分离              |
| 支持命令授权 | 支持                  |

------

## TACACS+ 核心优势

- 每条命令都可授权
- 可以拒绝某个命令
- 管理员访问控制最佳选择

------

#  四、AAA 工作流程图（结合两种协议）

```id="v6hndg"
用户登录
   ↓
AAA 框架启动
   ↓
使用 method list
   ↓
调用 RADIUS 或 TACACS+
   ↓
认证成功？
   ↓
授权？
   ↓
Accounting 记录
```

------

#  五、核心差异（考试高频）

## 1️⃣ 加密范围

| 协议    | 加密情况   |
| ------- | ---------- |
| RADIUS  | 仅密码     |
| TACACS+ | 整个数据包 |

------

## 2️⃣ 授权粒度

| 协议    | 授权能力 |
| ------- | -------- |
| RADIUS  | 登录级别 |
| TACACS+ | 命令级别 |

------

## 3️⃣ 连接可靠性

| 协议    | 特点            |
| ------- | --------------- |
| RADIUS  | UDP，不保证传输 |
| TACACS+ | TCP，可靠连接   |

------

#  六、什么时候用什么？

| 场景             | 推荐    |
| ---------------- | ------- |
| 管理员登录路由器 | TACACS+ |
| 企业 WiFi        | RADIUS  |
| VPN 远程接入     | RADIUS  |
| 命令级别控制     | TACACS+ |
| 高性能接入       | RADIUS  |

------

#  七、最重要考试记忆点

```text
RADIUS = UDP 1812/1813
TACACS+ = TCP 49
RADIUS = 网络接入
TACACS+ = 设备管理
RADIUS = 密码加密
TACACS+ = 全部加密
```

------

#  八、典型故障定位方向

### 能登录但不能执行命令？

→ TACACS+ 授权问题

------

### 无法接入 WiFi？

→ RADIUS 问题

------

### 认证成功但 privilege level 不对？

→ 授权阶段失败

------

#  九、终极总结

```text
AAA 是框架
RADIUS 是网络接入认证协议
TACACS+ 是设备管理认证协议
```

------

rogue RA:

要阻止 rogue RA → 用 PVLAN → Router 放 promiscuous → Host 放 isolated

它在 isolated port

它无法把 RA 发送给其他 isolated 主机

因为 isolated 之间不能互通

但合法路由器（promiscuous）仍然可以发送 RA 给所有主机。