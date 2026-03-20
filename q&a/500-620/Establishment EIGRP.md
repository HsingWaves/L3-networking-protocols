EIGRP 邻居必须满足以下条件：

1. **AS 号必须一致**（本题中应统一为 100）。
2. **接口必须被 network 命令涵盖**。
3. **K 值 (K-values) 必须匹配**。
4. **认证 (Authentication) 必须匹配**（如果有）。
5. **主 IP 地址必须在同一子网**。



```Router C
``
Router C
router eigrp CCNP
 address-family ipv4 unicast autonomous-system 100
  topology base
  exit-af-topology
  network 198.18.133.0
 exit-address-family
```



```
Router A
router eigrp CCNP
 address-family ipv4 unicast autonomous-system 100
  topology base
  exit-af-topology
  network 198.18.133.0
 exit-address-family
```

