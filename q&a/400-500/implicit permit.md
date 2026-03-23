This is a classic "Implicit Permit" logic trap in Cisco Route Maps. Here is why **Remove route-map sequence 20** is the correct answer:

### The Logic of the Route-Map

In Cisco IOS, a `route-map` behaves like a series of "If-Then" statements. If a prefix doesn't match a specific sequence, it moves to the next one.

1. **Sequence 10:** This matches networks `172.16.1.0/24` and `172.16.2.0/24` via ACL 10. These are permitted and tagged with 666.
2. **Sequence 20:** The exhibit shows `route-map Redistribution_EIGRP, permit, sequence 20` but **no match clauses** are defined under it.

### Why 172.16.10.0/24 is leaking

In a route-map, if a `permit` sequence has **no match clause**, it acts as a "catch-all" (Match Any). It effectively says: *"Permit everything else that hasn't been matched yet."*

Because `172.16.10.0/24` does not match ACL 10 in sequence 10, the router evaluates sequence 20. Since sequence 20 is a `permit` statement with no restrictions, it allows the prefix to be redistributed.

------

### How to Fix It

To stop the "leaking" of unintended prefixes, you have two main options, but **C** is the most direct fix for this configuration:

- **Option C (Remove Seq 20):** By deleting sequence 20, any prefix that doesn't match Sequence 10 will hit the **implicit deny** at the end of the route-map and be dropped.
- **Alternative (Not listed):** Change sequence 20 to a `deny` statement, or add a specific match clause to it that excludes `172.16.10.0/24`.

```
Rl# show route-map
route-map Redistribution_EIGRP, permit, sequence 10
Match clauses:ip address (access-lists): 10
Set clauses:
tag 666
Match clauses:
Policy routing matches: 0 packets, 0 bytes
route-map Redistribution_EIGRP, permit, sequence 20
Set clauses:
Policy routing matches: 0 packets, 0 bytes
Rl# show access-listsStandard IP access list 10
10 permit 172.16.1.0, wildcard bits 0.0.0.25520 permit 172.16.2.0, wildcard bits 0.0.0.255
```

