## xConnect Verification## 1. Goal
xConnect represents a Layer-2 VPN pseudowire between PE1 and PE2.
In this lab:

CE1
 |
PE1
 |
P1
 |
P2
 |
PE2
 |
CE2

xConnect endpoints:

PE1 Gi0/2
     |
     | VC ID 10
     |
PE2 Gi0/2

PE1 uses the remote PE Loopback:

44.44.44.44

PE2 uses:

11.11.11.11

------------------------------
## 2. Configuration of PE1

interface GigabitEthernet0/2
 xconnect 44.44.44.44 10 encapsulation mpls

## 3. Configuration of PE2

interface GigabitEthernet0/2
 xconnect 11.11.11.11 10 encapsulation mpls

VC ID is:

10

on both sides.
------------------------------
## 4. Verification of xConnect
On PE1:

show xconnect all

Expected:

XC ST  Segment 1                      S1  Segment 2
UP     ac Gi0/2:4(Ethernet)           UP  mpls 44.44.44.44:10

On PE2:

show xconnect all

Expected:

XC ST  Segment 1                      S1  Segment 2
UP     ac Gi0/2:4(Ethernet)           UP  mpls 11.11.11.11:10

------------------------------
## 5. How to read the output
Example:

UP

the first status means that xConnect is active.
Attachment Circuit:

ac Gi0/2

represents the local Ethernet side towards the CE device.
The other side:

mpls 44.44.44.44:10

represents the MPLS pseudowire towards the remote PE.
------------------------------
## 6. What must be UP
For a proper xConnect, the following must be UP:

xConnect       = UP
Attachment     = UP
Pseudowire     = UP

If:

AC = DOWN

the problem is local between CE and PE.
If:

AC = UP
PW = DOWN

the problem is in provider transport / LDP / remote PE / VC configuration.
------------------------------
## 7. Verification of CE-facing interfaces
On PE1:

show interfaces GigabitEthernet0/2

On PE2:

show interfaces GigabitEthernet0/2

Expected:

GigabitEthernet0/2 is up, line protocol is up

The interface has no IP address:

Internet protocol processing disabled

or the corresponding unassigned display in:

show ip interface brief

That is normal.
------------------------------
## 8. Why PE Gi0/2 has no IP address
Gi0/2 is not a Layer-3 routed interface.
It represents:

Attachment Circuit

from the CE device to the pseudowire.
CE1 has:

192.168.10.1/24

CE2 has:

192.168.10.2/24

PE1 and PE2 do not need to have an IP address on those interfaces.
The Ethernet frame from CE1 is forwarded through the pseudowire to CE2.
------------------------------
## 9. Verification of pseudowire status
On PE1:

show xconnect all

On PE2:

show xconnect all

One must see:

mpls <remote-PE-loopback>:10

and status:

UP

VC ID must be the same on both sides:

PE1 = 10
PE2 = 10

------------------------------
## 10. Verification of pseudowire interface
On PE1:

show interfaces pseudowire0

On PE2:

show interfaces pseudowire0

It should be checked that the pseudowire interface is operational.
During successful pseudowire formation, the following may appear:

%LINEPROTO-5-UPDOWN:
Line protocol on Interface pseudowire0, changed state to up

------------------------------
## 11. Verification of MAC/ARP behavior
On CE1:

show arp

On CE2:

show arp

After a mutual ping, CE devices should learn the ARP/MAC information required for communication.
Important thing:
ARP request is a broadcast.
xConnect allows the Ethernet broadcast from one CE to be transported over the pseudowire to the other CE.
That is why CE1 and CE2 can be in the same subnet network:

192.168.10.0/24

even though there is an MPLS provider core between them.
------------------------------
## 12. End-to-end test
On CE1:

ping 192.168.10.2

On CE2:

ping 192.168.10.1

If the ping works, it is proven that the L2VPN functions end-to-end.
------------------------------
## 13. xConnect verification checklist

[OK] PE1 Gi0/2 = up/up
[OK] PE2 Gi0/2 = up/up
[OK] PE1 xConnect = UP
[OK] PE2 xConnect = UP
[OK] Attachment Circuit = UP
[OK] Pseudowire = UP
[OK] VC ID PE1 = 10
[OK] VC ID PE2 = 10
[OK] Remote PE Loopbacks reachable
[OK] CE Ethernet connectivity works

## 14. Most important commands

show xconnect all
show interfaces GigabitEthernet0/2
show interfaces pseudowire0
show ip interface brief
show arp

## 15. What is proven
xConnect has successfully formed an MPLS pseudowire between PE1 and PE2.
Customer Ethernet traffic can be transported between CE1 and CE2 over the MPLS provider core.
