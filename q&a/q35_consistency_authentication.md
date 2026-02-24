**NY 的 E0/0 已经开了 OSPF 的 MD5（message-digest）认证**：

- `ip ospf authentication message-digest`
- `ip ospf message-digest-key 1 md5 Cisco123`

而 **LA 侧完全没配置认证** → 两端认证类型不一致，所以邻居起不来。

题目给的选项里，没有直接给你 “LA interface e0/0 ip ospf authentication message-digest” 这种接口级开启命令，所以要用 **area 级别**把 LA 的 area 0 统一开启 MD5，再在 LA 的接口上补 key。

✅ 正确两项是：

**C.** `LA interface E0/0 ip ospf message-digest-key 1 md5 Cisco123`
 **D.** `LA router ospf 1 area 0 authentication message-digest`





