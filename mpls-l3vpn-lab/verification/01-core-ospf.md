# 01 — Core OSPF Verification

## 1. Objective

Verify OSPF operation in the provider core of the MPLS L3VPN lab.

OSPF provides the IP underlay required for:
- Reachability between provider routers.
- Reachability to PE and P router loopbacks.
- Establishment of PE-to-PE MP-BGP sessions using loopback addresses.
- Supporting MPLS/LDP operation across the provider network.

OSPF does **not** exchange customer VPN routes in this lab. Customer VPN routes are exchanged using MP-BGP VPNv4.

## 2. Provider Core Topology

| Router | Loopback0 | Core-facing interfaces |
|---|---|---|
| PE1 | 5.5.5.5/32 | Gi0/2 — 15.15.15.5/24 |
| P1 | 1.1.1.1/32 | Gi0/0 — 15.15.15.1/24; Gi0/1 — 12.12.12.1/24 |
| P2 | 2.2.2.2/32 | Gi0/0 — 12.12.12.2/24; Gi0/1 — 26.26.26.2/24 |
| PE2 | 6.6.6.6/32 | Gi0/0 — 26.26.26.6/24 |

OSPF process: `1`

OSPF area: `0`

## 3. OSPF Configuration

### P1

```text
router ospf 1
 mpls ldp autoconfig
 router-id 1.1.1.1
 network 1.1.1.1 0.0.0.0 area 0
 network 12.12.12.0 0.0.0.255 area 0
 network 15.15.15.0 0.0.0.255 area 0
```

### P2

```text
router ospf 1
 mpls ldp autoconfig
 router-id 2.2.2.2
 network 2.2.2.2 0.0.0.0 area 0
 network 12.12.12.0 0.0.0.255 area 0
 network 26.26.26.0 0.0.0.255 area 0
```

### PE1

```text
interface Loopback0
 ip address 5.5.5.5 255.255.255.255
 ip ospf 1 area 0
!
interface GigabitEthernet0/2
 ip address 15.15.15.5 255.255.255.0
 ip ospf 1 area 0
 mpls ip
!
router ospf 1
 router-id 5.5.5.5
 network 5.5.5.5 0.0.0.0 area 0
 network 15.15.15.0 0.0.0.255 area 0
```

### PE2

```text
router ospf 1
 mpls ldp autoconfig
 router-id 6.6.6.6
 network 6.6.6.6 0.0.0.0 area 0
 network 26.26.26.0 0.0.0.255 area 0
```

**Note:** These snippets document the relevant OSPF configuration, not the complete router configurations. PE1 explicitly enables MPLS on its core-facing interface with `mpls ip`.

## 4. Verification Commands

Run these commands on each provider router.

### 4.1 Check OSPF neighbors

```text
show ip ospf neighbor
```

Expected neighboring router IDs:

| Router | Expected OSPF neighbors |
|---|---|
| PE1 | P1 — `1.1.1.1` |
| P1 | PE1 — `5.5.5.5`; P2 — `2.2.2.2` |
| P2 | P1 — `1.1.1.1`; PE2 — `6.6.6.6` |
| PE2 | P2 — `2.2.2.2` |

**What to check:**
- Expected neighbors are present.
- Neighbor state reaches `FULL`.
- The interface and neighbor IP address match the topology.

On an Ethernet segment, the full state may display a role such as `FULL/DR` or `FULL/BDR`. The role itself is not an error.

### 4.2 Check OSPF-learned routes

```text
show ip route ospf
```

Check that remote provider loopbacks and core subnets are reachable through the expected next hops, where applicable.

A router does not learn its own loopback as an OSPF route from a neighbor; its local loopback normally appears as a local or connected route.

### 4.3 Check interface addressing and status

```text
show ip interface brief
```

Confirm that the relevant core interfaces are `up/up` and have the planned IP addresses.

### 4.4 Check the OSPF process

```text
show ip protocols
show running-config | section router ospf
```

Confirm:
- The correct OSPF process is active.
- The router ID is correct.
- The intended networks participate in area 0.
- The configuration matches the topology.

## 5. Verify Provider Loopback Reachability

Test connectivity between provider loopbacks.

Examples:

From P1 to P2:

```text
ping 2.2.2.2 source 1.1.1.1
```

From PE1 to PE2:

```text
ping 6.6.6.6 source 5.5.5.5
```

Expected result: successful replies.

If a test fails, check OSPF adjacency, the routing table and the return path before investigating MP-BGP.

## 6. Troubleshooting

### No OSPF neighbor

Check:
1. Interface status and IP addressing.
2. OSPF network statements or interface-level OSPF configuration.
3. Matching area configuration.
4. OSPF process and router ID.
5. Relevant interface and OSPF logs.

Commands:

```text
show ip interface brief
show ip ospf neighbor
show ip ospf interface brief
show running-config | section router ospf
```

### Neighbor does not reach FULL

Inspect the neighbor state and OSPF interface details. Check for mismatched area settings, timers, authentication or network type where applicable.

### OSPF is FULL but a loopback is unreachable

Check whether the destination loopback is advertised and present in the routing table:

```text
show ip route <destination-loopback>
show ip ospf database
ping <destination-loopback>
```

An established adjacency does not guarantee that every expected route is being advertised or installed.

## 7. Completion Criteria

- [ ] All expected provider-core OSPF neighbors reach `FULL`.
- [ ] Provider loopbacks are reachable.
- [ ] Required provider routes are present in the routing tables.
- [ ] Core interfaces have the correct IP addresses and are operational.
- [ ] OSPF configuration matches the intended topology.

## 8. Key Concept

OSPF is the **underlay routing protocol** in this lab. It provides IP reachability between provider routers and PE loopbacks. MPLS/LDP uses this infrastructure to support label-switched forwarding, while MP-BGP VPNv4 distributes customer VPN routes between PE routers.

A successful OSPF check is an important first step, but it does not independently prove that the MPLS L3VPN service is working end to end.

