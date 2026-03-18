In the exhibit, the user has created the "ingredients" for a tracked route, but they haven't "linked" them together yet:

1. **Primary Route:** `ip route 0.0.0.0 0.0.0.0 1.1.1.1` (Administrative Distance of **1**).
2. **Backup Route:** `ip route 0.0.0.0 0.0.0.0 2.2.2.2 10` (Administrative Distance of **10**).
3. **The Probe:** An IP SLA is pinging `1.1.1.1`.
4. **The Monitor:** `track 1` is watching that SLA.

**The issue:** The primary route (via 1.1.1.1) is currently static and unconditional. Even if the SLA fails and the track goes "Down," the routing table will keep the primary route because it doesn't know it’s supposed to care about the track object.

### The Fix

By applying **Option D**: `R1(config)# ip route 0.0.0.0 0.0.0.0 1.1.1.1 track 1`

## You are telling the router: *"Only keep this primary route in the routing table as long as Track 1 is UP."*

### 1. 它是如何“绑定”的？

`track 1` 实际上起到了**桥梁**的作用，它连接了两个独立的部分：

- **底层（IP SLA）：** 你配置的 `ip sla 1` 负责不停地给 `1.1.1.1` 发 Ping 包。它只管看“通不通”。
- **上层（静态路由）：** 你的 `ip route ... track 1` 负责转发流量。它只管看“Track 1 是 Up 还是 Down”。

------

### 2. 为什么要加这一层“绑定”？

如果不加 `track`，只写普通的静态路由，路由器是很“死板”的：

- **普通路由：** 只要路由器的本地接口（比如 FastEthernet0/0）是 **Up** 的，它就认为 `1.1.1.1` 是可达的。
- **带 Track 的路由：** 即使接口是 Up 的，如果运营商内部线路断了（导致 Ping 不通 `1.1.1.1`），`track 1` 就会变成 **Down** 状态。一旦它变 Down，路由器会**立刻**把这条主路由从路由表中删掉。

### 3. 删掉之后会发生什么？

这就是你题目中提到的 **“浮动静态路由”**（Floating Static Route）发挥作用的时候了：

1. 主路由（AD值为 1）因为 `track 1` 挂了而被删除。
2. 路由器发现还有一条备用路由（AD值为 10）。
3. **备用路由自动上位**，流量改走 `2.2.2.2`。

------

### 总结

所以，`track 1` 确实是和 IP（通过 SLA 探测）绑定了，目的是为了让静态路由具备**“感知网络质量”**的能力，而不仅仅是看接口灯亮不亮。

> **小贴士：** 在实际工作中，我们不仅可以 track 一个 IP（Ping），还可以 track 一个接口的状态、甚至是一个 IPv6 的可达性。

你想看看当 `track 1` 真的断开时，用什么命令能查看到它变红（Down）的状态吗？

`track 1` 是一个**“中间监视器”**，它把**底层的探测结果**和**上层的路由决策**给“焊”在了一起