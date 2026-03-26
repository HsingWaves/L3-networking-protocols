| **SNMP 版本** | **对应特性 (Features)**   | **说明**                                               |
| ------------- | ------------------------- | ------------------------------------------------------ |
| **SNMPv2c**   | **community string**      | 使用明文团体字（如 public/private）作为唯一验证。      |
|               | **no encryption**         | 报文在网络中明文传输，极不安全。                       |
|               | **read-only**             | 典型的访问权限控制方式（也可以有 read-write）。        |
| **SNMPv3**    | **username and password** | 引入了 USM，基于用户名和密码进行管理。                 |
|               | **authentication**        | 提供 HMAC-MD5 或 HMAC-SHA 验证数据完整性。             |
|               | **privileged**            | 对应 **Priv** (Privacy)，即对数据进行加密（DES/AES）。 |

### 2. 深度解析：SNMPv3 的安全级别

SNMPv3 不仅仅是“有安全”，它还分了三个档次，这也是考试的高频考点：

- **noAuthNoPriv**: 无验证，无加密（类似 v2c，但用用户名）。
- **authNoPriv**: **有验证**（MD5/SHA），**无加密**。
- **authPriv**: **既有验证又有加密**（这是最安全的级别，对应选项中的 **privileged** 和 **authentication**）。

------

### 3. 记忆技巧

- **v2c = "Clear" (明文)**: 想到 **c**ommunity string 和 **c**lear text (no encryption)。
- **v3 = "VIP Service"**: 想象一个高级俱乐部，你需要 **Username**，需要身份 **Authentication**，还需要进入 **Privileged** (私密/加密) 包厢。

------

### 考场避坑

题目中有个 **privileged** 选项，在 SNMP 的语境下，它其实是 **Privacy (Priv)** 的变体，指代的是**加密（Encryption）**。很多同学会把它和交换机的特权模式混淆，但在 SNMPv3 里，看到它就要立刻想到 **AES/DES 加密**。



```
# 1. 定义一个视图 (可选，决定用户能看到哪些 MIB 节点)
snmp-server view MYVIEW iso included

# 2. 创建一个组 (Group)
# 必须指定安全级别为 priv，并关联视图
snmp-server group MYGROUP v3 priv read MYVIEW

# 3. 创建用户 (User) 
# 指定验证算法 (md5/sha) 和加密算法 (aes/des)
# 格式：snmp-server user [用户名] [组名] v3 auth [验证协议] [验证密码] priv [加密协议] [加密密码]
snmp-server user ADMINUSER MYGROUP v3 auth sha Cisco123 priv aes 128 Cisco456

# 4. 配置 Trap 主机 (让路由器主动告警)
snmp-server host 192.168.1.100 version 3 priv ADMINUSER
```



| **参数**     | **含义** | **考试/实战注意点**                                          |
| ------------ | -------- | ------------------------------------------------------------ |
| **v3 priv**  | 安全级别 | 对应题目中的 **privileged** 和 **authentication**，既验证身份又加密内容。 |
| **auth sha** | 验证协议 | SHA 比 MD5 更安全，通常是推荐做法。                          |
| **priv aes** | 加密协议 | AES (128/192/256) 是现代标准，远强于过时的 DES。             |
| **EngineID** | 唯一标识 | SNMPv3 内部使用 EngineID 来生成加密哈希，通常自动生成。      |

| **安全级别**     | **验证 (Authentication)** | **加密 (Privacy/Encryption)** | **对应题目选项**                    |
| ---------------- | ------------------------- | ----------------------------- | ----------------------------------- |
| **noAuthNoPriv** | **无** (仅靠 Username)    | **无** (明文传输)             | 类似 v2c，但不使用 community string |
| **authNoPriv**   | **有** (MD5 或 SHA)       | **无** (明文传输)             | 对应选项中的 **authentication**     |
| **authPriv**     | **有** (MD5 或 SHA)       | **有** (DES 或 AES)           | 对应选项中的 **privileged**         |