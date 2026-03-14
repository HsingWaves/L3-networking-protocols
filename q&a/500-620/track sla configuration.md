! 1. 定义探测器 (ID 必须统一)
ip sla 2
 icmp-echo 10.1.1.1
 frequency 5
!
! 2. 启动探测器 (这是原配置缺少的)
ip sla schedule 2 life forever start-time now
!
! 3. 定义 Track 对象，引用上面定义的 SLA 2
track 2 ip sla 2 reachability
 delay down 30 up 180
!
! 4. 静态路由绑定 Track
ip route 0.0.0.0 0.0.0.0 10.1.1.1 track 2





**IP SLA (Service Level Agreement)** 与 **Static Route Tracking** 的联动配置。

要使静态路由根据网络连通性动态撤回，必须满足三个关键步骤：

1. **定义 SLA 会话**：探测目标（如 ICMP echo）。
2. **调度 SLA**：必须使用 `schedule` 命令让探测任务开始运行。
3. **关联 Track 对象与静态路由**：在静态路由末尾添加 `track` 关键字。

------

### 题目分析

观察原始配置中的错误：

- **缺少调度**：配置了 `ip sla 2`，但没有 `ip sla schedule 2 ...`，导致探测从未开始。
- **Track 对象编号不匹配**：`track 2` 尝试关联 `ip sla 200`，但定义的 SLA 编号是 `2`。
- **路由未关联 Track**：`ip route 0.0.0.0 0.0.0.0 10.1.1.1` 后面没有跟 `track 2`，所以路由表不会根据探测结果更新。