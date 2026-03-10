To complete the configuration on **R1** and ensure the flow data actually leaves the interface and reaches the NMS, you need to tie the **Record**, **Exporter**, and **Monitor** together.

Since the **Exporter** ($10.221.10.11$) is now corrected, here is the full syntax to apply it to the interface:

### 1. Create the Flow Monitor

The monitor is the "glue." It tells the router which data to collect (the record) and where to send it (the exporter).

Bash

```
R1(config)# flow monitor FlowMonitor1
R1(config-flow-monitor)# record netflow-original
R1(config-flow-monitor)# exporter FlowAnalyzer1
R1(config-flow-monitor)# exit
```

### 2. Apply to the Interface

NetFlow is directional. To see traffic coming from the ISP or internal users, you must apply the monitor to the physical or logical interface.

Given the topology, you would likely want to monitor traffic entering the router from the internal network:

Bash

```
R1(config)# interface Ethernet0/1
R1(config-if)# ip flow monitor FlowMonitor1 input
R1(config-if)# exit
```

------

### Verification Commands

Once applied, use these commands to ensure the "Flow" is actually working:

- **`show flow monitor FlowMonitor1 statistics`**: Check if the cache is actually being populated.
- **`show flow exporter FlowAnalyzer1 statistics`**: Confirm that export packets are being sent (and not dropped due to routing issues).

### A Note on the "Source Interface"

In your original image, the source interface was **Ethernet0/1**. Since the NMS is connected via **Core_Sw1** (which connects to E0/1), this is correct. It ensures the router uses its "internal" IP address to talk to the NMS, which usually avoids firewall or ISP routing issues.

Would you like to move on to a different topic, like **OSPF** or **BGP** troubleshooting scenarios?