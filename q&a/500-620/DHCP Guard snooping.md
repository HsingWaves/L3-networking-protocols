

**DHCPv6 Guard** is a Layer 2 security feature designed to block **rogue DHCPv6 servers**. It works by identifying "trusted" ports (where your real server is) and "untrusted" ports (where users are). If a "Reply" or "Advertise" message (server-side messages) comes from an untrusted port, the switch drops it to prevent users from getting wrong IP information from a malicious source.

------

## Is there a DHCPv4 version?

**Yes**, but it has a different name. In the IPv4 world, this is called **DHCP Snooping**.

They perform the exact same job:

- **DHCP Snooping (IPv4):** Prevents rogue DHCPv4 servers by blocking unauthorized `OFFER` and `ACK` packets.
- **DHCPv6 Guard (IPv6):** Prevents rogue DHCPv6 servers by blocking unauthorized `ADVERTISE` and `REPLY` packets.

------

## Why D is incorrect

Option D suggests it blocks messages from **relay agents to a server**. This is the opposite of what you want. You *want* relay agents to be able to talk to servers; you want to block **unauthorized servers** from talking to your clients.

## Quick Comparison Table

| **Feature**     | **IPv4 Version**                 | **IPv6 Version**                       | **Primary Goal**      |
| --------------- | -------------------------------- | -------------------------------------- | --------------------- |
| **Name**        | DHCP Snooping                    | DHCPv6 Guard                           | Block Rogue Servers   |
| **Trust Logic** | Trusted vs. Untrusted Ports      | Trusted vs. Untrusted Ports            | Identity verification |
| **Action**      | Drops `OFFER`/`ACK` on untrusted | Drops `ADVERTISE`/`REPLY` on untrusted | Filter traffic        |

------

### Pro-Tip for CCNP

Cisco often bundles **DHCPv6 Guard** under a larger umbrella called **IPv6 First-Hop Security (FHS)**. This suite also includes:

- **RA Guard:** Blocks rogue Router Advertisements.
- **Source Guard:** Prevents IP spoofing.

