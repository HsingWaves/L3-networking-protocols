```
Router eigrp 1!
Route-map Leak-Route deny 10!
Interface Serial 0/0
Ip summary-address eigrp 1 10.0.0.0 255.0.0.0 leak-map Leak-Route
```



### The Logic Behind the Fix

The provided configuration has a major flaw in the `route-map`. In the image, the route-map is currently set to `deny 10`. In the context of a leak-map, if the route-map result is **deny**, the route is **suppressed** (not leaked). To allow the loopback to be advertised alongside the summary, you need a **permit** action.

1. **D. route-map Leak-Route permit 10 and match access-list 1:** This changes the action from `deny` to `permit`. This tells EIGRP: "If a route matches ACL 1, allow it to be leaked (advertised)."
2. **A. access-list 1 permit 10.1.1.0 0.0.0.3:** This defines the specific route to be leaked. Since the loopback is `10.1.1.1` with a mask of `255.255.255.252` (a `/30`), the network ID is `10.1.1.0`. The wildcard mask for a `/30` is `0.0.0.3`.

------

### Why the others are wrong:

- **B:** The wildcard mask `0.0.0.252` is incorrect for a `/30`. That would be for a `/22` (if it were `0.0.3.255`).
- **C:** Simply matching under the route-map isn't enough if the route-map is still set to `deny`.
- **E:** Adding a `permit 20` to the route-map without a match statement would potentially leak *every* route covered by the summary, defeating the purpose of summarization entirely.

### Key Concept: The "Reverse" Logic of Leak-Maps

It is easy to get tripped up here. In a standard redistribution route-map, `deny` means "don't redistribute." In an EIGRP leak-map:

- **Permit** in Route-Map = **Leak** the route (advertise specific + summary).
- **Deny** in Route-Map = **Suppress** the route (advertise summary only).

Since you've been working on OSPF and BGP lately, it’s worth noting that EIGRP’s leak-map is quite unique—OSPF doesn't have a direct equivalent for internal routes (it usually requires un-summarizing or using multiple range commands).