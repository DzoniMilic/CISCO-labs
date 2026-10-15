# 06 — End-to-End Ping Verification

## Objective

Verify end-to-end IP connectivity between CE1 and CE2 across the MPLS L3VPN provider network.

## Verification Steps

### 1. Verify the CE Interfaces

On CE1 and CE2, verify interface status and IP addressing.

```cisco
show ip interface brief
```

### 2. Verify the Customer Routing Table

Check whether each CE has a route to the remote customer subnet.

```cisco
show ip route
```

### 3. Test CE1-to-CE2 Connectivity

From CE1, ping the IP address of CE2:

```cisco
ping <CE2-IP-address>
```

### 4. Test CE2-to-CE1 Connectivity

From CE2, ping the IP address of CE1:

```cisco
ping <CE1-IP-address>
```

## Expected Result

Both ping tests should succeed when:

- The CE interfaces are up and correctly addressed.
- The customer routes are exchanged and installed correctly.
- The PE routers have the required VRF and MP-BGP VPNv4 configuration.
- The provider core has working OSPF and MPLS/LDP forwarding.

## Troubleshooting

If a ping fails, verify the layers in this order:

1. CE interface status and IP addressing.
2. CE routing table.
3. Customer routes in the PE VRF.
4. MP-BGP VPNv4 route exchange.
5. OSPF reachability between provider loopbacks.
6. LDP neighbor state and MPLS forwarding entries.

## Conclusion

Successful bidirectional CE-to-CE pings provide end-to-end confirmation that customer traffic can cross the MPLS L3VPN. A successful ping to a provider loopback alone does not prove that the customer VPN is working.

