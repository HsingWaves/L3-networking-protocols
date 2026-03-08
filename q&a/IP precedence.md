

------

## 先读配置

```cisco
access-list 1 permit 209.165.200.215
access-list 2 permit 209.165.200.216

interface ethernet 1
 ip policy route-map Texas

route-map Texas permit 10
 match ip address 1
 set ip precedence priority
 set ip next-hop 209.165.200.217

route-map Texas permit 20
 match ip address 2
 set ip next-hop 209.165.200.218
```

------

## 题目要求

- 源 `209.165.200.215` 的包，precedence 要设为 **1**
- 源 `209.165.200.216` 的包，precedence 要设为 **5**

------

## 先看第一个源 .215

它匹配的是：

```cisco
route-map Texas permit 10
 match ip address 1
 set ip precedence priority
```

这里 `priority` 对应的 precedence 值就是 **1**。

所以 **permit 10 已经正确**，不用改。

------

## 再看第二个源 .216

它匹配的是：

```cisco
route-map Texas permit 20
 match ip address 2
 set ip next-hop 209.165.200.218
```

这里只有 next-hop，**没有设置 precedence**，所以题目要求的 precedence 5 还没实现。

precedence **5** 的关键字是：

```cisco
critical
```

对应关系常考：

- `routine` = 0
- `priority` = 1
- `immediate` = 2
- `flash` = 3
- `flash-override` = 4
- `critical` = 5
- `internetwork` = 6
- `network` = 7

所以要在 **route-map Texas permit 20** 下加：

```cisco
set ip precedence critical
```

------

## 选项判断

### A. `set ip precedence critical in route-map Texas permit 20`

对。
正好把 `.216` 这类流量设成 precedence 5。

### B. `set ip precedence critical in route-map Texas permit 10`

错。
permit 10 是 `.215`，题目要求它是 precedence 1，不是 5。

### C. `set ip precedence priority in route-map Texas permit 20`

错。
`priority = 1`，不是 5。

### D. `set ip precedence immediate in route-map Texas permit 10`

错。
`immediate = 2`，而且 permit 10 本来就该是 1。

------

# 结论

**答案：A**

可以补成：

```cisco
route-map Texas permit 20
 match ip address 2
 set ip precedence critical
 set ip next-hop 209.165.200.218
```

------

### 这题顺手记住

**IP Precedence 关键字映射：**

- priority = 1
- immediate = 2
- flash = 3
- flash-override = 4
- critical = 5

