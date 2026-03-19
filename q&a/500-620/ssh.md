正确答案是 **B (Configure a domain name)** 和 **C (Configure a crypto key to be generated)**。

在 Cisco IOS 设备上启用 SSH（无论是作为服务器还是客户端发起连接），必须具备以下三个基础要素：

1. **Hostname（主机名）**：路由器必须有一个非默认的主机名。
2. **Domain Name（域名）**：通过 `ip domain-name [域名]` 命令配置。
3. **Crypto Key（加密密钥对）**：通过 `crypto key generate rsa` 生成。

------

### 为什么选择这两项？

- **B. Configure a domain name (配置域名):** SSH 密钥的生成过程依赖于“完全限定域名”(FQDN)。在 Cisco 设备中，FQDN 是由 **Hostname + Domain Name** 组合而成的。如果没有域名，系统将无法生成用于加密 SSH 会话的 RSA 密钥。
- **C. Configure a crypto key to be generated (生成加密密钥):** 这是最关键的一步。SSH（Secure Shell）之所以安全，是因为它使用非对称加密来保护数据。如果没有生成的 RSA 密钥，路由器就无法进行身份验证和数据加密，连接自然会失败。

####  **Remove class-map ANY from service-policy CoPP**

- **解析：** 在 `show policy-map control-plane` 中可以看到 `Class-map: ANY` 下面有一个 `drop` 动作。
- **作用：** 这个类匹配的是 ACL 199。如果 ACL 199 拒绝了某些流量（deny），在 CoPP 逻辑中，这些流量反而会跳过这个 drop 动作，进入 `class-default`。这种配置极易造成逻辑混乱和误删。移除这个自定义的 `ANY` 类可以简化控制平面的流量处理逻辑，让流量按照标准的 `class-default` 进行处理。

####  Configure transport input ssh on line vty and remove sequence 30 from access list 100

- **解析：** 这是实现“仅限 SSH”的核心步骤。
  1. **`transport input ssh`**: 在 `line vty` 下配置此命令后，路由器将直接**拒绝所有非 SSH 的协议**（如 Telnet、rlogin）。这是最有效、最简单的限制方式。
  2. **`remove sequence 30 from access list 100`**: ACL 100 的序列 30 是 `permit tcp any eq telnet any`。既然我们要限制为 SSH，就必须清理掉 ACL 中允许 Telnet 的条目，防止控制平面继续处理这些不安全的流量。