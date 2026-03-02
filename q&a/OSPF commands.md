

------

# 🧠 OSPF Expert Command Map (≈50 Core Commands)

We’ll divide them into 8 categories:

1. Process Basics
2. Interface Control
3. Cost & Metrics
4. Area Types
5. Summarization
6. Filtering
7. Redistribution
8. Advanced Tuning & Stability

------

# 1️⃣ Process-Level Commands (Core Brain)

```bash
router ospf 1
router-id 1.1.1.1
log-adjacency-changes
passive-interface g0/1
no passive-interface g0/1
auto-cost reference-bandwidth 10000
default-information originate
default-information originate always
max-lsa 1000
timers throttle spf 10 100 500
timers throttle lsa all 10 100 500
graceful-restart
```

## 🖼 Illustration – OSPF Process Layer

```
        OSPF Process 1
        ┌───────────────┐
        │ Router-ID     │
        │ SPF Engine    │
        │ LSA Database  │
        │ Area Mapping  │
        └───────────────┘
```

This is the control plane brain.

------

# 2️⃣ Interface-Level Activation

```bash
ip ospf 1 area 0
ip ospf cost 10
ip ospf priority 100
ip ospf network point-to-point
ip ospf network broadcast
ip ospf hello-interval 5
ip ospf dead-interval 20
ip ospf authentication
ip ospf authentication-key cisco
ip ospf message-digest-key 1 md5 cisco
```

## 🖼 Illustration – Adjacency Formation

```
R1 -------- R2
Hello → ← Hello
DBD   → ← DBD
LSR   → ← LSU
Full adjacency
```

Timers + network type control this behavior.

------

# 3️⃣ OSPF Cost Control

```bash
auto-cost reference-bandwidth 10000
ip ospf cost 50
bandwidth 1000
```

## 🖼 Cost Formula

```
Cost = Reference BW / Interface BW

Example:
Ref BW = 10000 Mbps
Interface = 1000 Mbps

Cost = 10
```

Without adjusting reference bandwidth, Gig links all look equal.

------

# 4️⃣ Area Types (Topology Control)

```bash
area 1 stub
area 1 stub no-summary
area 1 nssa
area 1 nssa no-summary
area 1 default-cost 20
area 1 range 10.1.0.0 255.255.0.0
area 1 filter-list prefix FILTER in
```

## 🖼 Area Types Overview

```
           Area 0
              |
      -------------------
      |                 |
    Stub             NSSA
```

Stub = No Type 5
NSSA = Type 7 allowed

------

# 5️⃣ Summarization

```bash
area 1 range 10.0.0.0 255.255.0.0
summary-address 10.0.0.0 255.255.0.0
```

## 🖼 Summarization Location

```
ABR → area range
ASBR → summary-address
```

------

# 6️⃣ Route Filtering

```bash
distribute-list prefix BLOCK in
area 1 filter-list prefix BLOCK in
ip prefix-list BLOCK seq 5 deny 10.0.0.0/8
```

## 🖼 Filtering Flow

```
LSA → LSDB → RIB → FIB

Filtering can occur:
- Between areas (ABR)
- Into RIB (distribute-list)
```

------

# 7️⃣ Redistribution (High-Level)

```bash
redistribute static subnets
redistribute connected subnets
redistribute eigrp 100 subnets
redistribute bgp 65000 subnets
default-metric 20
metric-type 1
metric-type 2
```

## 🖼 Redistribution Flow

```
Static → ASBR → Type 5 LSA → OSPF Domain
```

Metric-type 1 = accumulative
Metric-type 2 = external only

------

# 8️⃣ Advanced Stability & Scaling

```bash
max-metric router-lsa
lsa-group-pacing 10
timers pacing flood 5
ip ospf demand-circuit
ip ospf bfd
bfd interval 50 min_rx 50 multiplier 3
ip ospf mtu-ignore
neighbor 2.2.2.2
area 1 virtual-link 2.2.2.2
```

------

## 🖼 Virtual Link

```
Area 1 ---- Area 2
     \      /
      \    /
      Area 0 (via virtual link)
```

Used to repair broken backbone.

------

# 📊 OSPF Architecture Overview

```
              ┌─────────────┐
              │  Interface  │
              │ (Timers)    │
              └──────┬──────┘
                     │
              ┌──────▼──────┐
              │  OSPF Area  │
              │  LSDB       │
              └──────┬──────┘
                     │
              ┌──────▼──────┐
              │ SPF Engine  │
              └──────┬──────┘
                     │
              Routing Table
```

------



------



------

