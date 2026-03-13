

### The Problem

The exhibit shows that the **NMS Server (10.221.10.11)** is expected to use specific community strings for Read-Only (RO) and Read-Write (RW) access.

Looking at the "Communities" note next to the NMS Server:

- **Read Only:** `Cisc0Us3r`
- **Read/Write:** `Ciscowruser`

Now, look at the current configuration on **UserSW2**:

- `snmp-server community Cisc0Us3r RO 11` (Correct for RO)
- `snmp-server community CiscoWrus3r RW 11` (**Typo!** The switch is configured with `CiscoWrus3r`, but the server expects `Ciscowruser`).

Because the community string configured on the switch does not match what the NMS server is sending, the "Write" operations (modifying configurations) fail.

------

### Why Option B is Correct

**`snmp-server community Ciscowruser RW 11`**

1. **Matches the String:** It uses `Ciscowruser`, which matches the NMS server's requirements.
2. **Permissions:** It grants `RW` (Read-Write) access, which is necessary to "upload modified configurations."
3. **Security:** It references `access 11`, which permits the NMS Server's IP (`10.221.10.11`).

### Why the others fail

- **A:** This attempts to give RW access to the *Read-Only* string (`Cisc0Us3r`). While it might technically work if you changed the server settings, it doesn't follow the design provided in the exhibit.
- **C:** This references `access 22`. Access list 22 permits `10.221.10.12`, which is **not** the IP of the NMS server shown in the diagram.
- **D:** This is a v3 group configuration for a v2c string, which is syntactically messy and doesn't address the primary community string mismatch.

------

Since you're working through these CCNP-style questions, would you like me to explain the differences between how **ACLs** are applied to SNMP v2c versus SNMP v3 groups?