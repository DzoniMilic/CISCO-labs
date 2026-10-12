# VRF Routing Verification

## Objective

Verify that PE1 has installed the local and remote customer routes in VRF `CUSTOMER_A` and that the remote customer prefix has been learned through MP-BGP.

## Command 1: Display the VRF Routing Table

```text
PE1# show ip route vrf CUSTOMER_A
```

### Observed Results

```text
C 192.168.10.0/24 is directly connected, GigabitEthernet0/2
L 192.168.10.2/32 is directly connected, GigabitEthernet0/2
B 192.168.20.0/24 [200/0] via 44.44.44.44
```

### Analysis

* `C` — The local customer LAN, `192.168.10.0/24`, is directly connected to PE1.
* `L` — `192.168.10.2/32` is the local IP address assigned to PE1's customer-facing interface.
* `B` — The remote customer LAN, `192.168.20.0/24`, has been learned through BGP.
* `44.44.44.44` — The BGP next hop is PE2's loopback address.
* `[200/0]` — The route has an administrative distance of 200 and a BGP metric of 0.

The remote customer prefix is present in the VRF routing table and is available for forwarding.

## Command 2: Verify VPNv4 BGP Routes

```text
PE1# show ip bgp vpnv4 vrf CUSTOMER_A
```

### Observed Results

```text
Route Distinguisher: 65000:10 (default for vrf CUSTOMER_A)
 *>  192.168.10.0     0.0.0.0                  0         32768 ?
 *>i 192.168.20.0     44.44.44.44              0    100      0 ?
```

### Analysis

* `65000:10` — The Route Distinguisher associated with this VRF's VPNv4 routes.
* `192.168.10.0` — PE1's local customer prefix.
* `192.168.20.0` — The remote customer prefix advertised by PE2.
* `44.44.44.44` — The BGP next hop for the remote prefix.
* `*` — The route is valid.
* `>` — The route is the selected best path.
* `i` in the status column — The route was learned through internal BGP (iBGP).

## Conclusion

PE1 has successfully learned the remote customer network `192.168.20.0/24` through the VPNv4 MP-BGP session and installed it in VRF `CUSTOMER_A`.

This confirms the control-plane exchange of the remote customer route and its presence in the VRF routing table. MPLS forwarding and end-to-end connectivity are verified separately.

