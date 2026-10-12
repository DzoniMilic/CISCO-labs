# MPLS L3VPN Lab — IP Addressing Plan

## 1. Overview

This document describes the IPv4 addressing scheme used in the MPLS L3VPN lab.

The addressing plan separates customer-facing networks from provider infrastructure networks. Provider point-to-point links use `/30` subnets, while provider loopbacks use `/32` addresses.

## 2. Customer-Facing Interfaces

| Device | Interface          | IPv4 Address    | Subnet          | Purpose              |
| ------ | ------------------ | --------------- | --------------- | -------------------- |
| CE1    | GigabitEthernet0/0 | 192.168.10.1/24 | 192.168.10.0/24 | Site A customer LAN  |
| PE1    | GigabitEthernet0/2 | 192.168.10.2/24 | 192.168.10.0/24 | Site A PE-to-CE link |
| PE2    | GigabitEthernet0/2 | 192.168.20.2/24 | 192.168.20.0/24 | Site B PE-to-CE link |
| CE2    | GigabitEthernet0/0 | 192.168.20.1/24 | 192.168.20.0/24 | Site B customer LAN  |

Both PE customer-facing interfaces belong to the VRF `CUSTOMER_A`.

## 3. Provider Point-to-Point Links

| Link   | Local Interface | Local Address  | Remote Interface | Remote Address | Subnet         |
| ------ | --------------- | -------------- | ---------------- | -------------- | -------------- |
| PE1–P1 | PE1 Gi0/1       | 10.10.10.1/30  | P1 Gi0/0         | 10.10.10.2/30  | 10.10.10.0/30  |
| P1–P2  | P1 Gi0/1        | 10.10.10.9/30  | P2 Gi0/0         | 10.10.10.10/30 | 10.10.10.8/30  |
| P2–PE2 | P2 Gi0/1        | 10.10.10.13/30 | PE2 Gi0/1        | 10.10.10.14/30 | 10.10.10.12/30 |

These links form the provider's routed backbone and participate in OSPF Area 0.

MPLS is enabled on the provider-facing interfaces.

## 4. Provider Loopbacks

| Device | Interface | IPv4 Address   | Purpose                               |
| ------ | --------- | -------------- | ------------------------------------- |
| P1     | Loopback1 | 1.1.1.1/32     | Provider router identification        |
| P2     | Loopback1 | 2.2.2.2/32     | Provider router identification        |
| PE1    | Loopback1 | 11.11.11.11/32 | PE identification and MP-BGP endpoint |
| PE2    | Loopback1 | 44.44.44.44/32 | PE identification and MP-BGP endpoint |

The PE loopbacks are used as the source and destination addresses for the MP-BGP session. They must be reachable through the provider network.

The lab configures LDP to use Loopback1 as its router ID on the relevant provider routers.

## 5. Customer Routing

The customer routers use static routes to reach the remote site's subnet.

### CE1

```cisco
ip route 192.168.20.0 255.255.255.0 192.168.10.2
```

This route sends traffic destined for Site B to PE1's customer-facing address.

### CE2

```cisco
ip route 192.168.10.0 255.255.255.0 192.168.20.2
```

This route sends traffic destined for Site A to PE2's customer-facing address.

## 6. VRF and VPN Route Identification

Both PE routers use the following VRF parameters:

| Parameter                | Value      |
| ------------------------ | ---------- |
| VRF name                 | CUSTOMER_A |
| BGP autonomous system    | 65000      |
| Route Distinguisher (RD) | 65000:10   |
| Route Target import      | 65000:10   |
| Route Target export      | 65000:10   |

The RD makes VPN routes unique in the VPNv4 address family. The matching import and export Route Targets allow the customer routes to be exchanged between PE1 and PE2 for this VPN.

## 7. Addressing Summary

| Network                 | Type                    | Prefix          |
| ----------------------- | ----------------------- | --------------- |
| Site A customer network | Customer                | 192.168.10.0/24 |
| Site B customer network | Customer                | 192.168.20.0/24 |
| PE1–P1                  | Provider point-to-point | 10.10.10.0/30   |
| P1–P2                   | Provider point-to-point | 10.10.10.8/30   |
| P2–PE2                  | Provider point-to-point | 10.10.10.12/30  |
| P1 loopback             | Provider loopback       | 1.1.1.1/32      |
| P2 loopback             | Provider loopback       | 2.2.2.2/32      |
| PE1 loopback            | Provider loopback       | 11.11.11.11/32  |
| PE2 loopback            | Provider loopback       | 44.44.44.44/32  |

## 8. Design Notes

* Customer subnets are kept separate from the provider infrastructure addressing.
* `/30` subnets provide two usable IPv4 addresses for each point-to-point link.
* `/32` loopbacks provide stable router identifiers and MP-BGP endpoints.
* The customer VRF maintains a separate routing table on each PE.
* Customer routes are exchanged through MP-BGP VPNv4 and installed in the appropriate VRF according to Route Target policy.

