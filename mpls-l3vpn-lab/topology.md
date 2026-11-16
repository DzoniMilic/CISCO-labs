# MPLS L3VPN Lab — Network Topology

## 1. Overview

This lab demonstrates a service provider MPLS Layer 3 VPN architecture using Cisco IOS routers in GNS3.

The topology contains four provider routers and four customer edge routers. The provider network uses OSPF for internal reachability, LDP for MPLS label distribution, and MP-BGP VPNv4 to exchange customer VPN routes between the Provider Edge (PE) routers.

Two separate customers share the same provider core:

- **Customer A** — VRF `CUSTOMER_A`, customer AS 200.
- **Customer B** — VRF `CUSTOMER_B`, customer AS 300.

Both customers use overlapping addressing from `10.0.0.0/24` on their PE-CE links. VRF separation keeps their routing information independent.

## 2. Network Diagram

```text
                  MPLS PROVIDER CORE
          OSPF Area 0 / LDP / MP-BGP AS 65000

 CE1-A                  PE1                 P1                 P2                 PE2                  CE2-A
AS 200              Lo0: 5.5.5.5        Lo0: 1.1.1.1      Lo0: 2.2.2.2       Lo0: 6.6.6.6             AS 200
Lo0 3.3.3.3         Gi0/2               Gi0/0              Gi0/1              Gi0/0                  Lo0 7.7.7.7
   |                    |                   |                  |                  |                      |
   | 10.0.0.0/24         | 15.15.15.0/24     | 12.12.12.0/24    | 26.26.26.0/24    | 10.0.0.0/24          |
   +--------------------+-------------------+------------------+------------------+----------------------+
                        |                                                           |
                        |                                                           |
                      CE1-B                                                       CE2-B
                     AS 300                                                      AS 300
                   Lo0 4.4.4.4                                                Lo0 8.8.8.8
```

The diagram is a logical overview. Use the interface mapping and addressing tables below as the authoritative reference for the configured topology.

## 3. Provider Core Links

| Connection | Interface A | IP address A | Interface B | IP address B | Network |
|---|---|---|---|---|---|
| PE1 — P1 | PE1 Gi0/2 | 15.15.15.5/24 | P1 Gi0/0 | 15.15.15.1/24 | 15.15.15.0/24 |
| P1 — P2 | P1 Gi0/1 | 12.12.12.1/24 | P2 Gi0/0 | 12.12.12.2/24 | 12.12.12.0/24 |
| P2 — PE2 | P2 Gi0/1 | 26.26.26.2/24 | PE2 Gi0/0 | 26.26.26.6/24 | 26.26.26.0/24 |

Provider loopbacks:

| Router | Loopback0 | Purpose |
|---|---|---|
| P1 | 1.1.1.1/32 | OSPF router ID / LDP router ID |
| P2 | 2.2.2.2/32 | OSPF router ID / LDP router ID |
| PE1 | 5.5.5.5/32 | Provider loopback / MP-BGP endpoint |
| PE2 | 6.6.6.6/32 | Provider loopback / MP-BGP endpoint |

The provider IGP must provide reachability between the PE loopbacks so that the PE-to-PE MP-BGP session can operate.

## 4. Customer A Connections

Customer A uses VRF `CUSTOMER_A` and AS 200 on both CE routers.

| Connection | Interface | IP address | Network |
|---|---|---|---|
| CE1-A — PE1 | CE1-A customer-facing interface | 10.0.0.3/24 | 10.0.0.0/24 |
| PE1 — CE1-A | PE1 Gi0/0 | 10.0.0.5/24 | 10.0.0.0/24 |
| CE2-A — PE2 | CE2-A customer-facing interface | 10.0.0.7/24 | 10.0.0.0/24 |
| PE2 — CE2-A | PE2 Gi0/1 | 10.0.0.4/24 | 10.0.0.0/24 |

Customer A endpoint loopbacks:

| Router | Loopback0 |
|---|---|
| CE1-A | 3.3.3.3/32 |
| CE2-A | 7.7.7.7/32 |

## 5. Customer B Connections

Customer B uses VRF `CUSTOMER_B` and AS 300 on both CE routers.

| Connection | Interface | IP address | Network |
|---|---|---|---|
| CE1-B — PE1 | CE1-B customer-facing interface | 10.0.0.4/24 | 10.0.0.0/24 |
| PE1 — CE1-B | PE1 Gi0/1 | 10.0.0.55/24 | 10.0.0.0/24 |
| CE2-B — PE2 | CE2-B customer-facing interface | 10.0.0.8/24 | 10.0.0.0/24 |
| PE2 — CE2-B | PE2 Gi0/2 | 10.0.0.6/24 | 10.0.0.0/24 |

Customer B endpoint loopbacks:

| Router | Loopback0 |
|---|---|
| CE1-B | 4.4.4.4/32 |
| CE2-B | 8.8.8.8/32 |

**Important:** Customer A and Customer B reuse the `10.0.0.0/24` subnet on separate links. This is intentional for the VRF-isolation exercise. The overlapping prefixes are valid because the interfaces belong to different VRFs.

## 6. Routing and MPLS Design

### Provider core

- **OSPF Area 0:** distributes provider infrastructure reachability.
- **LDP:** distributes labels for provider transport.
- **MPLS:** transports packets across the provider core.
- **CEF:** provides the IP forwarding mechanism and should remain enabled in the final configuration.

### PE-to-PE VPN routing

- Provider BGP AS: `65000`.
- MP-BGP VPNv4 peers: PE1 `5.5.5.5` and PE2 `6.6.6.6`.
- VPNv4 extended communities carry Route Target information.

### Customer routing

- Customer A: eBGP between each CE and its directly connected PE, using customer AS 200.
- Customer B: eBGP between each CE and its directly connected PE, using customer AS 300.
- The PE configurations use `local-as 100` and `as-override` for the customer-facing BGP neighbors.

## 7. VRF and Route Target Design

| VRF | Route Distinguisher (RD) | Import Route Target | Export Route Target |
|---|---|---|---|
| CUSTOMER_A | 65000:10 | 65000:10 | 65000:10 |
| CUSTOMER_B | 65000:20 | 65000:20 | 65000:20 |

The RD makes VPN prefixes unique in the VPNv4 address family. Route Targets control which VPN routes are imported into each VRF.

The two customers remain logically separated even though their PE-CE links use overlapping IP addressing.

## 8. Expected End-to-End Tests

| Customer | Source | Destination | Test |
|---|---|---|---|
| Customer A | 3.3.3.3 | 7.7.7.7 | `ping 7.7.7.7 source 3.3.3.3` |
| Customer A | 7.7.7.7 | 3.3.3.3 | `ping 3.3.3.3 source 7.7.7.7` |
| Customer B | 4.4.4.4 | 8.8.8.8 | `ping 8.8.8.8 source 4.4.4.4` |
| Customer B | 8.8.8.8 | 4.4.4.4 | `ping 4.4.4.4 source 8.8.8.8` |

These are the intended final tests. Record actual results in the verification documents rather than treating the expected outcomes as already proven.

## 9. Learning Objectives

This lab demonstrates how a service provider can transport multiple isolated customer networks across one MPLS core.

The main learning objectives are:

1. Establish provider reachability with OSPF.
2. Distribute MPLS labels using LDP.
3. Configure customer VRFs and Route Targets.
4. Exchange VPN routes using MP-BGP VPNv4.
5. Connect CE routers to PEs using eBGP.
6. Verify forwarding through CEF and MPLS.
7. Confirm bidirectional connectivity and customer isolation.

## 10. Related Documentation

- `../addressing/ip-addressing.md` — complete IP addressing plan.
- `../configs/` — router configuration files.
- `../verification/01-core-ospf.md` — OSPF verification.
- `../verification/02-mpls-ldp.md` — MPLS and LDP verification.
- `../verification/03-vrf.md` — VRF verification.
- `../verification/04-mp-bgp.md` — MP-BGP VPNv4 verification.
- `../verification/05-customer-a.md` — Customer A verification.
- `../verification/06-customer-b.md` — Customer B verification.
- `../verification/07-final-verification.md` — final end-to-end checklist.
- `../troubleshooting/ospf-static-route-cef.md` — troubleshooting notes.

