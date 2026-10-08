# PE1 — Final Configuration

## Role

Provider Edge router.

PE1 povezuje CE1 sa MPLS provider core-om i predstavlja prvi endpoint xConnect pseudowire-a.

## Loopback

```cisco
hostname PE1

interface Loopback1
 ip address 11.11.11.11 255.255.255.255
```

## Core Interface — PE1 to P1

```cisco
interface GigabitEthernet0/1
 description MPLS-CORE-to-P1
 ip address 10.10.10.1 255.255.255.252
 no shutdown
```

## Customer-facing Interface — PE1 to CE1

```cisco
interface GigabitEthernet0/2
 description L2VPN-AC-to-CE1
 no ip address
 no shutdown
 xconnect 44.44.44.44 10 encapsulation mpls
```

The CE-facing interface is intentionally Layer 2.

There is no IP address on Gi0/2 because this interface is used as the Attachment Circuit for the xConnect.

## OSPF

```cisco
router ospf 1
 network 10.10.10.0 0.0.0.3 area 0
 network 11.11.11.11 0.0.0.0 area 0
 mpls ldp autoconfig
```

## MPLS / LDP

```cisco
mpls ldp router-id Loopback1
```

## Important xConnect Parameters

```text
Local PE:       PE1
Local AC:       Gi0/2
Remote PE:      44.44.44.44
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
mpls 44.44.44.44:10
```
