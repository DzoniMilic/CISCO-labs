# Verification 03 — MP-BGP VPNv4

## Objective

Verify that PE1 and PE2 have an established MP-BGP VPNv4 session and exchange customer routes for the `CUSTOMER_A` VRF.

## Commands

```cisco
show ip bgp vpnv4 all summary
show ip bgp vpnv4 all
```

## Results — PE1

### BGP Neighbor Summary

Observed output:

```text
Neighbor        V    AS     MsgRcvd MsgSent Up/Down   State/PfxRcd
44.44.44.44     4    65000  167     168     02:27:29  1
```

The BGP neighbor is PE2, identified by loopback address `44.44.44.44`. Both PE routers use AS `65000`.

The `State/PfxRcd` column contains `1`, indicating that the established neighbor has delivered one prefix to PE1.

### VPNv4 Routing Table

Observed routes for Route Distinguisher `65000:10`:

```text
Network          Next Hop       Metric LocPrf Weight Path
192.168.10.0     0.0.0.0        0             32768  ?
192.168.20.0     44.44.44.44    0       100   0      ?
```

The local customer network is `192.168.10.0/24`. The remote customer network, `192.168.20.0/24`, is learned through PE2 at `44.44.44.44`.

## Interpretation

* The VPNv4 BGP neighbor is established.
* PE1 and PE2 use BGP AS `65000`.
* The Route Distinguisher `65000:10` identifies the VPN routing context.
* PE1 has learned the remote customer prefix `192.168.20.0/24` from PE2.
* The VPNv4 address family carries VPN routes between PE routers; the customer VRF routing table is checked separately.

## Conclusion

**MP-BGP VPNv4 verification passed on PE1** for the observed neighbor state and remote customer route.

The next verification step is to confirm that the learned VPN route is installed in the `CUSTOMER_A` VRF routing table and can be resolved for forwarding.

