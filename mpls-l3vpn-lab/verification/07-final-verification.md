# 07 — Final MPLS L3VPN Verification

## 1. Objective

The objective of this document is to perform a final end-to-end verification of the MPLS Layer 3 VPN lab.

The verification covers:

- Provider core OSPF connectivity.
- MPLS and LDP operation.
- VRF configuration and route isolation.
- MP-BGP VPNv4 route exchange between PE routers.
- PE-CE eBGP connectivity.
- Customer route installation.
- CEF and MPLS packet forwarding.
- End-to-end connectivity between customer sites.

The final configuration keeps Cisco Express Forwarding (CEF) enabled.

## 2. Verification Order

Troubleshooting should follow the packet forwarding architecture, from the provider core toward the customer edge.

| Step | Layer / feature | Main verification |
|---|---|---|
| 1 | Interface connectivity | Interfaces and IP addressing |
| 2 | OSPF | Provider routing adjacency and reachability |
| 3 | LDP | Label distribution and LDP neighbors |
| 4 | MPLS forwarding | Label forwarding entries |
| 5 | VRF | Customer routing-table separation |
| 6 | MP-BGP VPNv4 | Exchange of VPN routes between PEs |
| 7 | PE-CE eBGP | Customer routing sessions |
| 8 | CEF | IP forwarding and adjacency resolution |
| 9 | End-to-end | Customer loopback reachability |

A successful control-plane session does not, by itself, prove that customer packets are being forwarded successfully.

## 3. Interface and IP Verification

Run on the relevant routers:

```text
show ip interface brief
show running-config
```

Confirm that:
- Required interfaces are up/up.
- IP addresses match the addressing plan.
- Provider-facing interfaces are configured for the core routing and MPLS design.
- Customer-facing PE interfaces belong to the correct VRF.
- CEF is enabled in the final configuration.

On PE1 and PE2, verify:

```text
show running-config | include ip cef
```

Expected configuration:

```text
ip cef
```

Do not use a global routing-table test as a substitute for checking a customer route inside its VRF.

## 4. Provider Core — OSPF

Run on PE1, P1, P2 and PE2:

```text
show ip ospf neighbor
show ip route ospf
show ip ospf interface brief
```

Verify that the expected OSPF adjacencies are established and that the provider loopbacks and core networks are reachable through the IGP.

Provider loopbacks:

| Router | Loopback0 |
|---|---|
| P1 | 1.1.1.1/32 |
| P2 | 2.2.2.2/32 |
| PE1 | 5.5.5.5/32 |
| PE2 | 6.6.6.6/32 |

Test provider loopback reachability from the relevant routers:

```text
ping 1.1.1.1
ping 2.2.2.2
ping 5.5.5.5
ping 6.6.6.6
```

Run only the tests relevant to each router's role and record the actual results.

## 5. MPLS and LDP Verification

Run on the provider core routers and PE routers:

```text
show mpls ldp neighbor
show mpls ldp parameters
show mpls forwarding-table
```

Verify that:
- Expected LDP neighbors are operational.
- LDP router IDs match the intended loopbacks.
- Labels are allocated and exchanged for relevant provider prefixes.
- MPLS forwarding entries point to the expected outgoing interface and next hop.

On PE1 and PE2, confirm that MPLS is enabled on the provider-facing interface.

Example configuration check:

```text
show running-config interface GigabitEthernet0/2
```

Use the appropriate interface on each router; PE2's provider-facing interface is GigabitEthernet0/0.

An LDP neighbor being operational proves label-distribution connectivity, but it does not alone prove that a customer VPN packet can traverse the entire path.

## 6. VRF Verification

Run on PE1 and PE2:

```text
show ip vrf
show ip route vrf CUSTOMER_A
show ip route vrf CUSTOMER_B
```

Expected VRF definitions:

| VRF | RD | Import RT | Export RT |
|---|---|---|---|
| CUSTOMER_A | 65000:10 | 65000:10 | 65000:10 |
| CUSTOMER_B | 65000:20 | 65000:20 | 65000:20 |

Verify that:
- Customer A interfaces and routes are associated with `CUSTOMER_A`.
- Customer B interfaces and routes are associated with `CUSTOMER_B`.
- Remote customer prefixes are installed in the intended VRF.
- Overlapping customer addressing does not cause routes to appear in the wrong VRF.

The presence of a VRF definition alone does not confirm that its customer routes have been learned successfully.

## 7. MP-BGP VPNv4 Verification

Run on PE1 and PE2:

```text
show ip bgp vpnv4 all summary
show ip bgp vpnv4 all
```

Verify that the PE-to-PE iBGP session is established between:
- PE1: `5.5.5.5`
- PE2: `6.6.6.6`

The VPNv4 address family must be active, and extended communities must be exchanged so that VPN Route Targets can be carried with the routes.

Expected provider BGP AS:

```text
65000
```

Confirm that the VPNv4 table contains the expected customer prefixes for both VRFs. Check the actual route attributes and next hops in the live output.

## 8. PE-CE eBGP Verification

### Customer A

On PE1 and PE2:

```text
show ip bgp vpnv4 all summary
show ip route vrf CUSTOMER_A
```

Expected CE loopbacks:
- CE1: `3.3.3.3/32`
- CE2: `7.7.7.7/32`

### Customer B

On PE1 and PE2:

```text
show ip route vrf CUSTOMER_B
```

Expected CE loopbacks:
- CE1: `4.4.4.4/32`
- CE2: `8.8.8.8/32`

Confirm the customer BGP neighbors are established and the expected prefixes are learned. If a route is missing, inspect the CE advertisement and the corresponding VRF BGP address family before troubleshooting the MPLS data plane.

## 9. CEF and MPLS Forwarding

CEF must remain enabled in the final configuration.

On PE1 and PE2:

```text
show ip cef
show ip cef vrf CUSTOMER_A
show ip cef vrf CUSTOMER_B
show mpls forwarding-table
```

Check whether the relevant destination prefixes have usable forwarding entries and whether the MPLS forwarding table has the expected label operations and next hops.

When investigating a failed customer ping, inspect the route in the correct VRF:

```text
show ip route vrf CUSTOMER_B 8.8.8.8
```

Use the corresponding command for other destinations and VRFs.

A route in the BGP table does not necessarily mean that the forwarding plane is working. Verify route installation and forwarding entries as well.

## 10. End-to-End Customer Tests

### Customer A

From Customer A CE1:

```text
ping 7.7.7.7 source 3.3.3.3
```

Reverse direction, from Customer A CE2:

```text
ping 3.3.3.3 source 7.7.7.7
```

### Customer B

From Customer B CE1:

```text
ping 8.8.8.8 source 4.4.4.4
```

Reverse direction, from Customer B CE2:

```text
ping 4.4.4.4 source 8.8.8.8
```

For each test, record:
- Source and destination.
- Actual packet-loss result.
- Whether both directions succeed.
- Any relevant troubleshooting observations.

A test is verified only after its real output has been captured.

## 11. Troubleshooting Decision Path

If an end-to-end customer ping fails, use this order:

1. **Interface and addressing:** Are the relevant interfaces up, and are the IP addresses correct?
2. **OSPF:** Can the provider routers reach one another's loopbacks?
3. **LDP/MPLS:** Are the expected label-distribution neighbors and forwarding entries present?
4. **PE-CE BGP:** Are customer routes being learned from the CE routers?
5. **MP-BGP VPNv4:** Is the remote customer route exchanged between PE routers?
6. **VRF routing:** Is the destination route installed in the correct VRF?
7. **CEF/MPLS forwarding:** Can the forwarding plane resolve the route and next hop?
8. **Return path:** Does the destination site have a route back to the source?

Troubleshoot the first layer that fails rather than changing multiple configuration sections at once.

## 12. Final Completion Checklist

- [ ] Required interfaces are up/up.
- [ ] Provider OSPF adjacencies and loopback reachability are verified.
- [ ] Expected LDP neighbors are operational.
- [ ] MPLS forwarding entries are present and usable.
- [ ] Both customer VRFs are configured with the intended RD and Route Targets.
- [ ] PE-to-PE VPNv4 BGP is established.
- [ ] Customer A PE-CE routing and routes are verified.
- [ ] Customer B PE-CE routing and routes are verified.
- [ ] CEF is enabled in the final configuration.
- [ ] Customer A end-to-end tests succeed in both directions.
- [ ] Customer B end-to-end tests succeed in both directions.
- [ ] Customer routing information remains isolated between VRFs.
- [ ] Actual verification outputs are saved in the repository.

## 13. Final Result

The lab can be considered fully verified only when the control plane, routing tables and forwarding plane operate as intended and both customers have successful bidirectional end-to-end connectivity.

Save the relevant command outputs alongside the existing verification documents. Do not mark a test as successful based solely on the presence of a configuration command or an established BGP session.

