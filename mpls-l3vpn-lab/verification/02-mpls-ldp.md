# 02 — MPLS and LDP Verification

## 1. Objective

Verify MPLS and Label Distribution Protocol (LDP) operation across the provider core.

MPLS forwards packets using labels rather than relying only on an IP destination lookup at every hop. LDP distributes labels between MPLS-enabled routers for eligible IP forwarding destinations.

In this lab:
- OSPF provides provider-core IP reachability.
- LDP establishes label-distribution sessions between neighboring routers.
- MPLS forwarding entries determine the outgoing label operation and next hop.
- PE routers use MPLS to transport customer VPN traffic across the provider core.

## 2. MPLS Core Topology

| Router | Loopback0 | Core-facing interfaces |
|---|---|---|
| PE1 | 5.5.5.5/32 | Gi0/2 — 15.15.15.5/24 |
| P1 | 1.1.1.1/32 | Gi0/0 — 15.15.15.1/24; Gi0/1 — 12.12.12.1/24 |
| P2 | 2.2.2.2/32 | Gi0/0 — 12.12.12.2/24; Gi0/1 — 26.26.26.2/24 |
| PE2 | 6.6.6.6/32 | Gi0/0 — 26.26.26.6/24 |

LDP router IDs:
- PE1: `5.5.5.5`
- P1: `1.1.1.1`
- P2: `2.2.2.2`
- PE2: `6.6.6.6`

## 3. MPLS and LDP Configuration

### 3.1 Enable MPLS on core-facing interfaces

Example on P1:

```text
interface GigabitEthernet0/0
 ip address 15.15.15.1 255.255.255.0
 mpls ip
!
interface GigabitEthernet0/1
 ip address 12.12.12.1 255.255.255.0
 mpls ip
```

Example on P2:

```text
interface GigabitEthernet0/0
 ip address 12.12.12.2 255.255.255.0
 mpls ip
!
interface GigabitEthernet0/1
 ip address 26.26.26.2 255.255.255.0
 mpls ip
```

PE1 and PE2 must also have MPLS enabled on their provider-facing interfaces.

### 3.2 Configure the LDP router ID

Example on P1:

```text
mpls label protocol ldp
mpls ldp router-id Loopback0
```

P2, PE1 and PE2 use the same commands, with each router's own Loopback0.

The loopback provides a stable LDP identifier instead of relying on a physical interface address.

### 3.3 OSPF LDP autoconfiguration

P1 and P2 have the following command under OSPF:

```text
router ospf 1
 mpls ldp autoconfig
```

This enables LDP on eligible OSPF-enabled interfaces on supported IOS platforms. The actual interfaces and sessions must still be verified.

PE1 explicitly enables MPLS using `mpls ip` on its provider-facing interface. Always check the running configuration rather than assuming that all routers use identical configuration methods.

## 4. Verification Commands

### 4.1 Check LDP neighbors

```text
show mpls ldp neighbor
```

**What to check:**
- The expected adjacent router is listed.
- The session state is `Oper`.
- The local and peer LDP identifiers match the intended router IDs.
- The discovery interface and peer address match the topology.

Expected direct LDP adjacencies:

| Link | Expected LDP peers |
|---|---|
| PE1 — P1 | `5.5.5.5` and `1.1.1.1` |
| P1 — P2 | `1.1.1.1` and `2.2.2.2` |
| P2 — PE2 | `2.2.2.2` and `6.6.6.6` |

LDP sessions are established between directly connected MPLS neighbors; PE1 and PE2 do not need to be direct LDP neighbors.

### 4.2 Check LDP parameters

```text
show mpls ldp parameters
```

Inspect:
- LDP identifier and operational state.
- Label distribution protocol.
- Hello and keepalive timers.
- Configured or negotiated LDP features.

The output varies by IOS version and platform.

### 4.3 Check the MPLS forwarding table

```text
show mpls forwarding-table
```

The forwarding table displays the local label, outgoing label operation, destination prefix, outgoing interface and next hop.

Common outgoing label operations include:
- A numeric label: forward using that label.
- `Pop Label`: remove the top label before forwarding.
- `No Label` or `drop`: investigate the entry and platform state before concluding that forwarding is valid.

An entry showing `Pop Label` can be normal. It is often used for penultimate-hop popping, where the upstream router removes the label before sending the packet to the egress router.

### 4.4 Verify IP reachability between provider loopbacks

Example from P1 to P2:

```text
ping 2.2.2.2 source 1.1.1.1
```

Expected result: successful replies.

This verifies IP reachability between the loopbacks. It does not, by itself, prove that customer VPN traffic is forwarding correctly.

## 5. Troubleshooting

### LDP neighbor is missing

Check the following:
1. The physical interface is `up/up`.
2. The IP addressing is correct.
3. OSPF provides reachability to the peer's LDP router ID.
4. MPLS is enabled on the correct provider-facing interface.
5. LDP is configured and the intended router ID is active.

Commands:

```text
show ip interface brief
show ip ospf neighbor
show ip route
show mpls ldp neighbor
show running-config
```

### LDP neighbor is operational but forwarding fails

Check:
1. The destination route exists in the IP routing table.
2. The relevant MPLS forwarding entry exists.
3. The outgoing interface and next hop are correct.
4. The outgoing label operation is appropriate for the destination.
5. CEF is enabled and operating as expected.

Commands:

```text
show ip route <destination>
show mpls forwarding-table
show ip cef
show mpls ldp neighbor
```

A healthy LDP session does not guarantee that every required destination has a usable forwarding entry.

## 6. CEF and MPLS Forwarding

CEF (Cisco Express Forwarding) is used by this IOSv platform to support packet forwarding, including MPLS forwarding.

The lab's controlled test on PE1 demonstrated that:
- Disabling CEF caused `show ip cef` to report that IPv4 CEF was not running.
- The MPLS forwarding table displayed `No Label` and `drop` entries for affected destinations.
- OSPF, LDP and MP-BGP sessions remained present.
- Customer-to-customer traffic failed during the test.

CEF was then re-enabled with:

```text
configure terminal
ip cef
end
```

The final lab configuration must retain CEF enabled where required. After restoring it, verify the forwarding table and repeat the end-to-end customer ping before recording recovery as confirmed.

## 7. Completion Criteria

- [ ] LDP neighbors are present and operational on all provider-core links.
- [ ] LDP router IDs match the intended loopbacks.
- [ ] MPLS is enabled on all intended provider-facing interfaces.
- [ ] The MPLS forwarding table contains appropriate entries for relevant provider destinations.
- [ ] Provider loopback reachability works.
- [ ] CEF is enabled in the final configuration.
- [ ] End-to-end customer connectivity is verified separately.

## 8. Key Concept

OSPF and LDP perform different roles:

- **OSPF** provides IP reachability through the provider network.
- **LDP** distributes labels for eligible forwarding destinations.
- **MPLS forwarding** uses label operations to transport packets through the provider core.
- **CEF** supports the actual forwarding process on this IOSv platform.

All of these layers must work together. An established LDP session is necessary for this lab's LDP-based MPLS design, but it is not sufficient proof of end-to-end VPN connectivity.

