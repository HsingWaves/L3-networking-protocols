| **Level** | **Name**      | **Description**                   |
| --------- | ------------- | --------------------------------- |
| 0         | Emergencies   | System is unusable                |
| 1         | Alerts        | Immediate action needed           |
| **2**     | **Critical**  | **Critical conditions**           |
| 3         | Errors        | Error conditions                  |
| 4         | Warnings      | Warning conditions                |
| 5         | Notifications | Normal but significant conditions |
| 6         | Informational | Informational messages            |
| 7         | Debugging     | Debugging messages                |



The requirement is to limit logs to "**critical and higher**" severity.

- Referring to the table, **Critical** is Level 2.
- "Higher severity" in the Syslog world means lower numbers (Level 1 and Level 0).
- Therefore, the command `logging console 2` tells the router: "Only send messages of level 2, 1, and 0 to the console."

L**ogging console 5:** This would show Level 5 (Notifications) and everything above it (4, 3, 2, 1, 0). This is far too many messages for the requirement.

**no logging console:** This would disable console logging entirely, which isn't what was asked.

| **Level** | **Keyword**       | **Mnemonic Word** |
| --------- | ----------------- | ----------------- |
| **0**     | **E**mergencies   | **E**very         |
| **1**     | **A**lerts        | **A**lley         |
| **2**     | **C**ritical      | **C**at           |
| **3**     | **E**rrors        | **E**ats          |
| **4**     | **W**arnings      | **W**hen          |
| **5**     | **N**otifications | **N**obody        |
| **6**     | **I**nformational | **I**s            |
| **7**     | **D**ebugging     | **D**ogs          |

Every Alley Cats eat when nobody is dog



SLA

Whenever you see an IP SLA that is "Down" or showing "No Returns," always check the **Operational State** or the **Next Scheduled Start Time**. If it isn't scheduled to start "now" or at a specific time, the tracking object will never transition to "Up."

Next Scheduled Start Time: Pending trigger



**We Love AS Orange Mellons Eating in Old red nets**

| **Mnemonic Word** | **Attribute**        | **Preference**         |
| ----------------- | -------------------- | ---------------------- |
| **W**e            | **Weight**           | Highest                |
| **L**ove          | **Local Preference** | Highest                |
| **A**S            | **AS Path**          | Shortest               |
| **O**range        | **Origin**           | IGP > EGP > Incomplete |
| **M**ellons       | **MED**              | Lowest                 |
| **E**ating        | **eBGP over iBGP**   | External preferred     |
| **I**n            | **IGP metric**       | Lowest to Next-Hop     |
| **O**ld           | **Oldest Path**      | Most stable (eBGP)     |
| **R**ed           | **Router ID**        | Lowest                 |
| **N**ets          | **Neighbor IP**      | Lowest                 |