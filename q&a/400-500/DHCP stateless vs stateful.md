The main difference between **Stateful** and **Stateless** DHCPv6 comes down to who is in charge of assigning the IP address and how much "memory" the server has of the transaction.

In the networking world, "State" refers to whether the server keeps a record (a binding) of which client has which address.

------

### 1. Stateful DHCPv6 (The "Full Control" Mode)

This is the IPv6 equivalent of traditional DHCP for IPv4.

- **The Server is Boss:** The DHCPv6 server chooses the interface address (the 128-bit IPv6 address) from a pool and assigns it to the client.
- **The Binding Table:** The server maintains a "State" table. It knows exactly which DUID (Device Unique Identifier) has which lease and when it expires.
- **Use Case:** Best for environments where you need strict auditing and control over exactly which IP each device is using (e.g., enterprise desktops).

------

### 2. Stateless DHCPv6 (The "Information Only" Mode)

In this mode, the DHCPv6 server **does not** give out IP addresses. Instead, the client uses **SLAAC** (Stateless Address Autoconfiguration) to create its own address.

- **The Client is Boss:** The client listens to a Router Advertisement (RA) to get the prefix (e.g., `2001:db8:1::/64`) and then generates its own interface ID (usually via EUI-64 or a random generation).
- **The Server's Role:** The DHCPv6 server only providing "additional information" that SLAAC doesn't provide, such as **DNS server addresses**, domain names, or NTP servers.
- **No Binding Table:** The server has no "State." It doesn't know or care what IP addresses the clients generated for themselves; it just answers requests for DNS info.
- **Use Case:** Best for massive-scale deployments or mobile devices where keeping a massive lease database is unnecessary overhead.

------

### Comparison Table

| **Feature**            | **Stateful DHCPv6**                                          | **Stateless DHCPv6**                    |
| ---------------------- | ------------------------------------------------------------ | --------------------------------------- |
| **Address Assignment** | Provided by DHCP Server                                      | Client generates it (SLAAC)             |
| **Server Records**     | Maintains a binding/lease table                              | No record of client addresses           |
| **RA Flags**           | Managed Address Config (**M=1**)                             | Other Config (**O=1**)                  |
| **Complexity**         | Higher (Server management)                                   | Lower (Plug-and-play)                   |
| **Protocol Flow**      | Solicit $\rightarrow$ Advertise $\rightarrow$ Request $\rightarrow$ Reply | Information-Request $\rightarrow$ Reply |

### The "M" and "O" Flags

Since you’re studying for the CCNP, keep an eye on the **Router Advertisement (RA)** flags. This is how the router tells the clients which method to use:

- **M-Flag (Managed):** Set to 1 for Stateful. "Use DHCP for everything."
- **O-Flag (Other):** Set to 1 for Stateless. "Get your IP from me (RA), but get your 'other' info (DNS) from the DHCP server."

Would you like to see the Cisco CLI commands to configure these specific RA flags on a router interface?