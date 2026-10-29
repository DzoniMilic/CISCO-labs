# Troubleshooting — OSPF, Routing and CEF

## 1. Troubleshooting Approach

Troubleshooting an MPLS L3VPN should follow a layered approach. Verify the provider underlay first, then MPLS, VPN route exchange, VRF routing and finally end-to-end packet forwarding.

Recommended order:

1. Interface status and IP addressing
2. OSPF neighbor relationships
3. Provider loopback reachability
4. MPLS/LDP neighbor and label forwarding
5. MP-BGP VPNv4 session and route exchange
6. Customer VRF routing tables
7. CEF/MPLS forwarding and end-to-end connectivity

Do not assume that a working control plane means customer traffic is being forwarded successfully.

## 2. Interface and IP Connectivity

### Symptoms
- OSPF neighbors are missing.
- LDP sessions are not established.
- Provider loopbacks are unreachable.

### Commands
```text
show ip interface brief
show running-config
show ip route
ping <destination-ip>
```

### What to check
- Interfaces must be `up/up`.
- IP addresses and subnet masks must match the addressing plan.
- Interfaces must connect to the expected neighboring router.
- The routing table must contain the necessary connected and remote routes.

## 3. OSPF Troubleshooting

### Commands
```text
show ip ospf neighbor
show ip ospf interface brief
show ip protocols
show ip route ospf
```

### What to check
- The expected neighbors should reach the `FULL` state.
- Confirm that the correct interfaces participate in OSPF area 0.
- Verify that provider loopbacks are advertised and reachable.
- Check the OSPF router ID and network statements.

An OSPF adjacency that is not `FULL` can prevent the provider from learning the reachability required by LDP and MP-BGP.

## 4. MPLS and LDP Troubleshooting

### Commands
```text
show mpls ldp neighbor
show mpls ldp parameters
show mpls forwarding-table
show running-config
```

### What to check
- The expected LDP neighbor should be present and operational.
- The LDP router ID should match the intended loopback address.
- MPLS should be enabled on the provider-facing interfaces.
- The forwarding table should contain usable label operations and next hops for relevant destinations.

A working LDP session does not prove that every VPN route has a valid forwarding path.

## 5. MP-BGP VPNv4 Troubleshooting

### Commands
```text
show ip bgp vpnv4 all summary
show ip bgp vpnv4 all
show ip bgp vpnv4 vrf CUSTOMER_A
show ip bgp vpnv4 vrf CUSTOMER_B
```

### What to check
- The PE-to-PE BGP neighbor should be established.
- The VPNv4 address family should be activated for the neighbor.
- Extended communities should be sent.
- Expected customer prefixes should be present in the VPNv4 BGP table.
- Each route should have the expected VRF and Route Target association.

BGP can remain established while customer forwarding is broken. Check the routing and forwarding tables separately.

## 6. VRF Routing Troubleshooting

### Commands
```text
show ip vrf
show ip route vrf CUSTOMER_A
show ip route vrf CUSTOMER_B
show ip route vrf CUSTOMER_B 8.8.8.8
```

### What to check
- The customer-facing interface must belong to the correct VRF.
- The VRF routing table should contain connected and expected remote customer routes.
- Route Targets must allow the intended VPN routes to be imported.
- A route visible in the VPNv4 BGP table is not necessarily installed in the VRF routing table.

**Important:** A ping to an address belonging to a VRF should be tested in the correct VRF context. A ping from the global routing table to a VRF-connected address is not a valid substitute for a customer-path test.

## 7. CEF and MPLS Data-Plane Failure — PE1

### Test objective

Determine whether disabling Cisco Express Forwarding (CEF) affects end-to-end customer traffic even when routing protocols and VPN routes remain present.

### Baseline

Before the test, the following customer-originated ping succeeded:

```text
Customer_B_CE1# ping 8.8.8.8 source 4.4.4.4
```

The ping returned 5 successful replies out of 5.

### Test procedure

On PE1, CEF was disabled in a controlled lab test:

```text
PE1# configure terminal
PE1(config)# no ip cef
PE1(config)# end
```

The following command confirmed that IPv4 CEF was no longer running:

```text
PE1# show ip cef
```

The router reported:

```text
%IPv4 CEF not running
```

The MPLS forwarding table was then inspected:

```text
PE1# show mpls forwarding-table
```

The output contained entries showing `No Label` and `drop` for affected forwarding entries.

The control plane was checked separately:

```text
PE1# show ip ospf neighbor
PE1# show mpls ldp neighbor
PE1# show ip bgp vpnv4 all summary
PE1# show ip bgp vpnv4 all
```

Observations:
- The OSPF neighbor to P1 remained `FULL`.
- The LDP neighbor to P1 remained operational.
- The PE-to-PE VPNv4 BGP session remained established.
- VPN routes were still visible in the BGP table.
- The customer-to-customer ping failed while CEF was disabled.

### Restoration

CEF was re-enabled:

```text
PE1# configure terminal
PE1(config)# ip cef
PE1(config)# end
```

### Conclusion

In this IOSv lab, disabling CEF disrupted MPLS data-plane forwarding while important control-plane relationships and VPN routes remained present.

This demonstrates the difference between:

- **Control plane:** OSPF, LDP and BGP exchange reachability and forwarding information.
- **Data plane:** CEF and MPLS forwarding entries are used to forward actual packets.

### Required post-restoration verification

Repeat the following checks after restoring CEF:

```text
PE1# show ip cef
PE1# show mpls forwarding-table
Customer_B_CE1# ping 8.8.8.8 source 4.4.4.4
```

Confirm that CEF is running, the relevant forwarding entries are usable, and the end-to-end ping succeeds again. Record the actual output before marking recovery as verified.

## 8. General Troubleshooting Checklist

- [ ] Interfaces are `up/up`.
- [ ] IP addresses and masks match the topology.
- [ ] OSPF neighbors are `FULL`.
- [ ] Provider loopbacks are reachable.
- [ ] LDP neighbors are operational.
- [ ] MPLS forwarding entries have usable label operations and next hops.
- [ ] MP-BGP VPNv4 sessions are established.
- [ ] Expected customer routes exist in the VPNv4 BGP table.
- [ ] Expected routes are installed in the correct VRF.
- [ ] CEF is running.
- [ ] End-to-end customer traffic succeeds.
- [ ] Test results and command output are saved in the repository.

