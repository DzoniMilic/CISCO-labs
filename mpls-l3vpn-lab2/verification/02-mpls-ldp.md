# Verification 02 — MPLS and LDP

## Objective

Verify that PE1 has an operational LDP session with its directly connected provider router and that MPLS forwarding entries are installed.

## Commands

```cisco
show mpls ldp neighbor
show mpls forwarding-table
```

## Results — PE1

### LDP Neighbor

Observed output:

```text
Peer LDP Ident: 1.1.1.1:0; Local LDP Ident 11.11.11.11:0
State: Oper
Up time: 03:24:59
LDP discovery interface: GigabitEthernet0/1
Peer address: 10.10.10.2
```

PE1 has an operational LDP session with P1. The local LDP identifier is `11.11.11.11`, and the peer LDP identifier is `1.1.1.1`.

### MPLS Forwarding Table

Selected entries observed on PE1:

| Local Label | Outgoing Label | Prefix                | Outgoing Interface   | Next Hop   |
| ----------: | -------------- | --------------------- | -------------------- | ---------- |
|          16 | 16             | 44.44.44.44/32        | Gi0/1                | 10.10.10.2 |
|          17 | 17             | 2.2.2.2/32            | Gi0/1                | 10.10.10.2 |
|          18 | Pop Label      | 1.1.1.1/32            | Gi0/1                | 10.10.10.2 |
|          19 | 18             | 10.10.10.12/30        | Gi0/1                | 10.10.10.2 |
|          20 | Pop Label      | 10.10.10.8/30         | Gi0/1                | 10.10.10.2 |
|          21 | No Label       | 192.168.10.0/24 [VRF] | aggregate/CUSTOMER_A | —          |

## Interpretation

* The `Oper` state confirms that the LDP session with P1 is established.
* The forwarding table contains label-switched entries for provider prefixes.
* `Pop Label` indicates that the outgoing label is removed before forwarding to the next hop.
* The customer subnet `192.168.10.0/24` appears as a VRF-associated entry, rather than an ordinary provider-core transport prefix.
* The displayed next hop for remote provider prefixes is P1 (`10.10.10.2`).

## Conclusion

**LDP verification passed on PE1** for the observed peer and forwarding entries.

This output verifies PE1's LDP session with P1 and its local MPLS forwarding state. It does not, by itself, prove that all LDP sessions throughout the provider core are operational.
