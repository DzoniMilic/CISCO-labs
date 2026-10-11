## CE-to-CE Ping Verification## 1. Goal
This is the final end-to-end test of the whole lab.
Customer devices:

CE1
Gi0/0 = 192.168.10.1/24

and:

CE2
Gi0/0 = 192.168.10.2/24

are located in the same Layer-3 subnet network:

192.168.10.0/24

Between them there is no Layer-3 routing on PE devices.
Traffic is transported as Layer-2 Ethernet over xConnect pseudowire.
------------------------------
## 2. Verification of CE1 interface
On CE1:

show ip interface brief

Expected:

GigabitEthernet0/0    192.168.10.1    up    up

------------------------------
## 3. Verification of CE2 interface
On CE2:

show ip interface brief

Expected:

GigabitEthernet0/0    192.168.10.2    up    up

------------------------------
## 4. CE1 -> CE2 ping
On CE1:

ping 192.168.10.2

Successful result:

!!!!!
Success rate is 100 percent (5/5)

In this lab a successful result was obtained:

Success rate is 100 percent (5/5)
round-trip min/avg/max = 12/16/24 ms

------------------------------
## 5. CE2 -> CE1 ping
On CE2:

ping 192.168.10.1

Expected:

!!!!!
Success rate is 100 percent (5/5)

Bi-directional ping is important because it confirms that communication works in both directions.
------------------------------
## 6. What happens when CE1 pings CE2
CE1 verifies that:

192.168.10.2

is in the same local subnet network.
Then CE1 uses ARP to learn the MAC address of CE2.
ARP broadcast goes:

CE1
 ↓
PE1 Gi0/2
 ↓
xConnect
 ↓
MPLS pseudowire
 ↓
PE2 Gi0/2
 ↓
CE2

CE2 replies with its MAC addresses.
After that ICMP Ethernet frame goes via the same L2VPN path between CE devices.
------------------------------
## 7. Important thing: PE routers do not route 192.168.10.0/24
PE1 has no need for:

192.168.10.0/24

in its routing table because of this xConnect.
The same applies to PE2.
Provider core knows how to reach:

PE1 Loopback = 11.11.11.11
PE2 Loopback = 44.44.44.44

MPLS/LDP provides transport through the core.
xConnect connects customer Ethernet domains.
------------------------------
## 8. End-to-end chain

CE1
192.168.10.1
   |
   | Ethernet
   |
PE1
11.11.11.11
   |
   | MPLS
   |
P1
1.1.1.1
   |
   | MPLS
   |
P2
2.2.2.2
   |
   | MPLS
   |
PE2
44.44.44.44
   |
   | Ethernet
   |
CE2
192.168.10.2

------------------------------
## 9. What CE-to-CE ping proves
Successful ping proves that the following are simultaneously functional:

CE interfaces
      ↓
PE attachment circuits
      ↓
xConnect
      ^^
MPLS pseudowire
      ↓
LDP
      ↓
MPLS core
      ↓
OSPF reachability
      ↓
remote PE
      ↓
CE2

That is why CE-to-CE ping is the most important final test of this lab.
------------------------------
## 10. Troubleshooting if ping does not work
If:

CE1 -> CE2 = FAIL

one should not immediately change the CE configuration.
Verify in order:
## 1. CE interfaces

show ip interface brief

Must be:

up/up

## 2. PE attachment circuit
On PE1/PE2:

show interfaces GigabitEthernet0/2

Must be:

up/up

## 3. xConnect

show xconnect all

Must be:

UP

## 4. LDP

show mpls ldp neighbor

All relevant neighbors must be:

Oper

## 5. PE Loopback reachability
From PE1:

ping 44.44.44.44 source 11.11.11.11

## 6. ARP
On CE1:

show arp

On CE2:

show arp

If CE devices do not learn each other's MAC/ARP information, verify the L2 path.
------------------------------
## 11. Final verification checklist

[OK] CE1 Gi0/0 = 192.168.10.1/24
[OK] CE2 Gi0/0 = 192.168.10.2/24
[OK] CE1 interface = up/up
[OK] CE2 interface = up/up

[OK] PE1 Gi0/2 = up/up
[OK] PE2 Gi0/2 = up/up

[OK] OSPF adjacencies = FULL
[OK] PE Loopbacks reachable

[OK] LDP neighbors = Oper
[OK] MPLS forwarding operational

[OK] PE1 xConnect = UP
[OK] PE2 xConnect = UP
[OK] Attachment circuits = UP
[OK] Pseudowire = UP

[OK] CE1 -> CE2 ping = 100%
[OK] CE2 -> CE1 ping = 100%

------------------------------
## 12. Final result
Cisco xConnect L2VPN is successfully implemented over the MPLS provider network.
Customer network:

192.168.10.0/24

functions as a single Layer-2 Ethernet domain over the provider infrastructure:

CE1 — PE1 — P1 — P2 — PE2 — CE2

Provider core uses:

OSPF
LDP
MPLS

while xConnect provides:

Layer-2 pseudowire

between PE1 and PE2.
Final proof of functionality:

CE1 -> CE2 = 100%
CE2 -> CE1 = 100%
