```
Router eigrp 1!
Route-map Leak-Route deny 10!
Interface Serial 0/0
Ip summary-address eigrp 1 10.0.0.0 255.0.0.0 leak-map Leak-Route
```

### Understanding the EIGRP Leak-Map

In EIGRP, when you configure a summary address (like the `10.0.0.0/8` above), the router’s default behavior is **suppression**. It will advertise the summary route to its neighbors and "hide" all the specific routes that fall within that range (e.g., `10.1.1.0/24`, `10.5.0.0/16`).

A **leak-map** is the exception to this rule. It allows you to advertise the summary route while simultaneously "leaking" specific component routes out of the summary.

#### 1. How the Logic Works (The "Reverse" Trap)

This is where most students lose points on the CCNP. Unlike a redistribution route-map where `deny` means "don't send," in an EIGRP leak-map, the logic applies to the **exception list**:

- **Route-map PERMIT:** "I permit this specific route to be leaked." (Neighbor sees the **Summary + Specific**).
- **Route-map DENY:** "I deny the leak for this route." (Neighbor sees **Summary only**).

In your extracted text, `Route-map Leak-Route deny 10` is effectively telling the router **not** to leak whatever is in sequence 10. This is why the customer in the previous question wasn't receiving their specific loopback route.

#### 2. Why use a Leak-Map?

- **Traffic Engineering:** You want most traffic to follow the summary route, but you want to force traffic for one specific subnet (like a critical server or a high-speed link) to take a specific path.
- **Redundancy:** If you have two exits, you might summarize on both, but leak a specific route on one side to ensure it is the preferred entry point for that specific subnet.

#### 3. The Configuration Components

To make a leak-map work, you need three pieces:

1. **The Match Criterion:** Usually an Access Control List (ACL) or Prefix-List identifying the specific route you want to see.

2. **The Route-Map:** References the ACL and uses the `permit` action to allow the leak.

3. **The Interface Command:** Applied under the specific interface where the summary is configured.

4. If you wanted to leak `10.1.1.0/24` out of your `10.0.0.0/8` summary, the correct configuration would look like this:

   ```
   ip access-list standard AL_LEAK_THESE
    permit 10.1.1.0 0.0.0.255
   
   route-map RM_LEAK_MAP permit 10
    match ip address AL_LEAK_THESE
   
   interface Serial 0/0
    ip summary-address eigrp 1 10.0.0.0 255.0.0.0 leak-map RM_LEAK_MAP
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