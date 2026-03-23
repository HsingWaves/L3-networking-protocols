| **报错信息**                                | **可能的原因**                                   | **解决方法**                                      |
| ------------------------------------------- | ------------------------------------------------ | ------------------------------------------------- |
| **`%Error opening... (Timeout)`**           | 路径不通、中间有防火墙拦截、UDP 69 没开。        | 检查路由、ACL 或开启端口（Option A 对应的情况）。 |
| **`%Error opening... (Permission denied)`** | 服务器软件没启动、读写权限受限、文件未预先创建。 | **启动服务器软件（Option D）** 或修改权限。       |
| **`%Error opening... (No such file)`**      | 下载 IOS 时，服务器上找不到对应的文件名。        | 检查文件名拼写。                                  |

1. Why is D the most accurate answer?

Key error: Permission denied

This error means the request reached the server but was rejected.

Eliminating A (Open UDP port 69): If UDP port 69 is blocked by a firewall, or the port is not open causing packets to be dropped, the switch will usually display a Timeout. Since the packet never even reached the server, it cannot be considered "rejected."

D's logic: When the TFTP server software is in a "stopped" state, or although it is running but without "write permission" configured, the server operating system or service will return a rejection response.

Exam logic: If ping is successful, it means the network layer (L3) is fine. If the error is not "timeout" but a specific "rejection," it usually means the problem lies in the application layer, i.e., the state or configuration of the server software itself.

2. Error Message Comparison Table (Troubleshooting Tips)

To help you prepare for the exam, I have summarized several common TFTP error states, which are often used as distractors in the exam:



### The Problem: Subnet Mismatch

If you look closely at the IP addresses and subnet masks in the diagram:

- **TFTP Server:** `10.0.0.2 /24` (Subnet: `10.0.0.0` to `10.0.0.255`)
- **Switch (E0/1):** `10.0.1.1 /24` (Subnet: `10.0.1.0` to `10.0.1.255`)

The TFTP server and the switch are on **different subnets**. In the CLI output shown in the image, the engineer attempts to send the configuration to `10.0.0.1`. However, there is no device at `10.0.0.1` visible, and more importantly, the switch's own interface is in the `10.0.1.x` range.

### Why C is the solution

For the `copy` command to work in this specific peer-to-peer topology without a router in between:

1. The devices must be in the **same broadcast domain/subnet**.
2. The engineer is currently trying to reach an IP (`10.0.0.1`) that doesn't match the switch's local interface configuration.
3. By using **10.0.1.1** (or ensuring both devices are on the `10.0.1.x` or `10.0.0.x` network), the connectivity issue is resolved.

> **Note:** In many certification exam questions like this, "Use TFTP server IP address 10.0.1.1" implies changing the server's IP to match the switch's subnet so they can communicate directly.