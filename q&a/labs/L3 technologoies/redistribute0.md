

------

## Task 1: 配置 OSPF (P1R1 & P1R2)

这个阶段的目标是让左侧的两个路由器说话。

- **Problem 1: 在 P1R1 上配置 OSPF**

  进入 P1R1，将它的 Loopback0 和 Serial0/0 宣告进 Area 0。

  Bash

  ```
  P1R1(config)# router ospf 1
  P1R1(config-router)# network 192.168.1.0 0.0.0.15 area 0
  P1R1(config-router)# network 192.168.1.32 0.0.0.3 area 0
  ```

- **Problem 2: 在 P1R2 上配置 OSPF**

  同样，在 P1R2 上宣告它左侧的接口。

  Bash

  ```
  P1R2(config)# router ospf 1
  P1R2(config-router)# network 192.168.1.16 0.0.0.15 area 0
  P1R2(config-router)# network 192.168.1.32 0.0.0.3 area 0
  ```

- **Problem 3: 检查 OSPF 邻居状态**

  在 P1R1 上输入 `show ip ospf neighbor`。

  **检查点：** State 是否显示为 `FULL`？如果是，说明它们已经交换了拓扑信息。

- **Problem 4: 检查 P1R1 的路由表**

  输入 `show ip route`。

  **检查点：** 你是否看到一条以 `O` 开头的路由指向 `192.168.1.17`（P1R2的Loopback）？

------

## Task 2: 配置 EIGRP (P1R2 & P1R3)

现在我们要处理右侧的“邻里关系”。

- **Problem 5: 在 P1R2 上启用 EIGRP 100**

  宣告连接 Switch1 的网段。

  Bash

  ```
  P1R2(config)# router eigrp 100
  P1R2(config-router)# network 192.168.1.48 0.0.0.15
  ```

- **Problem 6: 在 P1R3 上启用 EIGRP 100**

  宣告它的 Loopback 和连接 Switch1 的网段。

  Bash

  ```
  P1R3(config)# router eigrp 100
  P1R3(config-router)# network 192.168.1.48 0.0.0.15
  P1R3(config-router)# network 192.168.1.64 0.0.0.15
  ```

- **Problem 7: 在 P1R2 上配置被动接口 (Passive Interface)**

  为了防止 EIGRP 报文发往左侧的 OSPF 区域（浪费带宽且不安全）：

  Bash

  ```
  P1R2(config-router)# passive-interface serial 0/0
  ```

- **Problem 8: 检查 EIGRP 邻居**

  在 P1R2 上输入 `show ip eigrp neighbors`。

  **检查点：** 应该能看到 P1R3 的地址（192.168.1.50）。

- **Problem 9: 检查 P1R2 的 EIGRP 路由**

  输入 `show ip route`。

  **检查点：** 是否看到一条以 `D` 开头的路由指向 `192.168.1.64`？

------

## Task 3: 配置路由重分布 (核心步骤)

此时 P1R1 依然无法 ping 通 P1R3，因为 P1R2 还没有把两边的信息进行“翻译”。

- **Problem 10: 测试连通性**

  在 P1R1 上 `ping 192.168.1.65`。

  **结果：** 必然失败（Success rate is 0 percent），因为 P1R1 的路由表里还没有右侧的网段。

- **Problem 11: 在 P1R2 上执行双向重分布**

  这是全实验最重要的命令。

  Bash

  ```
  # 把 OSPF 塞进 EIGRP (必须给 Metric)
  P1R2(config)# router eigrp 100
  P1R2(config-router)# redistribute ospf 1 metric 10000 100 255 1 1500
  
  # 把 EIGRP 和 直连网段 塞进 OSPF
  P1R2(config)# router ospf 1
  P1R2(config-router)# redistribute eigrp 100 subnets
  P1R2(config-router)# redistribute connected subnets
  ```

- **Problem 12: 最终检查路由表**

  分别在 P1R1 和 P1R3 上查看 `show ip route`。

  - **P1R1：** 会出现 `O E2` 路由（表示这是从外部重分布进来的）。
  - **P1R3：** 会出现 `D EX` 路由（表示外部 EIGRP 路由）。

- **Problem 13: 最终 Ping 测试**

  再次从 P1R1 `ping 192.168.1.65`。

  **结果：** 此时应该能够看到 `!!!!!`，实验大功告成！

------

