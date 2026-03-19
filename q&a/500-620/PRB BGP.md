| **场景**                         | **结果**                                                     |
| -------------------------------- | ------------------------------------------------------------ |
| **PBR Route-map Match + Permit** | 执行 `set` 动作（策略路由生效）                              |
| **PBR Route-map Match + Deny**   | **不丢弃**，转为普通路由查找 (Normal Routing)                |
| **PBR Route-map No Match**       | 继续评估下一个 `sequence`；若全不匹配，转普通路由查找        |
| **ACL 中的 Deny**                | 如果 `match ip address` 指向的 ACL 里有 `deny`，表示“不匹配这一行 route-map”，继续看下一个 `sequence` |



BGP ajacent

| **需求**                  | **必配命令**                                               |
| ------------------------- | ---------------------------------------------------------- |
| **使用非直链接口建邻居**  | `update-source [Interface]`                                |
| **eBGP 邻居不是物理直连** | `ebgp-multihop [hops]` **或** `disable-connected-check`    |
| **iBGP 邻居建立**         | 只需 `update-source`，不需要 multihop（iBGP 默认 TTL 255） |

R1 和 R2 试图使用 **Loopback 地址** 建立 eBGP 邻居，但目前存在三个障碍：

1. **路由不可达：** R1 的路由表中没有去往 `10.1.1.2` 的路由，R2 同理。BGP 基于 TCP，没有路由就无法完成三次握手。
2. **源 IP 不匹配：** 默认情况下，BGP 使用出接口 IP 作为源地址。R1 发出的包源 IP 是 `192.168.1.1`，但 R2 期待的邻居是 `10.1.1.1`。
3. **eBGP 直连检测限制：** 默认情况下，eBGP 邻居必须是物理直连的（TTL=1）。由于 Loopback 地址在逻辑上被认为是“跳了一跳”，BGP 会检查邻居是否在同一个子网。如果不在，邻居无法建立。

**方法 A：`disable-connected-check`**

- 这个命令告诉路由器：“我知道邻居不是物理直连的，不用检查子网，直接尝试建立连接。”
- **对应选项：** 图中最后一张截屏的两个勾选选项。

**方法 B：`ebgp-multihop` 或 `ttl-security`**

- `ebgp-multihop` 将 TTL 增加（默认增加到 255）。
- `ttl-security hops 1` 也是一种安全加固方式，虽然它主要用于防攻击，但在某些版本中配置它也能让 Loopback 邻居起来。
- **但注意：** 选项 A 和 B 中对 R1/R2 的配置有逻辑错误（例如 R1 的配置里写了 `neighbor 10.1.1.1` 指向自己，这显然是错的）。

**看图：** 如果中间**没有**云或其他路由器，选 `disable-connected-check`。

**看图：** 如果中间**有**云或写着 WAN，必须选 `ebgp-multihop`。

**看命令：** 只要用了 `ttl-security`，它会自动处理直连检查逻辑，但这通常是针对安全加固的题。