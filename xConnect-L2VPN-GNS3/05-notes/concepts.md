# Key Concepts

## What is xConnect?

xConnect creates a point-to-point Layer 2 connection between two interfaces across an MPLS network.

In this lab:

```text
CE1 Gi0/0
    |
PE1 Gi0/2
    |
xConnect
    |
MPLS pseudowire
    |
PE2 Gi0/2
    |
CE2 Gi0/0
```

---

## Why do PE1 Gi0/2 and PE2 Gi0/2 have no IP address?

Because they are not being used as Layer 3 routed interfaces.

They are Layer 2 attachment circuits.

The PE does not route:

```text
192.168.10.1 → 192.168.10.2
```

Instead, it transports the Ethernet frame through the pseudowire.

---

## Why can CE1 and CE2 use the same subnet?

Because xConnect provides Layer 2 connectivity.

Both CE routers are effectively connected to the same Layer 2 segment:

```text
192.168.10.0/24
```

The provider core does not need to participate in that customer subnet.

---

## What does MPLS do here?

MPLS provides the transport mechanism through the provider core.

The provider routers do not need to know the customer subnet:

```text
192.168.10.0/24
```

They only need connectivity to the PE loopbacks and MPLS labels.

---

## What does LDP do?

LDP distributes MPLS labels and establishes label-switched paths used by the MPLS transport.

Example:

```text
PE1 → P1 → P2 → PE2
```

LDP neighbors:

```text
PE1 ↔ P1
P1  ↔ P2
P2  ↔ PE2
```

---

## What does OSPF do?

OSPF provides IP reachability inside the provider network.

It allows PE1 to reach PE2's loopback:

```text
11.11.11.11 → 44.44.44.44
```

This reachability is required for the PE-to-PE pseudowire.

---

## What is the VC ID?

The VC ID identifies the pseudowire.

PE1:

```cisco
xconnect 44.44.44.44 10 encapsulation mpls
```

PE2:

```cisco
xconnect 11.11.11.11 10 encapsulation mpls
```

The value:

```text
10
```

must match on both sides.

---

## Troubleshooting Order

If CE1 cannot ping CE2, check in this order:

1. Physical interfaces
2. OSPF neighbors
3. Loopback reachability
4. LDP neighbors
5. MPLS forwarding
6. xConnect state
7. Attachment circuit
8. CE ARP
9. CE-to-CE ping

Do not immediately change the xConnect configuration.

First identify which layer is failing.
