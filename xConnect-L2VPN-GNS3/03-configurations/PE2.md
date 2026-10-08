# PE2 — Final Configuration

## Role

Provider Edge router.

PE2 connects the MPLS provider core to CE2 and represents the remote endpoint of the xConnect pseudowire.

## Loopback

```cisco
hostname PE2

interface Loopback1
 ip address 44.44.44.44 255.255.255.255
```

## Core Interface — PE2 to P2

```cisco
interface GigabitEthernet0/1
 description MPLS-CORE-to-P2
 ip address 10.10.10.14 255.255.255.252
 no shutdown
```

## Customer-facing Interface — PE2 to CE2

```cisco
interface GigabitEthernet0/2
 description L2VPN-AC-to-CE2
 no ip address
 no shutdown
 xconnect 11.11.11.11 10 encapsulation mpls
```

The CE-facing interface is intentionally Layer 2 and has no IP address.

## OSPF

```cisco
router ospf 1
 network 10.10.10.12 0.0.0.3 area 0
 network 44.44.44.44 0.0.0.0 area 0
 mpls ldp autoconfig
```

## MPLS / LDP

```cisco
mpls ldp router-id Loopback1
```

## Important xConnect Parameters

```text
Local PE:       PE2
Local AC:       Gi0/2
Remote PE:      11.11.11.11
VC ID:          10
Encapsulation:  MPLS
```

## Verification

```cisco
show ip interface brief
show ip ospf neighbor
show ip route
show mpls interfaces
show mpls ldp neighbor
show mpls forwarding-table
show xconnect all
show interfaces GigabitEthernet0/2
show interfaces pseudowire0
```

## Expected xConnect

```text
UP
ac Gi0/2
mpls 11.11.11.11:10
```

