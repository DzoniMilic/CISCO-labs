# 05 — Customer A Verification

## 1. Objective

Verify end-to-end connectivity for Customer A across the MPLS L3VPN provider network.

Customer A uses VRF `CUSTOMER_A` on PE1 and PE2. Its routes are exchanged between the PE routers using MP-BGP VPNv4, while eBGP provides routing between each PE and its directly connected CE.

The verification covers:
- CE-PE interface connectivity.
- PE-CE eBGP sessions.
- Customer route advertisement.
- VRF route installation.
- End-to-end customer traffic forwarding.

## 2. Customer A Topology and Addressing

| Device | Interface | IP address | Role |
|---|---|---|---|
| CE1 | PE-facing interface | 10.0.0.3/24 | Customer A, AS 200 |
| PE1 | Gi0/0 | 10.0.0.5/24 | VRF CUSTOMER_A |
| PE1 | Loopback0 | 5.5.5.5/32 | Provider loopback |
| PE2 | Gi0/1 | 10.0.0.4/24 | VRF CUSTOMER_A |
| CE2 | PE-facing interface | 10.0.0.7/24 | Customer A, AS 200 |
| CE1 | Loopback0 | 3.3.3.3/32 | Customer endpoint |
| CE2 | Loopback0 | 7.7.7.7/32 | Customer endpoint |

Customer A uses AS `200` on both CE routers. The PE-CE BGP sessions use `local-as 100` on the PE side, and `as-override` is configured to support the same customer AS at both ends.

## 3. VRF Configuration

The following VRF is configured on both PE routers:

```text
ip vrf CUSTOMER_A
 rd 65000:10
 route-target export 65000:10
 route-target import 65000:10
```

PE1 customer interface:

```text
interface GigabitEthernet0/0
 ip vrf forwarding CUSTOMER_A
 ip address 10.0.0.5 255.255.255.0
```

PE2 customer interface:

```text
interface GigabitEthernet0/1
 ip vrf forwarding CUSTOMER_A
 ip address 10.0.0.4 255.255.255.0
```

Verify VRF assignment and interface status:

```text
show ip vrf
show ip interface brief
show ip route vrf CUSTOMER_A
```

## 4. Verify PE-CE eBGP

### PE1

The Customer A CE neighbor is `10.0.0.3`:

```text
router bgp 65000
 address-family ipv4 vrf CUSTOMER_A
  neighbor 10.0.0.3 remote-as 200
  neighbor 10.0.0.3 local-as 100
  neighbor 10.0.0.3 activate
  neighbor 10.0.0.3 as-override
 exit-address-family
```

### PE2

The Customer A CE neighbor is `10.0.0.7`:

```text
router bgp 65000
 address-family ipv4 vrf CUSTOMER_A
  neighbor 10.0.0.7 remote-as 200
  neighbor 10.0.0.7 local-as 100
  neighbor 10.0.0.7 activate
  neighbor 10.0.0.7 as-override
 exit-address-family
```

### Verification commands

On the PE routers:

```text
show ip bgp vpnv4 all summary
show running-config | section router bgp
show ip route vrf CUSTOMER_A
```

Confirm that the Customer A CE neighbors are established and that expected customer prefixes are learned.

On each CE, verify the BGP neighbor using the address and AS configured for its PE peer.

A BGP session being established does not guarantee that the required customer routes are being advertised or installed.

## 5. Verify Customer A VPN Routes

On PE1 and PE2:

```text
show ip bgp vpnv4 vrf CUSTOMER_A
show ip route vrf CUSTOMER_A
```

Expected customer loopback prefixes:

| Prefix | Expected origin |
|---|---|
| 3.3.3.3/32 | Customer A CE1 |
| 7.7.7.7/32 | Customer A CE2 |

Verify that the remote loopback is learned and installed in the `CUSTOMER_A` routing table on the opposite PE.

The exact next hop and BGP attributes should be confirmed from the live output.

If the prefix appears in the VPNv4 BGP table but not in the VRF routing table, investigate route selection, VRF import policy and next-hop reachability.

## 6. End-to-End Connectivity Test

The primary test is a ping between the Customer A loopbacks.

From CE1:

```text
ping 7.7.7.7 source 3.3.3.3
```

Expected result: successful replies from `7.7.7.7`.

From CE2, test the reverse direction:

```text
ping 3.3.3.3 source 7.7.7.7
```

Expected result: successful replies from `3.3.3.3`.

Record the actual packet loss and response times in the lab results. Do not mark the test successful until the live output confirms it.

## 7. Troubleshooting

### CE-PE BGP session is down

Check:
- CE and PE interface status.
- IP addressing and subnet masks.
- Correct BGP neighbor IP and AS.
- The PE interface's VRF assignment.
- BGP address-family activation.

Commands:

```text
show ip interface brief
show ip bgp summary
show running-config | section router bgp
```

### Remote customer loopback is missing

Check:
1. The remote CE has the loopback configured and advertises it.
2. The local PE learns the prefix from its CE neighbor.
3. The PE-to-PE VPNv4 session is established.
4. The route is exported with the correct RT.
5. The remote PE imports the route into `CUSTOMER_A`.
6. The route is installed in the VRF routing table.

Commands:

```text
show ip bgp vpnv4 all
show ip bgp vpnv4 vrf CUSTOMER_A
show ip route vrf CUSTOMER_A
```

### Routes exist but the ping fails

Check the forwarding path:

```text
show ip route vrf CUSTOMER_A
show mpls forwarding-table
show ip cef
```

Verify that CEF is enabled in the final configuration and that the relevant MPLS forwarding entries are usable. Also confirm that the destination CE has a return route to the source loopback.

## 8. Completion Criteria

- [ ] Customer A interfaces are up and correctly addressed.
- [ ] CE1 and CE2 are connected to the correct PE interfaces.
- [ ] Both Customer A PE-CE BGP sessions are established.
- [ ] The remote customer loopback prefixes are learned.
- [ ] Remote prefixes are installed in the correct `CUSTOMER_A` VRF.
- [ ] CE1 can ping CE2's loopback using the intended source address.
- [ ] The reverse ping also succeeds.
- [ ] CEF and MPLS forwarding are operational.

## 9. Key Concept

Customer A's traffic enters the provider network at a PE, is forwarded using the customer's VRF and VPN routing information, crosses the MPLS core, and exits through the remote PE toward the destination CE.

Successful service depends on the complete chain: PE-CE routing, MP-BGP VPNv4 route exchange, correct VRF import/export policy, MPLS forwarding and a valid return path.

