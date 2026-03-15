在 OSPF 中，过滤 LSA 的手段非常丰富。你可以根据需求选择**“区域类型（整体过滤）”\**或者\**“针对特定 LSA（精确过滤）”**。

以下是针对 Type 1 到 Type 5 的详细过滤方案：

### 一、 整体过滤（改变区域类型）

这种方式最简单，通过改变区域属性直接“一刀切”。

- **过滤 Type 4/5（外部路由）：** 将区域配置为 **Stub Area**。
- **过滤 Type 3/4/5（区域间+外部路由）：** 将区域配置为 **Totally Stubby Area**（即题目中的 `no-summary`）。
- **过滤 Type 5 但允许重分布：** 将区域配置为 **NSSA**。

------

### 二、 针对不同 LSA 的精准过滤

#### 1. 过滤 Type 1 & Type 2 (Router/Network LSA)

- **不能直接过滤：** 在一个区域内部，所有路由器的 LSDB 必须完全同步。如果你在区域内强行过滤 Type 1 或 2，会导致 SPF 计算失败，邻居关系震荡。
- **变通方法：**
  - **接口汇总 (Interface Range)：** 虽然不能过滤，但可以通过修改接口的 `ip ospf cost` 或将接口设为 `passive-interface` 来控制流量路径或减少不必要的 LSA 扩散。

#### 2. 过滤 Type 3 (Summary LSA)

这是 ABR 上最常见的操作。

- **区域间汇总 + 过滤 (Filter-List)：**

  Bash

  ```
  # 在 ABR 上，阻止特定前缀进入/离开区域
  ip prefix-list STOP_NETWORK deny 192.168.10.0/24
  ip prefix-list STOP_NETWORK permit 0.0.0.0/0 le 32
  
  router ospf 1
   area 1 filter-list prefix STOP_NETWORK in  # 阻止路由进入区域 1
  ```

- **汇总时不通告：**

  Bash

  ```
  # 在 ABR 上汇总时直接控制不发布
  area 1 range 192.168.10.0 255.255.255.0 not-advertise
  ```

#### 3. 过滤 Type 4 (ASBR Summary LSA)

- **自动过滤：** 当你配置 **Stub** 或 **NSSA** 区域时，ABR 会自动停止产生针对该区域的 Type 4 LSA。
- **手动控制：** 通常不需要手动过滤 Type 4，因为它是配合 Type 5 使用的。如果没有 Type 5，Type 4 也没有存在的意义。

#### 4. 过滤 Type 5 (External LSA)

Type 5 是由 ASBR 产生的，通常在 ASBR 或 ABR 上控制。

- **在 ASBR 上重分布时过滤：**

  Bash

  ```
  # 使用 Route-Map 在重分布入口处拦截
  route-map REDIST_FILTER deny 10
   match ip address prefix-list BAD_ROUTE
  route-map REDIST_FILTER permit 20
  
  router ospf 1
   redistribute static subnets route-map REDIST_FILTER
  ```

- **在 ABR 上过滤（仅限 NSSA 转 Type 5 时）：**

  如果是在 NSSA 区域的 ABR 上做 7 转 5 转换，可以用 `summary-address` 配合 `not-advertise`。

- **全局数据库过滤 (Distribute-List)：**

  注意：`distribute-list in` 只能过滤路由表（RIB），不能过滤数据库（LSDB）。如果想过滤 LSDB 里的 Type 5，可以使用：

  Bash

  ```
  router ospf 1
   distribute-list prefix STOP_EXTERNAL out static  # 仅对本地重分布有效
  ```

------

### 三、 总结对比表

| **LSA 类型**     | **过滤位置** | **推荐方法**                                              |
| ---------------- | ------------ | --------------------------------------------------------- |
| **Type 1 & 2**   | 无法过滤     | 只能通过 `passive-interface` 停止接口参与 OSPF。          |
| **Type 3**       | **ABR**      | 使用 `area filter-list` 或 `area range not-advertise`。   |
| **Type 4 & 5**   | **ABR/ASBR** | 配置 **Stub/NSSA** 区域，或在重分布时使用 **Route-Map**。 |
| **整体 (3/4/5)** | **ABR**      | 使用 **Totally Stubby** (`no-summary`)。                  |

这两个关键字虽然都能起到“过滤”的效果，但它们作用的**范围**和**逻辑**完全不同。你可以把它们理解为：一个是“外科手术（精准切除）”，一个是“关掉水闸（整块切断）”。

------

### 1. `not-advertise`：精准的“外科手术”

这个关键字通常配合 `area range`（用于 Type 3）或 `summary-address`（用于 Type 5）使用。它的作用是：**汇总了这组网段，但我不告诉别人。**

- **作用对象：** 特定的 IP 网段。
- **配置位置：** * **ABR** 上针对 Type 3：`area <id> range <network> <mask> not-advertise`
  - **ASBR** 上针对 Type 5：`summary-address <network> <mask> not-advertise`
- **使用场景：**
  - 你想隐藏某个特定子网（比如财务服务器的 IP 段），不让它传到其他区域。
  - 防止特定路由导致的环路。

### 2. `no-summary`：粗暴的“关掉水闸”

这个关键字是配置 **Totally Stubby** 或 **Totally NSSA** 区域时的灵魂。它的作用是：**禁止所有的 Type 3 进入该区域。**

- **作用对象：** 所有的区域间路由（Summary LSAs）。
- **配置位置：** 仅在 **ABR** 上配置：`area <id> stub no-summary`。
- **使用场景：**
  - **精简路由表：** 比如你的分支机构（Area 1）只有一台低端路由器，它不需要知道总部（Area 0）和海外办公室（Area 2）里那几千条路由明细。
  - 你只想让它通过一条默认路由（0.0.0.0/0）走天下，省 CPU 和内存。

------

### 关键对比表

| **特性**     | **not-advertise**            | **no-summary**                       |
| ------------ | ---------------------------- | ------------------------------------ |
| **细粒度**   | **高**（针对具体的某个网段） | **低**（针对所有区域间路由）         |
| **过滤类型** | 过滤特定的 Type 3 或 Type 5  | 过滤**所有**明细 Type 3              |
| **配置前提** | 必须先做汇总（Range）        | 必须先配置 Stub/NSSA 区域            |
| **默认路由** | 不会自动产生默认路由         | **会自动产生**一条 Type 3 默认路由   |
| **类似比喻** | 就像在名单上划掉某人的名字   | 就像直接把这页纸撕掉，换成“详见大厅” |

------

### 举个例子

假设 Area 0 有三个网段：`10.1.1.0/24`, `10.1.2.0/24`, `10.1.3.0/24`。你想发给 Area 1。

- **情况 A：** 你只想让 Area 1 知道 `.1` 和 `.2`，隐藏 `.3`。
  - **配置：** `area 0 range 10.1.3.0 255.255.255.0 not-advertise`
- **情况 B：** 你觉得 Area 1 太弱了，不想给它任何明细，只要它能上网就行。
  - **配置：** `area 1 stub no-summary`
  - **结果：** Area 1 的路由表里看不到任何 `10.x.x.x`，只能看到一条 `O*IA 0.0.0.0/0`。

**注意一个细节：** 在你之前做的那个填空题里，题目要求 R3 **“only type 1 and 2”**。因为 `not-advertise` 是针对特定路由的，如果你用它来过滤，你得把全网成百上千条路由都写一遍，这不现实。所以这种情况必须选 `no-summary` 来搞定“整体过滤”。

