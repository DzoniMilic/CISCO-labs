# 04 — MP-BGP VPNv4 Verification

## 1. Objective

Verify Multiprotocol BGP (MP-BGP) operation between PE1 and PE2 in the MPLS L3VPN lab.

MP-BGP exchanges customer VPN routes between Provider Edge (PE) routers. It carries VPNv4 prefixes, which combine IPv4 prefixes with Route Distinguisher (RD) information, and uses extended communities to carry Route Targets.

In this lab:
- PE1 uses loopback `5.5.5.5`.
- PE2 uses loopback `6.6.6.6`.
- Both PE routers belong to provider BGP AS `65000`.
- The PE-to-PE session uses iBGP over the provider IP underlay.
- Customer routes are exchanged through the `vpnv4` address family.

## 2. MP-BGP Design

| Parameter | PE1 | PE2 |
|---|---|---|
| BGP AS | 65000 | 65000 |
| Loopback0 | 5.5.5.5/32 | 6.6.6.6/32 |
| BGP neighbor | 6.6.6.6 | 5.5.5.5 |
| Update source | Loopback0 | Loopback0 |
| VPN address family | vpnv4 | vpnv4 |

The provider underlay must provide IP reachability between the PE loopbacks before the BGP session can establish.

## 3. Configuration

### 3.1 PE1

```text
router bgp 65000
 bgp log-neighbor-changes
 neighbor 6.6.6.6 remote-as 65000
 neighbor 6.6.6.6 update-source Loopback0
 !
 address-family vpnv4
  neighbor 6.6.6.6 activate
  neighbor 6.6.6.6 send-community extended
 exit-address-family
```

### 3.2 PE2

```text
router bgp 65000
 bgp log-neighbor-changes
 neighbor 5.5.5.5 remote-as 65000
 neighbor 5.5.5.5 update-source Loopback0
 !
 address-family vpnv4
  neighbor 5.5.5.5 activate
  neighbor 5.5.5.5 send-community extended
 exit-address-family
```

These are the relevant MP-BGP configuration sections, not complete router configurations.

### 3.3 Explanation of key commands

- `router bgp 65000` — starts the BGP process in AS 65000.
- `neighbor ... remote-as 65000` — defines the remote PE as an iBGP neighbor in the same AS.
- `neighbor ... update-source Loopback0` — uses Loopback0 as the source of the BGP session.
- `address-family vpnv4` — enables the VPNv4 address family.
- `neighbor ... activate` — activates the neighbor for VPNv4 exchange.
- `neighbor ... send-community extended` — sends extended communities, including Route Targets, to the peer.

The PE loopbacks must be reachable through the provider underlay. The BGP session must also be established before VPN routes can be exchanged.

## 4. Verify the BGP Session

Run on PE1 and PE2:

```text
show ip bgp vpnv4 all summary
```

Check:
- The expected PE neighbor is present.
- The session is established.
- The neighbor address matches the remote PE loopback.
- Prefix counters indicate whether routes are being received.

On this IOS version, an established neighbor typically displays a numeric prefix count in the final column. A neighbor that is not established displays a state such as `Idle` or `Active`.

An established session confirms that the BGP control-plane connection is operational. It does not by itself prove that the correct customer routes are present or that customer traffic can be forwarded.

## 5. Verify VPNv4 Routes

Run:

```text
show ip bgp vpnv4 all
```

To inspect routes for a particular customer VRF:

```text
show ip bgp vpnv4 vrf CUSTOMER_A
show ip bgp vpnv4 vrf CUSTOMER_B
```

Check:
- Expected customer prefixes are present.
- Routes are learned from the correct PE or local customer routing process.
- RD and Route Target associations are consistent with the VRF design.
- Expected remote customer routes are being advertised and received.

Example customer loopbacks in this lab:

| Customer | Local-side loopback | Remote-side loopback |
|---|---|---|
| Customer A | 3.3.3.3/32 | 7.7.7.7/32 |
| Customer B | 4.4.4.4/32 | 8.8.8.8/32 |

These prefixes should be checked against the actual BGP output. Their presence in a VPNv4 table depends on the CE-PE routing configuration and route advertisement.

## 6. Route Targets and Extended Communities

The VRFs use the following Route Targets:

| VRF | RD | Export RT | Import RT |
|---|---|---|---|
| CUSTOMER_A | 65000:10 | 65000:10 | 65000:10 |
| CUSTOMER_B | 65000:20 | 65000:20 | 65000:20 |

Route Targets are carried as BGP extended communities. They control which VPN routes are eligible for import into a VRF.

The `send-community extended` command is important because it allows the PE to send the extended communities required for the receiving PE to apply the intended VPN route-import policy.

The RD and RT have different roles:
- **RD:** distinguishes VPNv4 prefixes that might otherwise overlap.
- **RT:** controls VPN route import and export policy.

## 7. Verify VRF Route Installation

After confirming that VPNv4 routes are exchanged, verify that the expected remote routes are installed in the appropriate VRF routing table.

```text
show ip route vrf CUSTOMER_A
show ip route vrf CUSTOMER_B
```

For a specific destination:

```text
show ip route vrf CUSTOMER_B 8.8.8.8
```

Check:
- The route is present in the correct VRF.
- The routing protocol and next hop are appropriate.
- The route is installed in the routing table, not merely visible in BGP.

A route may be present in the VPNv4 BGP table but not installed in the VRF routing table. If this occurs, inspect route selection, competing routes, next-hop reachability and VRF import policy.

## 8. Troubleshooting

### BGP neighbor is not established

Check:

```text
show ip interface brief
show ip route 6.6.6.6
show ip bgp vpnv4 all summary
show running-config | section router bgp
```

On PE2, check reachability to `5.5.5.5` instead.

Verify:
1. Provider loopbacks are reachable.
2. The neighbor address and remote AS are correct.
3. `update-source Loopback0` is configured.
4. The VPNv4 address family is activated for the neighbor.
5. No TCP connectivity or configuration issue prevents session establishment.

### BGP is established but VPN routes are missing

Check:

```text
show ip bgp vpnv4 all
show ip bgp vpnv4 all summary
show ip route vrf CUSTOMER_A
show ip route vrf CUSTOMER_B
```

Then verify:
- The CE-PE routing sessions are operational.
- Customer routes are learned by the local PE.
- The VPNv4 address family is active.
- Extended communities are sent.
- RD and Route Target import/export values are correct.

### VPNv4 routes exist but customer traffic fails

Check the VRF routing table and MPLS forwarding path:

```text
show ip route vrf CUSTOMER_B 8.8.8.8
show mpls forwarding-table
show ip cef
```

Confirm that CEF is enabled in the final configuration. A working BGP control plane does not guarantee successful data-plane forwarding.

## 9. Completion Criteria

- [ ] PE1 and PE2 have an established iBGP session over their loopbacks.
- [ ] The VPNv4 address family is activated on both sides.
- [ ] Extended communities are sent to the remote PE.
- [ ] Expected customer prefixes are present in the VPNv4 BGP table.
- [ ] Expected remote prefixes are installed in the correct VRF routing tables.
- [ ] MPLS forwarding and end-to-end customer connectivity are verified separately.

## 10. Key Concept

MP-BGP VPNv4 distributes customer VPN routes between PE routers. VRFs maintain customer-specific routing tables, RDs distinguish overlapping prefixes, and RT extended communities control route import and export.

MP-BGP depends on provider underlay reachability, while successful end-to-end traffic also depends on correct VRF routing and a working MPLS/CEF data plane.

