

------

## 核心概念

在 **OSPF** 与 **MPLS VPN** 结合使用时，如果 **CE 路由器之间还有一条直接连接（backdoor link）**，就会出现 **OSPF backdoor routing problem**。

### 为什么会出现问题

OSPF 有一个规则：

> **Intra-area routes 优先于 Inter-area routes**

典型拓扑：

```
CE1 ---- PE1 ===== MPLS ===== PE2 ---- CE2
  \___________________________________/
            backdoor link
```

- **通过 MPLS VPN**
  - OSPF route type → **Inter-area (O IA)**
- **通过 backdoor link**
  - OSPF route type → **Intra-area (O)**

因为 **Intra-area 优先级更高**，所以流量会走 **backdoor link**，而不是 MPLS VPN。

------

## Sham-link 的作用

**OSPF Sham-link** 的作用：

👉 **把 PE1 和 PE2 之间“虚拟成一条 Intra-area link”**

这样：

- MPLS VPN 路径也变成 **Intra-area route**
- OSPF 就可以按 **cost 选择路径**
- 从而避免 **backdoor routing problem**

------

## 记忆口诀（ENARSI 秒杀）

看到题目关键词：

| 关键词                        | 选什么         |
| ----------------------------- | -------------- |
| MPLS VPN + OSPF               | 想到 sham-link |
| backdoor link                 | sham-link      |
| intra-area preference problem | sham-link      |

**一句话记忆：**

> **Sham-link = 修复 OSPF backdoor routing**

------

