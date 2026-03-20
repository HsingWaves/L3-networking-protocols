### Why Micro-BFD?

Standard BFD typically runs on the logical bundle interface (the Port-Channel). If you have four 10Gbps links in a LAG and one of them fails or experiences "soft failure" (where the link is physically up but cannot pass traffic), a standard BFD session might still stay **UP** because it's only checking the aggregate path.

This results in a "black hole" where 25% of your traffic is dropped because the hashing algorithm continues to send packets over the broken member link.

### The Goals of Micro-BFD (RFC 7130)

1. **Verification of Continuity (Choice C):** It ensures that every single physical path within the bundle is actually capable of forwarding Layer 3 traffic. If a specific member link fails the BFD test, it is removed from the LAG even if the physical light (L1) is still present.
2. **Individual Member Link Sessions (Choice E):** Instead of one session for the whole bundle, a separate "micro" BFD session is established on **each** physical member link.

------

### Comparison Table

| **Feature**           | **Standard BFD on LAG**       | **Micro-BFD (RFC 7130)**        |
| --------------------- | ----------------------------- | ------------------------------- |
| **Session Target**    | Logical Bundle Interface      | Individual Physical Members     |
| **Failure Detection** | Total aggregate failure       | Single link "soft" failure      |
| **Traffic Impact**    | May cause partial packet loss | Removes bad links instantly     |
| **Addressing**        | Uses Peer IP                  | Often uses Link-Local/Multicast |