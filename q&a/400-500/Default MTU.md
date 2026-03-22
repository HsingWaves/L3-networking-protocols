In Cisco networking, **1500 bytes** is the standard, implicit **default MTU** (Maximum Transmission Unit) for Ethernet interfaces.

Here is how you can tell from the exhibit that R1 is using 1500:

### 1. The "Default" Principle

Look at the `show running-config interface FastEthernet1/0` output for **R1**. You will notice that there is **no `ip mtu` command** listed.

- In Cisco IOS, if a non-default value is configured (like the `ip mtu 1400` we see on R2), it shows up in the configuration.
- If the command is missing, the interface reverts to the factory default for that media type. For FastEthernet, that default is always **1500**.

### 2. Comparing the Configs

The exhibit specifically shows both configurations to highlight the difference:

- **R2** has an explicit override: `ip mtu 1400`
- **R1** has no such line.

### 3. OSPF State Behavior

The fact that the routers are stuck in **EXSTART/EXCHANGE** is a "smoking gun" for an MTU mismatch in OSPF.

- When R1 (MTU 1500) sends a Database Description (DBD) packet to R2 (MTU 1400), R2 sees that the MTU in the packet header is larger than its own interface can handle.
- R2 will then drop that packet, preventing the adjacency from ever reaching **FULL**.

------

### Verification Command

If you were logged into the actual router, you would confirm this by running: `R1# show interface fa1/0`

In the first few lines of the output, you would see: `MTU 1500 bytes, BW 100000 Kbit/sec, DLY 100 usec...`