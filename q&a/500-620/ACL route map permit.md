

------

### 1. 用于路由重分布（Redistribution / Filtering）

在这种场景下，`route-map` 就像是一个“安检口”。

- **`permit`**：如果匹配成功，该路由**允许**通过，会被注入到目标协议或更新中。
- **`deny`**：如果匹配成功，该路由**被阻断/丢弃**，不会被重分布。

**逻辑关系表：**

| **Route-map 动作** | **ACL/Prefix-list 结果** | **最终结果**                |
| ------------------ | ------------------------ | --------------------------- |
| **Permit**         | Permit                   | **放行路由**                |
| **Permit**         | Deny                     | 不匹配，寻找下一个 sequence |
| **Deny**           | Permit                   | **丢弃路由**                |
| **Deny**           | Deny                     | 不匹配，寻找下一个 sequence |

> **注意**：如果所有 sequence 都不匹配，末尾有一个隐藏的 **Deny All**，路由会被丢弃。

------

### 2. 用于策略路由（PBR - Policy Based Routing）

在这种场景下，`route-map` 更像是一个“分流器”。**它不决定路由的死活，只决定是否执行 `set` 语句。**

- **`permit`**：
  - 如果匹配成功（ACL 为 Permit），执行 `set` 语句（比如修改下一跳或优先级）。
  - 如果匹配失败（ACL 为 Deny），**跳过**当前 sequence，去看下一个。
- **`deny`**：
  - 即便匹配成功，也**不执行** `set` 语句。
  - 该流量会直接**退出策略路由**，交给传统的“路由表（RIB）”进行正常转发。

**逻辑关系表：**

| **Route-map 动作** | **ACL 匹配结果** | **最终结果**                      |
| ------------------ | ---------------- | --------------------------------- |
| **Permit**         | Permit           | **执行 `set` 动作（如改下一跳）** |
| **Permit**         | Deny             | 跳过，看下一个 sequence           |
| **Deny**           | Permit           | **不执行 `set`，按普通路由转发**  |

------

### 3. ACL 的 Permit/Deny 又是什么角色？

无论在哪种场景，`route-map` 里的 `match` 语句调用 ACL 时：

- **ACL Permit** = “选中”了这波流量/路由。
- **ACL Deny** = “没选中”这波流量/路由。

**举个极端的例子（PBR 场景）：**

Bash

```
access-list 1 permit 10.1.1.0 0.0.0.255

route-map MY_PBR deny 10
  match ip address 1
  set ip next-hop 1.1.1.1
```

- **结果**：10.1.1.0 的流量匹配了 ACL 1（Permit），进入了 `deny 10`。因为是 `deny`，所以 `set` 语句**失效**，流量按普通路由转发。

------

### 💡 核心口诀

1. **分发路由看生死**：`permit` 是活，`deny` 是死。
2. **策略路由看特权**：`permit` 是改（执行 `set`），`deny` 是不改（交给路由表）。
3. **末尾隐藏大 Boss**：
   - 路由重分布：末尾隐藏 `deny all`（全丢弃）。
   - 策略路由 PBR：末尾隐藏 `permit all` 但无 `set`（全走普通路由）。

