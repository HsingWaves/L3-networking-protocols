在规划 **MPLS Layer 3 VPN** 时，尤其是针对**全网状（Full-Mesh）**拓扑，Route Target (RT) 的配置逻辑如下：

- **全网状（Full-Mesh）逻辑：** 在这种模型下，VPN 中的每一个站点（Site）都需要能够接收来自其他所有站点的路由，并且也要能将自己的路由发布给所有人。
- **RT 的作用：** RT 是一种 BGP 扩展团体属性（Extended Community），用于控制路由的导入（Import）和导出（Export）。
  - **Export RT：** 决定这条路由带有什么“标签”发出去。
  - **Import RT：** 决定 PE 路由器将接受哪些带有特定“标签”的路由进入 VRF。

### 总结

对于 **Full-Mesh VPN**：

- **RD：** 通常每个 PE/VRF 配置唯一的 RD（推荐做法，以便调试和路径选择）。
- **RT：** 整个 VPN 范围内使用**相同（Identical）**的 RT。