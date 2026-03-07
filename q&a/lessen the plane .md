

------

# 

题目：

> The control plane is heavily impacted after the CoPP configuration is applied to the router.
> Which command removal lessens the impact on the control plane?

翻译：

```text
在路由器上应用 CoPP 配置后，
控制平面（control plane）受到严重影响。

删除哪条命令可以减少对控制平面的影响？
```

也就是：

```text
配置了 CoPP 后 CPU 很高
要删哪条 ACL 规则能缓解
```

------

# 二、什么是 Control Plane

路由器有三个 plane：

| plane            | 作用                  |
| ---------------- | --------------------- |
| Data Plane       | 转发数据包            |
| Control Plane    | 路由协议 / BGP / OSPF |
| Management Plane | SSH / SNMP            |

**Control Plane 的流量全部进 CPU**。

例如：

- OSPF hello
- BGP keepalive
- EIGRP update
- PIM

这些都是：

```text
CPU 处理
```

------

# 三、什么是 CoPP

**CoPP = Control Plane Policing**

作用：

```text
保护路由器 CPU
```

防止：

- DoS
- routing protocol flood
- control-plane overload

通常结构：

```text
ACL → class-map → policy-map → control-plane
```

------

# 四、题目里的 ACL 在干嘛

这段：

```text
ACL for CoPP Routing class-map
```

说明：

```text
ACL 是用来匹配控制平面流量
```

例如：

```text
permit tcp ... bgp
permit ospf
permit eigrp
```

这些都是：

```text
路由协议控制流量
```

------

# 五、问题在哪

第一条：

```cisco
access-list 120 permit tcp any gt 1024 eq bgp log
```

重点：

```text
log
```

------

# 六、为什么 log 会杀 CPU

ACL logging 工作方式：

每次匹配都会：

```text
生成 syslog
写入 buffer
CPU 处理
```

而 BGP 是：

```text
大量 keepalive / update
```

所以结果是：

```text
BGP packet
   ↓
ACL match
   ↓
log
   ↓
CPU
```

如果 BGP session 多：

```text
CPU = 爆炸
```

------

# 七、题目要你干什么

题目问：

```text
删哪条命令能减少 control-plane impact
```

就是：

```text
删掉最耗 CPU 的规则
```

显然就是：

```cisco
access-list 120 permit tcp any gt 1024 eq bgp log
```

------

# 八、为什么其他选项不是

### B

```cisco
permit ospf any
```

OSPF 本来就是 control-plane traffic。

删除它反而可能：

```text
OSPF 被错误限速
邻居断
```

------

### C

```cisco
permit udp any any eq pim-auto-rp
```

只是 multicast 控制协议。

流量很少。

------

### D

```cisco
permit eigrp any host 224.0.0.10
```

EIGRP multicast。

正常配置。

------

# 九、一句话总结

题目其实在问：

```text
为什么 CoPP ACL 会让 CPU 爆？
```

答案：

```text
ACL logging
```

------

# 十、考试秒杀规律

看到：

```text
CoPP
CPU high
ACL
log
```

直接想到：

```text
remove log
```

------

