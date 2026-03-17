**Named IPv4 access lists can be configured either as standard or extended access lists.**

While numbered ACLs use specific ranges (e.g., 1–99 for standard, 100–199 for extended), named ACLs allow you to define the type explicitly in the command line before giving it a descriptive name.

| **ACL Type**       | **Command Example**                       |
| ------------------ | ----------------------------------------- |
| **Named Standard** | `ip access-list standard MY_STANDARD_ACL` |
| **Named Extended** | `ip access-list extended MY_EXTENDED_ACL` |

One of the biggest advantages of using named ACLs (besides the descriptive name) is the ability to **edit specific lines** using sequence numbers without having to delete and re-create the entire list.