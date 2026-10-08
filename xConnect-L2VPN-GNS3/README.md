# Cisco xConnect (L2VPN) over MPLS — GNS3 Lab

## Overview

This lab demonstrates a Cisco Layer 2 VPN (L2VPN) using **xConnect over MPLS** in GNS3.

The topology consists of:

* 2 Customer Edge routers (CE1, CE2)
* 2 Provider Edge routers (PE1, PE2)
* 2 Provider routers (P1, P2)
* OSPF as the IGP
* MPLS/LDP as the transport mechanism
* xConnect as the Layer 2 VPN service

The final goal is to provide Layer 2 connectivity between CE1 and CE2 through the MPLS provider network.

---

## Topology

```text
CE1 -------- PE1 -------- P1 -------- P2 -------- PE2 -------- CE2
 |            |           |           |            |            |
Gi0/0       Gi0/2       Gi0/0       Gi0/1        Gi0/2        Gi0/0
 |            |           |           |            |            |
192.168.10.1  |        MPLS CORE                   |       192.168.10.2
              |                                    |
              +========= xConnect / VC 10 =========+
```

---

## Technologies

* Cisco IOS
* GNS3
* OSPF
* MPLS
* LDP
* xConnect
* L2VPN / Pseudowire

---

## Lab Objectives

1. Configure IP addressing between PE and P routers.
2. Establish OSPF adjacency across the provider core.
3. Configure MPLS.
4. Configure LDP.
5. Establish LDP neighbor relationships.
6. Verify MPLS forwarding.
7. Configure an xConnect pseudowire between PE1 and PE2.
8. Connect CE1 and CE2 through the Layer 2 VPN.
9. Verify end-to-end connectivity.

---

## Final Result

CE1 and CE2 are in the same IP subnet:

```text
CE1 = 192.168.10.1/24
CE2 = 192.168.10.2/24
```

They successfully communicate through the MPLS provider network.

Verification:

```text
CE2#ping 192.168.10.1

!!!!!
Success rate is 100 percent (5/5)
```

---

## Important Concept

The interfaces connecting the PE routers to the CE routers do **not** require IP addresses for this xConnect configuration.

Example:

```text
PE1 Gi0/2 = no IP address
PE2 Gi0/2 = no IP address
```

These interfaces are Layer 2 attachment circuits.

The PE routers use their loopback addresses and the MPLS/LDP infrastructure to transport the pseudowire between PE1 and PE2.

---

## Verification Commands

### OSPF

```cisco
show ip ospf neighbor
show ip route
```

### LDP

```cisco
show mpls ldp neighbor
```

### MPLS

```cisco
show mpls forwarding-table
```

### xConnect

```cisco
show xconnect all
```

### End-to-end test

```cisco
CE1#ping 192.168.10.2
CE2#ping 192.168.10.1
```

---

## Expected xConnect State

```text
XC ST  Segment 1                    S1  Segment 2                    S2
UP     ac Gi0/2:4(Ethernet)        UP  mpls PE2:10                   UP
```

The xConnect must be `UP` on both PE routers.

---

## Key Learning

### L3 routing

In a normal routed connection, the PE would need an IP address toward the CE.

### L2VPN / xConnect

With xConnect, the PE does not route the CE's IP packet.

Instead, the PE transports the Ethernet frame across the MPLS pseudowire.

```text
CE1
 |
 | Ethernet frame
 ↓
PE1
 |
 | MPLS pseudowire
 ↓
PE2
 |
 | Ethernet frame
 ↓
CE2
```

The MPLS core provides transport; xConnect provides the Layer 2 service.
