# 03 — VRF Configuration and Verification

## 1. Objective

Verify Virtual Routing and Forwarding (VRF) configuration on PE1 and PE2.

A VRF maintains a separate routing table for a customer. It allows multiple customers to use overlapping IP address ranges without mixing their routing information.

In this lab:
- Customer A uses VRF `CUSTOMER_A`.
- Customer B uses VRF `CUSTOMER_B`.
- Both customers use the `10.0.0.0/24` address range on their PE-CE networks.
- Each customer has its own Route Distinguisher (RD) and Route Targets (RTs).

## 2. VRF Design

| Parameter | Customer A | Customer B |
|---|---|---|
| VRF name | CUSTOMER_A | CUSTOMER_B |
| RD | 65000:10 | 65000:20 |
| Import RT | 65000:10 | 65000:20 |
| Export RT | 65000:10 | 65000:20 |
| Customer AS | 200 | 300 |

The same RD and RT values are configured on PE1 and PE2 for the corresponding customer VRF.

### Route Distinguisher (RD)

The RD makes otherwise overlapping IPv4 prefixes distinguishable in the VPNv4 routing table.

For example, both customers may use `10.0.0.0/24`, but their VPN routes are distinguished using their respective RDs.

### Route Target (RT)

Route Targets are BGP extended communities used to control which VPN routes a VRF imports and exports.

In this lab, each customer imports routes tagged with its own RT. This keeps Customer A and Customer B routing information separate.

The RD identifies VPN prefixes; the RT controls VPN route distribution.

## 3. VRF Configuration

The following configuration is present on PE1 and PE2.

### Customer A

```text
ip vrf CUSTOMER_A
 rd 65000:10
 route-target export 65000:10
 route-target import 65000:10
```

### Customer B

```text
ip vrf CUSTOMER_B
 rd 65000:20
 route-target export 65000:20
 route-target import 65000:20
```

## 4. Assign PE-CE Interfaces to VRFs

A customer-facing interface must be assigned to the correct VRF.

### PE1

```text
interface GigabitEthernet0/0
 ip vrf forwarding CUSTOMER_A
 ip address 10.0.0.5 255.255.255.0
!
interface GigabitEthernet0/1
 ip vrf forwarding CUSTOMER_B
 ip address 10.0.0.55 255.255.255.0
```

### PE2

```text
interface GigabitEthernet0/1
 ip vrf forwarding CUSTOMER_A
 ip address 10.0.0.4 255.255.255.0
!
interface GigabitEthernet0/2
 ip vrf forwarding CUSTOMER_B
 ip address 10.0.0.6 255.255.255.0
```

**Important:** On Cisco IOS, applying `ip vrf forwarding` to an interface can remove its existing IP address. Configure the VRF assignment first, then configure the interface IP address.

## 5. Verification Commands

### 5.1 Verify configured VRFs

Run on PE1 and PE2:

```text
show ip vrf
```

Confirm that `CUSTOMER_A` and `CUSTOMER_B` exist and that the customer-facing interfaces are associated with the correct VRFs.

### 5.2 Verify interface addressing

```text
show ip interface brief
show running-config
```

Check that:
- PE1 Gi0/0 belongs to `CUSTOMER_A`.
- PE1 Gi0/1 belongs to `CUSTOMER_B`.
- PE2 Gi0/1 belongs to `CUSTOMER_A`.
- PE2 Gi0/2 belongs to `CUSTOMER_B`.
- The interfaces have the expected IP addresses and are operational.

### 5.3 Verify each VRF routing table

```text
show ip route vrf CUSTOMER_A
show ip route vrf CUSTOMER_B
```

These commands display the routing table for the selected VRF rather than the global routing table.

Check for:
- Connected customer-facing subnets.
- Local interface addresses.
- Expected remote customer routes, once the relevant routing protocols and VPN route exchange are operational.

### 5.4 Verify a specific destination

Example on PE1:

```text
show ip route vrf CUSTOMER_B 8.8.8.8
```

This checks how PE1 routes traffic destined for `8.8.8.8` within `CUSTOMER_B`.

In the lab, this route is expected to be learned through BGP after VPN route exchange is working. Its exact next hop and MPLS details should be verified from the live output.

## 6. VRF Routing and Customer Isolation

The global routing table and each VRF routing table are separate.

A route existing in the global routing table does not automatically mean that it exists in a customer's VRF. Likewise, a route in one customer VRF is not automatically available in another customer VRF.

This separation is important because Customer A and Customer B both use `10.0.0.0/24`.

To troubleshoot a customer route, always specify the appropriate VRF when checking its routing table.

## 7. Troubleshooting

### VRF is missing

Commands:

```text
show ip vrf
show running-config
```

Check that the VRF is configured with the correct name and RD.

### Customer interface is in the wrong VRF

Commands:

```text
show ip vrf
show running-config
show ip interface brief
```

Check the `ip vrf forwarding` command under the interface and verify its IP address.

### Connected route is missing

Commands:

```text
show ip interface brief
show ip route vrf CUSTOMER_A
show ip route vrf CUSTOMER_B
```

Check interface state, VRF assignment and IP addressing.

### Remote customer route is missing

Check the relevant layers in order:

1. Confirm the local customer-facing interface and VRF.
2. Verify the CE-PE routing relationship.
3. Check whether the route exists in the PE's BGP VRF address family.
4. Verify PE-to-PE MP-BGP VPNv4 exchange.
5. Check Route Target import/export configuration.
6. Confirm that the route is installed in the destination VRF routing table.

Useful commands:

```text
show ip bgp vpnv4 all summary
show ip bgp vpnv4 all
show ip route vrf CUSTOMER_A
show ip route vrf CUSTOMER_B
```

A route visible in the VPNv4 BGP table is not necessarily installed in the VRF routing table.

## 8. Completion Criteria

- [ ] `CUSTOMER_A` and `CUSTOMER_B` exist on PE1 and PE2.
- [ ] Each VRF has the correct RD and Route Targets.
- [ ] Customer-facing interfaces are assigned to the correct VRFs.
- [ ] Connected routes appear in the appropriate VRF tables.
- [ ] Expected remote customer routes appear once routing exchange is operational.
- [ ] Customer A and Customer B routing information remains isolated.

## 9. Key Concept

VRFs provide customer routing separation on PE routers. RDs distinguish overlapping prefixes in VPNv4, while RTs determine which VPN routes a VRF imports and exports.

VRF configuration alone does not provide end-to-end connectivity. The CE-PE routing protocol, MP-BGP VPNv4 exchange, MPLS transport and packet forwarding must also be working.

