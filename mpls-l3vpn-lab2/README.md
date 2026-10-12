# Cisco MPLS L3VPN Lab

## Project Overview

This project demonstrates a Multiprotocol Label Switching Layer 3 Virtual Private Network (MPLS L3VPN) built and tested in GNS3 using Cisco IOSv routers.

The lab connects two customer sites through a service provider network. The provider uses OSPF for internal IP reachability, LDP for MPLS label distribution, and MP-BGP VPNv4 to exchange customer VPN routes between Provider Edge (PE) routers.

The objective is to understand how these technologies work together to provide isolated Layer 3 connectivity between customer sites across a shared provider core.

## Lab Objectives

* Build a provider network using OSPF Area 0.
* Enable MPLS forwarding on provider-facing links.
* Establish LDP adjacencies and verify label distribution.
* Configure a customer VRF on both PE routers.
* Establish an MP-BGP VPNv4 peering between the PE routers.
* Exchange customer routes using VPN route targets.
* Verify MPLS transport and VPN label forwarding.
* Test end-to-end connectivity in both directions.
* Troubleshoot a static-route preference issue affecting OSPF route installation and CEF resolution.
* Document configuration, verification commands, results, and lessons learned.

## Topology

```text
CE1 --- PE1 --- P1 --- P2 --- PE2 --- CE2
```

| Router | Role          | Loopback       |
| ------ | ------------- | -------------- |
| CE1    | Customer Edge | Not configured |
| PE1    | Provider Edge | 11.11.11.11/32 |
| P1     | Provider Core | 1.1.1.1/32     |
| P2     | Provider Core | 2.2.2.2/32     |
| PE2    | Provider Edge | 44.44.44.44/32 |
| CE2    | Customer Edge | Not configured |

See [`topology/topology.md`](topology/topology.md) for the topology description and [`addressing/ip-addressing.md`](addressing/ip-addressing.md) for the complete IP addressing plan.

## Technologies Used

| Technology               | Purpose                                                                  |
| ------------------------ | ------------------------------------------------------------------------ |
| OSPF                     | Internal routing and reachability between provider routers               |
| MPLS                     | Label-based packet forwarding across the provider core                   |
| LDP                      | Distribution of transport labels                                         |
| VRF                      | Separation of customer routing information                               |
| MP-BGP VPNv4             | Exchange of VPN routes between PE routers                                |
| Route Distinguisher (RD) | Identification of VPN routes                                             |
| Route Target (RT)        | Control of VPN route import and export                                   |
| CEF                      | Cisco's forwarding mechanism used to inspect packet forwarding decisions |

## VPN Design

The customer VRF is named `CUSTOMER_A`.

| Parameter           | Value      |
| ------------------- | ---------- |
| VRF name            | CUSTOMER_A |
| Route Distinguisher | 65000:10   |
| Route Target import | 65000:10   |
| Route Target export | 65000:10   |
| Provider BGP AS     | 65000      |

Both PE routers use the same route target to import and export the customer's VPN routes.

## Customer Networks

| Site   | Router | Customer IP  | Subnet          |
| ------ | ------ | ------------ | --------------- |
| Site A | CE1    | 192.168.10.1 | 192.168.10.0/24 |
| Site A | PE1    | 192.168.10.2 | 192.168.10.0/24 |
| Site B | PE2    | 192.168.20.2 | 192.168.20.0/24 |
| Site B | CE2    | 192.168.20.1 | 192.168.20.0/24 |

The CE routers use static routes to reach the remote customer subnet through their directly connected PE router. The PE routers exchange customer routes through MP-BGP within the `CUSTOMER_A` VRF context.

## Verification

The lab was verified using actual router command output.

Verified results include:

* OSPF reachability across the provider network.
* OSPF route installation to the remote PE loopback.
* MPLS/LDP operation across the provider core.
* MP-BGP VPNv4 peering between PE1 and PE2.
* Customer routes present in the VRF routing tables.
* MPLS forwarding and CEF resolution for the remote VPN route.
* Successful CE1-to-CE2 connectivity.
* Successful CE2-to-CE1 connectivity.

The end-to-end ping tests completed successfully in both directions (5/5 replies in each direction).

Detailed command output and evidence are stored in the `verification/` directory.

## Troubleshooting Case Study

During verification, PE2 had a static route to PE1's loopback address:

`11.11.11.11/32 via 10.10.10.13`

OSPF had already calculated a route to the same destination, but the static route had a lower administrative distance and was preferred for installation in the global routing table.

The OSPF route was therefore calculated but not installed. The VRF CEF lookup also showed an unresolved transport path (`unusable: no label`).

After the unnecessary static route was removed, the OSPF route was installed. Subsequent checks showed the transport and VPN label resolution in the VRF CEF entry.

The VPN was then verified again using end-to-end connectivity tests.

See [`troubleshooting/ospf-static-route-cef.md`](troubleshooting/ospf-static-route-cef.md) for the diagnostic sequence and evidence.

## Repository Structure

```text
cisco-mpls-l3vpn-lab/
├── README.md
├── topology/
│   ├── topology.md
│   └── topology.png
├── addressing/
│   └── ip-addressing.md
├── configs/
│   ├── CE1.txt
│   ├── PE1.txt
│   ├── P1.txt
│   ├── P2.txt
│   ├── PE2.txt
│   └── CE2.txt
├── verification/
│   ├── 01-ospf.md
│   ├── 02-mpls-ldp.md
│   ├── 03-vpnv4-bgp.md
│   ├── 04-vrf-routing.md
│   ├── 05-mpls-forwarding-cef.md
│   └── 06-end-to-end-ping.md
├── troubleshooting/
│   └── ospf-static-route-cef.md
└── notes/
    └── lessons-learned.md
```

## Lab Environment

* Simulator: GNS3
* Router platform: Cisco IOSv
* Routing protocol: OSPF
* MPLS control plane: LDP
* VPN routing control plane: MP-BGP VPNv4
* Customer routing separation: VRF

Exact software versions and complete device configurations are documented in the corresponding files where available.

## Learning Outcomes

This lab demonstrates how a provider can transport traffic between customer sites using MPLS while maintaining customer-specific routing information.

It also demonstrates a practical troubleshooting workflow: verify the routing information, identify route-selection behavior, inspect MPLS forwarding, examine CEF resolution, and confirm the final result with end-to-end tests.

**Status:** MPLS L3VPN connectivity verified in both directions.
