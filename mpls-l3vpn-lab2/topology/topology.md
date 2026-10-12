# MPLS L3VPN Lab Topology

## 1. Overview

This lab demonstrates a basic Multiprotocol Label Switching Layer 3 VPN (MPLS L3VPN) implementation using Cisco IOS routers in GNS3.

The service connects two customer sites through a service provider network. The provider uses OSPF for internal IP reachability, LDP for MPLS label distribution, and MP-BGP VPNv4 to exchange customer VPN routes between Provider Edge (PE) routers.

## 2. Logical Topology

The network consists of six routers arranged in the following order:

```text
CE1 --- PE1 --- P1 --- P2 --- PE2 --- CE2
```

| Device | Role                   | Loopback1      |
| ------ | ---------------------- | -------------- |
| CE1    | Customer Edge — Site A | Not configured |
| PE1    | Provider Edge — Site A | 11.11.11.11/32 |
| P1     | Provider Core          | 1.1.1.1/32     |
| P2     | Provider Core          | 2.2.2.2/32     |
| PE2    | Provider Edge — Site B | 44.44.44.44/32 |
| CE2    | Customer Edge — Site B | Not configured |

## 3. Physical and Logical Links

| Local Device | Local Interface    | Remote Device | Remote Interface   |
| ------------ | ------------------ | ------------- | ------------------ |
| CE1          | GigabitEthernet0/0 | PE1           | GigabitEthernet0/2 |
| PE1          | GigabitEthernet0/1 | P1            | GigabitEthernet0/0 |
| P1           | GigabitEthernet0/1 | P2            | GigabitEthernet0/0 |
| P2           | GigabitEthernet0/1 | PE2           | GigabitEthernet0/1 |
| PE2          | GigabitEthernet0/2 | CE2           | GigabitEthernet0/0 |

## 4. Network Roles

### Customer Edge (CE) routers

CE1 and CE2 represent customer routers at two geographically separate sites. They use ordinary IPv4 addressing and static routes to reach the remote customer subnet through their local PE router.

The CE routers do not participate in the provider's MPLS core.

### Provider Edge (PE) routers

PE1 and PE2 connect the customer sites to the service provider network.

Both PE routers use the VRF `CUSTOMER_A` to maintain customer-specific routing information. They exchange VPNv4 routes through MP-BGP.

### Provider (P) routers

P1 and P2 form the provider's MPLS core. They run OSPF and LDP to provide IP reachability and label-switched transport between the PE routers.

The P routers do not need to maintain the customer's individual VPN routes.

## 5. Control Plane

The lab uses the following protocols:

* **OSPF, process 1, Area 0:** Provides IP reachability across the provider core, including PE loopbacks.
* **LDP:** Distributes MPLS transport labels for reachable provider prefixes.
* **MP-BGP VPNv4:** Exchanges customer VPN routes between PE1 and PE2 using BGP AS 65000.
* **VRF:** Separates customer routing information from the global routing table.

The provider-facing links carry MPLS traffic. The customer-facing interfaces are assigned to the customer VRF on their respective PE routers.

## 6. Customer Addressing

| Site   | Customer Router | Customer Subnet | Default Next Hop for Remote Site |
| ------ | --------------- | --------------- | -------------------------------- |
| Site A | CE1             | 192.168.10.0/24 | 192.168.10.2                     |
| Site B | CE2             | 192.168.20.0/24 | 192.168.20.2                     |

The objective is to enable communication between `192.168.10.0/24` and `192.168.20.0/24` through the MPLS L3VPN service.

## 7. Verification Objective

The topology is considered operational when:

1. OSPF adjacencies and provider routing are operational.
2. LDP sessions are established across the provider core.
3. MP-BGP VPNv4 peering between PE1 and PE2 is established.
4. Both PE routers have the expected remote customer subnet in `CUSTOMER_A`.
5. MPLS forwarding information is correctly resolved.
6. CE1 can ping CE2, and CE2 can ping CE1.

Command outputs and test results are documented separately in the `verification/` directory.

## 8. Lab Environment

* Platform: GNS3
* Network operating system: Cisco IOS
* Routing protocol: OSPF
* MPLS label distribution: LDP
* VPN route exchange: MP-BGP VPNv4
* Customer isolation: VRF

This is a lab environment intended for learning, verification, and troubleshooting. It is not a production network design.

