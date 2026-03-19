1. 财务部门的服务器位于 `200.30.40.0/24` 网络中。
2. 服务器的可达性不稳定（not reachable consistently）。
3. 服务器每两个月会获取一个新的 IP 地址。

**核心问题：** 对于服务器（Server）这类基础设施，**绝不应该使用动态 DHCP 获取地址**。如果 IP 地址随租约（Lease）过期而改变，客户端和应用将无法通过固定地址找到服务器，导致连接中断。此外，如果服务器占用的 IP 没有在 DHCP 地址池中排除，可能会发生 **IP 地址冲突**。

### 

#### **B. Configure the server with a static IP address and default gateway.**

- **解析：** 为了保证服务器的稳定性，必须为其配置**静态 IP（Static IP）**。
- **作用：** 这样服务器的 IP 永远不会改变，确保了服务的持续可达性。同时，配置默认网关确保服务器可以跨网段通信。

#### **E. Configure the router to exclude a server IP address and default gateway.**

- **解析：** 在路由器（DHCP Server）端，必须将分配给服务器的静态 IP 从 DHCP 地址池中**排除（Exclude）**。
- **作用：** 使用 `ip dhcp excluded-address` 命令。如果不排除，DHCP 服务器可能会把这个 IP 分配给其他新接入的设备（如员工的电脑），导致 **IP 冲突**，进而造成服务器网络断断续续。