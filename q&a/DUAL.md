```
#include <iostream>
#include <vector>
#include <string>
#include <limits>
#include <queue>
#include <unordered_map>
#include <unordered_set>
#include <algorithm>

using namespace std;

static constexpr int INF = numeric_limits<int>::max() / 4;

struct Edge {
    int to;
    int cost;
    bool up = true;
};

struct Graph {
    vector<string> name;
    vector<vector<Edge>> adj;

    int addNode(const string& n) {
        name.push_back(n);
        adj.push_back({});
        return (int)name.size() - 1;
    }

    void addUndirected(int a, int b, int cost) {
        adj[a].push_back({b, cost, true});
        adj[b].push_back({a, cost, true});
    }

    void setLink(int a, int b, bool up) {
        for (auto &e : adj[a]) if (e.to == b) e.up = up;
        for (auto &e : adj[b]) if (e.to == a) e.up = up;
    }
};

// Dijkstra from src to get shortest distances and next-hop candidates
static vector<int> dijkstra(const Graph& g, int src) {
    vector<int> dist(g.adj.size(), INF);
    dist[src] = 0;
    using P = pair<int,int>;
    priority_queue<P, vector<P>, greater<P>> pq;
    pq.push({0, src});

    while (!pq.empty()) {
        auto [d,u] = pq.top(); pq.pop();
        if (d != dist[u]) continue;
        for (const auto& e : g.adj[u]) {
            if (!e.up) continue;
            int v = e.to;
            int nd = d + e.cost;
            if (nd < dist[v]) {
                dist[v] = nd;
                pq.push({nd, v});
            }
        }
    }
    return dist;
}

struct CandidatePath {
    int via;   // neighbor
    int FD;    // feasible distance (total from local to dest via this neighbor)
    int RD;    // reported distance (neighbor's distance to dest)
};

static vector<CandidatePath> computeCandidatesDUALStyle(
    const Graph& g,
    int local,               // Branch
    int dest                 // R2 (target network sits here)
) {
    // Precompute shortest distance from every node to dest
    vector<int> distToDestFrom = dijkstra(g, dest);
    // distToDestFrom[x] = shortest from dest -> x, but undirected so same as x->dest

    vector<CandidatePath> cands;
    for (const auto& e : g.adj[local]) {
        if (!e.up) continue;
        int nbr = e.to;

        int RD = distToDestFrom[nbr]; // neighbor's distance to destination
        if (RD >= INF) continue;      // neighbor doesn't know a path

        long long FDll = (long long)e.cost + (long long)RD; // total via that neighbor
        if (FDll >= INF) continue;

        cands.push_back({nbr, (int)FDll, RD});
    }
    sort(cands.begin(), cands.end(), [](const auto& a, const auto& b){
        return a.FD < b.FD;
    });
    return cands;
}

static void printCandidates(const Graph& g, int local, int dest) {
    auto cands = computeCandidatesDUALStyle(g, local, dest);

    cout << "\n=== DUAL-like view at " << g.name[local] << " for destination@" << g.name[dest] << " ===\n";
    if (cands.empty()) {
        cout << "No candidates: destination unreachable from neighbors.\n";
        return;
    }

    // Successor is the lowest FD
    auto succ = cands.front();
    int succFD = succ.FD;

    cout << "Candidates (sorted by FD):\n";
    for (auto &c : cands) {
        cout << "  via " << g.name[c.via]
             << " | FD(total)=" << c.FD
             << " | RD(neighbor-to-dest)=" << c.RD
             << "\n";
    }

    cout << "\nSuccessor = via " << g.name[succ.via] << " (FD=" << succ.FD << ")\n";

    // Feasible Successor(s): RD < succFD (feasibility condition)
    vector<CandidatePath> fs;
    for (size_t i = 1; i < cands.size(); i++) {
        if (cands[i].RD < succFD) fs.push_back(cands[i]);
    }

    if (fs.empty()) {
        cout << "Feasible Successor = NONE (no neighbor satisfies RD < FD(successor))\n";
    } else {
        cout << "Feasible Successor(s):\n";
        for (auto &c : fs) {
            cout << "  via " << g.name[c.via] << " (FD=" << c.FD << ", RD=" << c.RD << ")\n";
        }
    }
}

static void simulateDiffusingQuery(const Graph& g, int local, int dest) {
    // This is a simplified "diffusion": local asks neighbors for a route;
    // neighbors reply with their RD, local picks best.
    cout << "\n--- Diffusing query (simplified) ---\n";
    cout << g.name[local] << " enters ACTIVE for destination@" << g.name[dest] << "\n";
    cout << g.name[local] << " sends QUERY to all neighbors...\n";

    vector<int> distToDestFrom = dijkstra(g, dest);
    vector<CandidatePath> replies;

    for (const auto& e : g.adj[local]) {
        if (!e.up) continue;
        int nbr = e.to;
        cout << "  QUERY -> " << g.name[nbr] << "\n";

        int RD = distToDestFrom[nbr];
        if (RD >= INF) {
            cout << "  REPLY <- " << g.name[nbr] << " : unreachable\n";
            continue;
        }
        cout << "  REPLY <- " << g.name[nbr] << " : RD=" << RD << "\n";
        long long FDll = (long long)e.cost + (long long)RD;
        if (FDll >= INF) continue;
        replies.push_back({nbr, (int)FDll, RD});
    }

    if (replies.empty()) {
        cout << g.name[local] << " receives no usable replies => destination unreachable.\n";
        return;
    }

    sort(replies.begin(), replies.end(), [](auto& a, auto& b){ return a.FD < b.FD; });
    cout << g.name[local] << " chooses new Successor = " << g.name[replies[0].via]
         << " (FD=" << replies[0].FD << ")\n";
    cout << g.name[local] << " returns to PASSIVE.\n";
}

int main() {
    Graph g;

    // Nodes
    int B  = g.addNode("Branch");
    int R1 = g.addNode("R1");
    int R2 = g.addNode("R2(Internet)");
    int R3 = g.addNode("R3");

    // Links (costs are just example metrics)
    // Branch--R1--R2 is "main" path, Branch--R3--R2 is "backup"
    g.addUndirected(B,  R1, 10);
    g.addUndirected(R1, R2, 10);

    g.addUndirected(B,  R3, 20);
    g.addUndirected(R3, R2, 5);

    cout << "Topology:\n";
    cout << "  Branch--10--R1--10--R2(Internet)\n";
    cout << "  Branch--20--R3--5---R2(Internet)\n";

    // 1) Normal: compute successor & feasible successor
    printCandidates(g, B, R2);

    // 2) Fail the primary link Branch-R1
    cout << "\n*** Event: link Branch <-> R1 goes DOWN ***\n";
    g.setLink(B, R1, false);

    // Recompute: if FS existed, Branch should switch without diffusing query
    printCandidates(g, B, R2);

    // In this topology, after B-R1 down, only R3 remains -> still fine.
    // 3) Now also fail Branch-R3 to force "no candidates" and show diffusing query behavior
    cout << "\n*** Event: link Branch <-> R3 goes DOWN too (force ACTIVE) ***\n";
    g.setLink(B, R3, false);

    printCandidates(g, B, R2);

    // Trigger simplified diffusion (will show queries and replies, then fail)
    simulateDiffusingQuery(g, B, R2);

    return 0;
}
```

动态路由协议最大的问题是：

- 怎么保证没有环路（loop）
- 怎么快速收敛
- 怎么有备份路径

DUAL 就是解决这些问题的

想象 3 条路通向 Internet：

```
Branch
   |10
  R1
   |5
  R2 (Internet)
```

距离越小越好。

EIGRP 会算出：

```
总距离 = 本地到邻居 + 邻居到目的地
```

这就是 EIGRP 的“距离计算逻辑”。

# Successor 是什么？

## 定义

> Successor = 当前“最优路径”

也就是：

👉 被选进路由表的那条路径

------

例如：

```
P ::/0, 1 successors, FD is 2816
```

说明：

- 有 1 条最优路径
- FD = Feasible Distance（总距离）

------

## Successor 特点

| 特点           | 说明              |
| -------------- | ----------------- |
| 必须是最小距离 | metric 最小       |
| 无环           | 保证不会形成 loop |
| 会进路由表     | 会被真正使用      |

------

# 三、Feasible Successor 是什么？

这个是 EIGRP 最牛的地方 🔥

------

## 定义

> Feasible Successor = 备份路径

但是不是随便的备份。

它必须满足：

### Feasibility Condition（可行性条件）

邻居报告的距离 < 当前最优路径的距离

听不懂？没关系。

------

## 用简单例子

假设：

```
Branch 有两条路去 Internet：

路径A：距离 20
路径B：距离 30
```

那：

- 路径A = Successor
- 路径B = 备份候选

但是 EIGRP 不是随便选备份。

它会看：

> 路径B 的“报告距离”是不是比路径A的总距离小？

如果满足，才是 Feasible Successor。

------

## 为什么这么设计？

为了保证：

👉 备份路径绝对不会形成环路

这是 EIGRP 比 OSPF 更智能的地方。

好问题 👍🔥

你这个问题说明你已经在真正理解 DUAL 了。

你问：

> 为什么 DUAL 的例子里会出现 Dijkstra？

答案是：

👉 **因为那段代码不是在实现真正的 DUAL，而是用 Dijkstra 来“模拟全网已经收敛后的最短路径结果”。**

------

# 一、关键结论

> 🚨 EIGRP 的 DUAL 算法 ≠ Dijkstra 算法
>
> 它们是完全不同的算法。

------

# 二、那为什么代码里用了 Dijkstra？

因为：

我们写的是一个**小模拟程序**，它需要一个方法来：

```
算出：
“邻居到目的地的距离是多少？”
```

也就是：

```cpp
RD = neighbor 到 destination 的最短距离
```

在真实网络中：

- 邻居会通过 EIGRP update 告诉你 RD
- 不需要你自己算

但在程序里：

- 没有真实网络
- 没有 update 消息
- 没有邻居通信

所以我们用 Dijkstra：

👉 直接计算每个节点到目标的最短路径
👉 当成“邻居告诉你的 RD”

它只是一个数学工具。

------

# 三、真正的 DUAL 是怎么工作的？

DUAL 是：

> 分布式算法（distributed algorithm）

什么意思？

- 每个路由器只知道邻居信息
- 不知道整个拓扑
- 不会跑全网 SPF
- 通过 Query / Reply 机制逐步计算

------

# 四、Dijkstra vs DUAL 对比

|                  | Dijkstra | DUAL             |
| ---------------- | -------- | ---------------- |
| 类型             | 全局算法 | 分布式算法       |
| 是否知道全网拓扑 | 是       | 否               |
| 是否重新算全图   | 是       | 否               |
| 是否提前算好备份 | 否       | 是               |
| 收敛速度         | 快       | 更快（有 FS 时） |

------

# 五、为什么 OSPF 用 Dijkstra？

OSPF 是 Link-State 协议：

- 每台路由器知道全网拓扑
- 每次变化都跑 SPF（Dijkstra）

------

# 六、为什么 EIGRP 不用 Dijkstra？

EIGRP 是 Distance-Vector（增强版）：

- 只知道邻居报告的距离（RD）
- 通过可行性条件避免环路
- 不用知道全网拓扑
- 不用跑 SPF

------

# 七、那我们代码里在干什么？

我们那段代码在做两件事：

1️⃣ 用 Dijkstra 预先算好“真实世界的最短路径”

2️⃣ 再用 DUAL 的规则：

```cpp
Successor = FD 最小
Feasible Successor = RD < FD(successor)
```

来决定主路和备份。

所以：

> Dijkstra 是“环境模拟器”
> DUAL 是“选路规则”

------

# 八、如果不用 Dijkstra 能写吗？

可以。

你可以改成：

- 每个节点手动写 RD
- 不跑全图计算
- 完全按 DUAL 消息机制模拟

但那样代码会复杂很多（需要模拟 Query/Reply 状态机）。

------

# 九、真正的 DUAL 更复杂的地方

真实 DUAL 还包含：

- Active / Passive 状态机
- Query 扩散
- Reply 汇总
- stuck-in-active 计时器
- feasibility condition 检查

那是一个完整分布式状态机。

------

# 十、总结一句话

代码里有 Dijkstra：

👉 只是为了“算 RD”
👉 不是因为 EIGRP 用 Dijkstra
👉 真实 EIGRP 完全不跑 SPF

------



