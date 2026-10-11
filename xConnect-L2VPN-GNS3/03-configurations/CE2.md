# CE2 — Final Configuration

## Role

Customer Edge router.

CE2 je povezan direktno na PE2 i predstavlja jednu stranu customer Layer-2 mreže.

## Interface Configuration

```cisco
hostname CE2

interface GigabitEthernet0/0
 description CUSTOMER-LAN-to-PE2
 ip address 192.168.10.2 255.255.255.0
 no shutdown
```

## Verification

```cisco
show ip interface brief
show running-config interface GigabitEthernet0/0
show arp
ping 192.168.10.2
```

Expected:

```text
GigabitEthernet0/0    192.168.10.2    up    up
```

CE2 ne koristi:

* OSPF
* LDP
* MPLS
* xConnect

CE2 samo šalje Ethernet/IP saobraćaj prema PE2.

