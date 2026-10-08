# CE1 — Final Configuration

## Role

Customer Edge router.

CE1 je povezan direktno na PE1 i predstavlja jednu stranu customer Layer-2 mreže.

## Interface Configuration

```cisco
hostname CE1

interface GigabitEthernet0/0
 description CUSTOMER-LAN-to-PE1
 ip address 192.168.10.1 255.255.255.0
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
GigabitEthernet0/0    192.168.10.1    up    up
```

CE1 ne koristi:

* OSPF
* LDP
* MPLS
* xConnect

CE1 samo šalje Ethernet/IP saobraćaj prema PE1.

