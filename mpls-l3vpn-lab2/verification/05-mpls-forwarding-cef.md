# MPLS Forwarding Verification with CEF

## Objective

Verify that PE1 has a resolved forwarding path to the remote customer network and that CEF identifies the MPLS labels used to transport customer traffic across the provider network.

## Command

```text
PE1# show ip cef vrf CUSTOMER_A 192.168.20.1
```

## Observed Output

```text
192.168.20.0/24
  nexthop 10.10.10.2 GigabitEthernet0/1 label 16() 21()
```

## Analysis

* `192.168.20.0/24` — The remote customer network behind CE2.
* `10.10.10.2` — The next-hop address of P1.
* `GigabitEthernet0/1` — PE1's provider-facing interface toward P1.
* `label 16()` — The outer transport label used to forward traffic through the MPLS core.
* `21()` — The inner VPN label associated with forwarding the customer traffic to the appropriate VRF on the remote PE.

The CEF entry shows that PE1 has resolved the next hop and has MPLS label information for the remote customer prefix.

## Conclusion

PE1 has a resolved CEF forwarding entry for the remote customer network `192.168.20.0/24`. The entry identifies P1 as the next hop and displays both the transport and VPN labels.

This verifies the forwarding information available on PE1. The forwarding behavior across the provider core and end-to-end customer connectivity are verified separately.

