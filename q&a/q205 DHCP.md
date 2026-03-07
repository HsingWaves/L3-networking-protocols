这是一个很典型的 **DHCP relay + Option 82 trust** 题：

```
Client -> R1(relay) -> R2(server)
```

故障点：

- R1 要能做 DHCP relay → `service dhcp`
- R2 要接受 relay info / Option 82 → `ip dhcp relay information trust-all`