To achieve a **granular level** of logging in Cisco IOS, you should use the **msec** (millisecond) option.



------

### Why "msec"?

By default, Cisco service timestamps usually provide logs in a date/time format (or uptime) that only goes down to the second. In high-speed networking or complex troubleshooting (like the BGP and OSPF flaps seen in your exhibit), multiple events can happen within a single second.

Adding milliseconds allows you to see the exact sequence of events, which is critical for:

- **Debugging race conditions:** Seeing which protocol went down first.
- **Packet timing:** Correlating logs with packet captures (PCAPs).
- **Convergence analysis:** Measuring exactly how long a failover took.

### Command Example

To enable this on a Cisco router, you would use:

```
Router(config)# service timestamps log datetime msec
```

### Comparison of Options

| **Option**   | **Description**                                          | **Granularity**         |
| ------------ | -------------------------------------------------------- | ----------------------- |
| **uptime**   | Shows time since the router last booted.                 | Seconds                 |
| **datetime** | Shows the actual date and time (requires NTP/Clock set). | Seconds                 |
| **msec**     | Appends milliseconds to the datetime or uptime.          | **Milliseconds (High)** |
| **timezone** | Adds the timezone offset (e.g., UTC, CET).               | N/A (Contextual)        |

Since your exhibit shows protocols like BGP and OSPF transitioning, milliseconds are the standard "best practice" for granular troubleshooting.

Would you like me to explain how to synchronize the router clock so these timestamps are actually accurate?