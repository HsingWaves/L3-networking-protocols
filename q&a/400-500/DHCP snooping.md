 DHCP Snooping 的 **“全家桶”行为**：

1. **所有非信任接口（Untrusted）**的 DHCP Server 报文会被丢弃。
2. 默认会插入 **Option 82**。
3. 如果插入了 Option 82 但 `giaddr` 为零，路由器/服务器可能会不高兴。

**解决 Option 82 冲突的三个办法：**

1. **交换机端（如本题）：** `no ip dhcp snooping information option`（简单粗暴，直接关掉）。
2. **路由器/中继端：** `ip dhcp relay information trust-all`（告诉中继器：哪怕 giaddr 是 0，也请相信这个带 Option 82 的报文）。
3. **服务器端：** 配置服务器允许处理 giaddr 为 0 的 Option 82 报文。