# Lessons Learned — MPLS L3VPN Lab

## 1. OSPF in the Provider Core

- OSPF provides IP reachability between provider routers (P routers and PE routers).
- Loopback interfaces are used as stable router IDs and as endpoints for MP-BGP sessions.
- OSPF neighbor states should reach `FULL` before proceeding with MPLS and MP-BGP troubleshooting.
- OSPF provides underlay reachability; it does not carry customer VPN routes in this lab.

**Verification commands:**
```text
show ip ospf neighbor
show ip route ospf
show ip interface brief
```

## 2. MPLS and LDP

- MPLS must be enabled on the provider-facing interfaces that participate in label switching.
- LDP distributes labels between MPLS-enabled routers.
- The LDP router ID is configured to use Loopback0 for stable identification.
- The command `mpls ldp autoconfig` under OSPF enables LDP on eligible OSPF-enabled interfaces on supported IOS platforms.
- An established LDP session does not by itself prove that end-to-end customer traffic is forwarding correctly.

**Verification commands:**
```text
show mpls ldp neighbor
show mpls forwarding-table
show mpls ldp parameters
ping 2.2.2.2 source 1.1.1.1
```

## 3. VRFs and Customer Isolation

- A VRF maintains a separate routing table for a customer.
- Customer A and Customer B use overlapping IPv4 addressing in this lab.
- Separate VRFs allow both customers to use `10.0.0.0/24` without merging their routing information.
- Each VRF uses its own Route Distinguisher (RD) and Route Targets (RTs).

| Customer | VRF | RD | Import RT | Export RT |
|---|---|---|---|---|
| Customer A | CUSTOMER_A | 65000:10 | 65000:10 | 65000:10 |
| Customer B | CUSTOMER_B | 65000:20 | 65000:20 | 65000:20 |

**Verification commands:**
```text
show ip vrf
show ip route vrf CUSTOMER_A
show ip route vrf CUSTOMER_B
```

## 4. MP-BGP VPNv4

- MP-BGP exchanges VPN routes between PE routers.
- The PE-to-PE BGP session uses provider loopbacks and AS 65000.
- The VPNv4 address family carries VPN routes with their associated VPN information.
- Extended communities are sent so that Route Targets can be communicated.
- The PE-CE routing sessions are configured separately inside each customer's VRF address family.

**Verification commands:**
```text
show ip bgp vpnv4 all summary
show ip bgp vpnv4 all
show ip bgp vpnv4 vrf CUSTOMER_A
show ip bgp vpnv4 vrf CUSTOMER_B
```

## 5. PE-CE eBGP and AS Override

- Each customer VRF has its own PE-CE BGP neighbor relationship.
- `local-as 100` changes the AS presented by the PE to the customer neighbor in this lab.
- `as-override` replaces the customer's AS in advertised routes with the PE's AS, helping prevent rejection by a remote CE using the same customer AS.
- The Customer A CEs use AS 200; the Customer B CEs use AS 300.
- BGP session establishment alone does not prove that the correct customer prefixes have been advertised and installed in the routing table.

**Verification commands:**
```text
show ip bgp vpnv4 all summary
show ip route vrf CUSTOMER_A
show ip route vrf CUSTOMER_B
```

## 6. CEF and MPLS Forwarding

CEF (Cisco Express Forwarding) is an important component of IP packet forwarding on this IOSv platform.

During a controlled test on PE1:

1. With CEF enabled, a ping from the Customer B CE to `8.8.8.8`, sourced from `4.4.4.4`, succeeded.
2. CEF was disabled using `no ip cef`.
3. `show ip cef` reported that IPv4 CEF was not running.
4. The MPLS forwarding table showed entries with `No Label` and `drop`.
5. OSPF, LDP and BGP control-plane information remained present, and VPNv4 routes were still visible.
6. The same customer-to-customer ping failed while CEF was disabled.
7. CEF was re-enabled with `ip cef`.

**Conclusion:** In this IOSv lab, disabling CEF disrupted MPLS data-plane forwarding even though key control-plane sessions and routes remained present. This demonstrates why control-plane status alone is insufficient to confirm end-to-end service health.

**Verification commands:**
```text
show ip cef
show mpls forwarding-table
show ip bgp vpnv4 all summary
show ip route vrf CUSTOMER_B
ping 8.8.8.8 source 4.4.4.4
```

The end-to-end ping should be repeated after restoring CEF to confirm that forwarding has recovered.

## 7. Troubleshooting Method

Troubleshooting should proceed in layers, from the provider underlay to the customer service:

1. **Physical and interface state** — confirm that the relevant interfaces are up/up and correctly addressed.
2. **IGP underlay** — verify OSPF neighbors and reachability to provider loopbacks.
3. **MPLS/LDP** — verify LDP neighbors and label forwarding entries.
4. **MP-BGP VPNv4** — verify that the PE-to-PE session is established and VPN routes are exchanged.
5. **VRF routing** — verify that each customer's routes are installed in the correct VRF.
6. **Data plane** — test connectivity from the customer CE and inspect CEF/MPLS forwarding if traffic fails.

A successful result at one layer does not guarantee success at the next layer. For example, an established BGP session does not guarantee that packets can be forwarded through the MPLS data plane.

## 8. Key Takeaways

- OSPF provides provider-core IP reachability.
- LDP distributes MPLS labels.
- MP-BGP distributes VPN routes between PE routers.
- VRFs isolate customer routing tables.
- Route Targets control VPN route import and export.
- `as-override` helps support customers using the same AS at both ends.
- CEF and the MPLS forwarding table are essential parts of data-plane troubleshooting.
- Always verify both control-plane state and actual end-to-end traffic forwarding.

