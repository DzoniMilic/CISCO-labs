# OSPF, Static Routes and CEF Troubleshooting

## 1. OSPF Route Verification

Verify that the expected OSPF routes are installed in the routing table.

```cisco
show ip ospf neighbor
show ip route ospf
show ip route <destination>
```

An OSPF neighbor in the `FULL` state confirms adjacency, but the expected route must still be verified in the routing table.

## 2. Static Route vs. OSPF Route

During troubleshooting, a static route to the remote PE loopback was removed to confirm that OSPF was advertising the route correctly.

Example:

```cisco
no ip route 44.44.44.44 255.255.255.255 10.10.10.2
show ip route 44.44.44.44
```

The route was subsequently learned through OSPF:

```text
Known via "ospf 1", distance 110, metric 4, type intra area
```

This confirmed that the remote PE loopback was reachable through OSPF without relying on the static route.

## 3. MPLS Forwarding Verification

Use the following commands to inspect label forwarding and LDP sessions:

```cisco
show mpls forwarding-table
show mpls ldp neighbor
show mpls ldp discovery
```

- `show mpls forwarding-table` displays local labels, outgoing labels, prefixes, interfaces, and next hops.
- `show mpls ldp neighbor` verifies whether an LDP session is operational.
- `show mpls ldp discovery` displays LDP discovery information on enabled interfaces.

A route to the remote PE loopback is necessary for LDP to establish a session with that PE.

## 4. CEF Verification

CEF can be used to inspect how a destination is resolved for forwarding.

```cisco
show ip cef <destination>
show ip cef <destination> detail
```

Use CEF output together with the routing table and MPLS forwarding table to distinguish an IP routing issue from a label-forwarding issue.

## 5. Troubleshooting Sequence

1. Verify OSPF adjacency.
2. Verify that the expected remote loopback routes are installed.
3. Identify whether a route is learned through OSPF or configured statically.
4. Verify LDP neighbor state.
5. Inspect MPLS label forwarding.
6. Use CEF to inspect destination forwarding resolution.
7. Continue with VRF routing and MP-BGP VPNv4 verification for the L3VPN service.

## Key Lesson

A working OSPF adjacency alone does not prove that the required route is installed. Likewise, an IP route alone does not prove that MPLS label forwarding is operational. Verify each stage independently.
