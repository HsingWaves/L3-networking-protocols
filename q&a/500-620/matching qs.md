| **SNMPv2c**          | **SNMPv3**                |
| -------------------- | ------------------------- |
| **community string** | **username and password** |
| **no encryption**    | **authentication**        |
| **read-only**        | **privileged**            |

**SNMPv2c:** This version is inherently insecure. It uses a **community string** (essentially a shared password sent in clear text) to permit access. It lacks modern **encryption**, meaning any packet sniffer can see the data and the community string. Its access levels are usually basic, like **read-only** or read-write.

**SNMPv3:** This is the "gold standard" for secure monitoring. It moves away from community strings to a **username/password** model.

- **Authentication (Auth):** Ensures the person connecting is who they say they are (using MD5 or SHA).
- **Privileged/Encryption (Priv):** Ensures the data is encrypted (using DES or AES) so it cannot be read by unauthorized parties.



| **MPLS Term**                          | **Definition / Function**                                    |
| -------------------------------------- | ------------------------------------------------------------ |
| **Penultimate Hop Popping**            | **Allows an LSR to remove the label before forwarding the packet** |
| **Label Edge Router (LER)**            | **Accepts unlabeled packets and imposes labels**             |
| **Forwarding Equivalence Class (FEC)** | **Group of packets that are forwarded in the same manner**   |
| **Label Switch Router (LSR)**          | **Receives labeled packets and swaps labels**                |



**Label Edge Router (LER):** Also known as the Ingress/Egress PE (Provider Edge). Its primary job is **Label Imposition** (pushing a label onto a packet coming from a customer) or **Label Disposition** (removing it).

**Label Switch Router (LSR):** These are core routers (P routers). They don't care about the IP payload; they look at the incoming label, consult their LFIB (Label Forwarding Information Base), and **swap** it for an outgoing label.

**Forwarding Equivalence Class (FEC):** This is a set of packets with similar characteristics (like the same destination prefix or QoS marking) that follow the same path. In MPLS, all packets in a specific FEC are assigned the same label.

**Penultimate Hop Popping (PHP):** To save the very last router (Egress LER) from performing two lookups (one for the label and one for the IP destination), the router *just before* it removes the label. This is triggered by a "Label 3" (Implicit Null) advertisement.



| **Step**   | **Action**                                                   |
| ---------- | ------------------------------------------------------------ |
| **Step 1** | **Download the Cisco IOS image to the TFTP Server**          |
| **Step 2** | **Copy the IOS image in the file system** (e.g., `copy tftp: flash:`) |
| **Step 3** | **Verify the configuration register & boot variable**        |
| **Step 4** | **Save the configuration & Reload the router.**              |



A common "gotcha" on the exam is the difference between a "poor" score and a "no data" status.

### DNA Center Client Health Scores

| **Health Color** | **Health Score Range**                   |
| ---------------- | ---------------------------------------- |
| **Red**          | **Health Score is 1 to 3** (Poor)        |
| **Orange**       | **Health Score is 4 to 7** (Fair)        |
| **Green**        | **Health Score is 8 to 10** (Good)       |
| **Gray**         | **Health Score is 0** (No data/Inactive) |





| **NDP Message**                 | **ICMPv6 Type**     |
| ------------------------------- | ------------------- |
| **Router Solicitation (RS)**    | **ICMPv6 Type 133** |
| **Router Advertisement (RA)**   | **ICMPv6 Type 134** |
| **Neighbor Solicitation (NS)**  | **ICMPv6 Type 135** |
| **Neighbor Advertisement (NA)** | **ICMPv6 Type 136** |
| **Redirect Message**            | **ICMPv6 Type 137** |

**Router Solicitation (133):** Sent by a host to find routers on the local link. Usually sent to the "all-routers" multicast address (**ff02::2**).

**Router Advertisement (134):** Sent by routers periodically or in response to an RS. This is how hosts learn about prefixes for **SLAAC** (Stateless Address Autoconfiguration) and their default gateway.

**Neighbor Solicitation (135):** The IPv6 version of an **ARP Request**. It asks a neighbor for its MAC address or is used for Duplicate Address Detection (DAD).

**Neighbor Advertisement (136):** The IPv6 version of an **ARP Reply**. It confirms a neighbor's MAC address in response to an NS.

**Redirect (137):** Used by routers to inform a host that a better first-hop router exists for a specific destination.





| **MPLS Term**                          | **Definition / Function**                                    |
| -------------------------------------- | ------------------------------------------------------------ |
| **LSR** (Label Switch Router)          | **Routers in the core of the provider network known as P routers** |
| **FEC** (Forwarding Equivalence Class) | **All traffic to be forwarded using the same path and same label** |
| **LER** (Label Edge Router)            | **Routers that connect to the customer routers known as PE routers** |
| **LDP** (Label Distribution Protocol)  | **Used for exchanging label mapping information between MPLS enabled routers** |
| **LSP** (Label Switched Path)          | **Path along which the traffic flows across an MPLS network** |



**LDP (Label Distribution Protocol):** This is the "language" routers use to talk to each other about labels. If Router A has a route to `10.1.1.0/24`, it uses LDP to tell its neighbors, "Hey, if you want to send traffic to this network, slap label 25 on it and send it to me."

**LSP (Label Switched Path):** Think of this as the "tunnel." It is the unidirectional path from the ingress LER to the egress LER. While the underlying routing protocol (like OSPF or IS-IS) determines the physical path, the LSP is the logical path defined by the labels.

**LER vs. LSR:** * The **LER (PE)** sits at the edge. It does the heavy lifting of looking at IP headers and assigning them to a **FEC**.

- The **LSR (P)** sits in the middle. It only looks at the labels, making it extremely fast because it doesn't have to perform complex longest-prefix matches in the routing table.





| **Security Feature**   | **Definition / Function**                                    |
| ---------------------- | ------------------------------------------------------------ |
| **IPv6 RA Guard**      | **Block a malicious host and permit the router from a legitimate route.** |
| **IPv6 DHCPv6 Guard**  | **Block reply and advertisement messages from unauthorized DHCP servers and relay agents.** |
| **IPv6 ND Inspection** | **Create a binding table that is based on NS and NA messages.** |
| **IPv6 Source Guard**  | **Filter inbound traffic on Layer 2 switch ports that are not in the IPv6 binding table.** |
| **IPv6 Binding Table** | **Create IPv6 neighbors connected to the device from information sources such as NDP snooping.** |