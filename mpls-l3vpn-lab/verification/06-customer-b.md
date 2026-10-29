# 06 — Customer B Verification

## 1. Objective

Verify end-to-end connectivity for Customer B across the MPLS L3VPN provider network.

Customer B uses VRF `CUSTOMER_B` on PE1 and PE2. Its routes are learned from the customer edge (CE) routers and exchanged between the provider edge (PE) routers using MP-BGP VPNv4.

This verification covers:
- Customer-facing interface connectivity.
- PE-CE eBGP sessions.
- Customer route advertisement and installation.
- VRF route isolation.
- End-to-end customer traffic forwarding.

## 2. Customer B Topology and Addressing

| Device | Interface | IP address | Role |
|---|---|---|---|
| CE1 | PE-facing interface | 10.0.0.4/24 | Customer B, AS 300 |
| PE1 | Gi0/1 | 10.0.0.55/24 | VRF CUSTOMER_B |
| PE1 | Loopback0 | 5.5.5.5/32 | Provider loopback |
| PE2 | Gi0/2 | 10.0.0.6/24 | VRF CUSTOMER_B |
| CE2 | PE-facing interface | 10.0.0.8/24 | Customer B, AS 300 |
| CE1 | Loopback0 | 4.4.4.4/32 | Customer endpoint |
| CE2 | Loopback0 | 8.8.8.8/32 | Customer endpoint |

Customer B uses AS `300` on both CE routers. The PE-CE BGP sessions use `local-as 100` on the PE side, with `as-override` configured to support the same customer AS at both ends.

**Addressing note:** Customer B reuses the `10.0.0.0/24` address range used by Customer A. The separate VRFs keep their routing information isolated.

## 3. VRF Configuration

The following VRF is configured on both PE routers:

```text
ip vrf CUSTOMER_B
 rd 65000:20
 route-target export 65000:20
 route-target import 65000:20
```

PE1 customer interface:

```text
interface GigabitEthernet0/1
 ip vrf forwarding CUSTOMER_B
 ip address 10.0.0.55 255.255.255.0
```

PE2 customer interface:

```text
interface GigabitEthernet0/2
 ip vrf forwarding CUSTOMER_B
 ip address 10.0.0.6 255.255.255.0
```

Verify VRF assignment and interface status:

```text
show ip vrf
show ip interface brief
show ip route vrf CUSTOMER_B
```

## 4. Verify PE-CE eBGP

### PE1

The Customer B CE neighbor is `10.0.0.4`:

```text
router bgp 65000
 address-family ipv4 vrf CUSTOMER_B
  redistribute connected
  neighbor 10.0.0.4 remote-as 300
  neighbor 10.0.0.4 local-as 100
  neighbor 10.0.0.4 activate
  neighbor 10.0.0.4 as-override
 exit-address-family
```

### PE2

The Customer B CE neighbor is `10.0.0.8`:

```text
router bgp 65000
 address-family ipv4 vrf CUSTOMER_B
  redistribute connected
  neighbor 10.0.0.8 remote-as 300
  neighbor 10.0.0.8 local-as 100
  neighbor 10.0.0.8 activate
  neighbor 10.0.0.8 as-override
 exit-address-family
```

### Verification commands

On PE1 and PE2:

```text
show ip bgp vpnv4 all summary
show running-config | section router bgp
show ip route vrf CUSTOMER_B
```

Confirm that the Customer B CE neighbors are established and that expected customer prefixes are learned and advertised.

`redistribute connected` is present in the supplied configuration under the `CUSTOMER_B` IPv4 VRF address family. Verify the actual advertised routes rather than assuming that every connected prefix should be exported.

## 5. Verify Customer B VPN Routes

On PE1 and PE2:

```text
show ip bgp vpnv4 vrf CUSTOMER_B
show ip route vrf CUSTOMER_B
```

Expected customer loopback prefixes:

| Prefix | Expected origin |
|---|---|
| 4.4.4.4/32 | Customer B CE1 |
| 8.8.8.8/32 | Customer B CE2 |

Verify that the remote loopback prefix is learned and installed in the correct `CUSTOMER_B` routing table on the opposite PE.

The exact next hop and BGP attributes must be confirmed from the live output.

## 6. End-to-End Connectivity Test

From Customer B CE1, test reachability to CE2's loopback:

```text
ping 8.8.8.8 source 4.4.4.4
```

Expected result: successful replies from `8.8.8.8`.

From Customer B CE2, test the reverse direction:

```text
ping 4.4.4.4 source 8.8.8.8
```

Expected result: successful replies from `4.4.4.4`.

Record the actual packet loss and response times. Mark the tests as successful only after confirming the live output.

## 7. Verify Customer Isolation

Customer A and Customer B use overlapping PE-CE addressing but separate VRFs and Route Targets.

Verify the VRF routing tables independently:

```text
show ip route vrf CUSTOMER_A
show ip route vrf CUSTOMER_B
```

Confirm that:
- Customer B routes are installed in `CUSTOMER_B`.
- Customer A routes are installed in `CUSTOMER_A`.
- The two customers' routing information is not unintentionally imported into the other VRF.

Separate VRFs provide routing-table isolation. Do not assume that a route in one VRF is available in another without explicit route leaking or matching import policy.

## 8. Troubleshooting

### CE-PE BGP session is down

Check:
- Customer-facing interfaces are up.
- IP addresses and subnet masks are correct.
- The correct CE neighbor IP and AS are configured.
- The PE interface belongs to `CUSTOMER_B`.
- The BGP neighbor is activated in the VRF address family.

Commands:

```text
show ip interface brief
show ip bgp vpnv4 all summary
show running-config | section router bgp
```

### Remote customer loopback is missing

Check:
1. The remote CE has the loopback configured.
2. The remote CE advertises its loopback prefix.
3. The local PE learns the prefix from its CE.
4. MP-BGP VPNv4 exchanges the route between the PE routers.
5. The correct Route Target is exported and imported.
6. The route is installed in `CUSTOMER_B`.

Commands:

```text
show ip bgp vpnv4 all
show ip bgp vpnv4 vrf CUSTOMER_B
show ip route vrf CUSTOMER_B
```

### Routes exist but the ping fails

Inspect the route and forwarding path:

```text
show ip route vrf CUSTOMER_B 8.8.8.8
show mpls forwarding-table
show ip cef
```

Confirm that CEF is enabled in the final configuration and that the required MPLS forwarding entries are usable. Check the destination CE's return route as well.

## 9. Completion Criteria

- [ ] Customer B interfaces are up and correctly addressed.
- [ ] Both Customer B PE-CE BGP sessions are established.
- [ ] The remote customer loopback prefixes are learned.
- [ ] Remote routes are installed in the correct `CUSTOMER_B` VRF.
- [ ] CE1 can ping CE2's loopback using source `4.4.4.4`.
- [ ] The reverse ping also succeeds.
- [ ] Customer A and Customer B routing tables remain isolated.
- [ ] CEF and MPLS forwarding are operational.

## 10. Key Concept

Customer B uses the same provider MPLS core as Customer A but has its own VRF and Route Targets. MP-BGP distributes its VPN routes between PE routers, while the VRF keeps those routes separate from Customer A.

Successful connectivity requires correct PE-CE routing, VPNv4 route exchange, VRF route installation, MPLS/CEF forwarding and a valid return path.

