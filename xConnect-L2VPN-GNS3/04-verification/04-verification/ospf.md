## OSPF Verification## 1. Goal
OSPF in this lab serves as the IGP within the MPLS provider network.
OSPF provides IP reachability between:

* PE1
* P1
* P2
* PE2

It is particularly important that PE routers can reach the Loopback interfaces of the remote PE router, because Loopback addresses are used as LDP router-ID and as endpoint addresses for the MPLS pseudowire.
Customer interfaces towards CE devices are not part of OSPF.
------------------------------
## 2. OSPF Topology

PE1 -------- P1 -------- P2 -------- PE2

 |            |           |            |
Lo1          Lo1         Lo1          Lo1
11.11.11.11  1.1.1.1     2.2.2.2      44.44.44.44

Provider links:

PE1 Gi0/1 <-> P1 Gi0/0
10.10.10.1/30   10.10.10.2/30

P1 Gi0/1 <-> P2 Gi0/0
10.10.10.9/30   10.10.10.10/30

P2 Gi0/1 <-> PE2 Gi0/1
10.10.10.13/30  10.10.10.14/30

------------------------------
## 3. Verification of OSPF neighbor status
On each PE/P device:

show ip ospf neighbor

Expected:
## PE1

P1
State = FULL

## P1

PE1
State = FULL

P2
State = FULL

## P2

P1
State = FULL

PE2
State = FULL

## PE2

P2
State = FULL

------------------------------
## 4. What FULL means
FULL means that the OSPF adjacency has been successfully formed and that the routers have exchanged the required OSPF information.
For this lab, there must be:

PE1 <-> P1       FULL
P1  <-> P2       FULL
P2  <-> PE2      FULL

If any neighbor is not FULL, MPLS transport should not be diagnosed later until the OSPF problem is resolved.
------------------------------
## 5. Verification of OSPF routes
Command:

show ip route ospf

On PE1, there should be a route to remote provider networks and specifically to:

44.44.44.44/32

On PE2, there should be a route to:

11.11.11.11/32

Detailed verification:
On PE1:

show ip route 44.44.44.44

On PE2:

show ip route 11.11.11.11

------------------------------
## 6. Verification of reachability of Loopback addresses
From PE1:

ping 44.44.44.44 source 11.11.11.11

From PE2:

ping 11.11.11.11 source 44.44.44.44

Expected:

!!!!!
Success rate is 100 percent

With this, we prove that the provider IP network has end-to-end reachability between PE Loopback addresses.
------------------------------
## 7. Important note
CE network:

192.168.10.0/24

is not part of OSPF.
CE1:

192.168.10.1/24

CE2:

192.168.10.2/24

Communication between them is not achieved via OSPF routing.
CE-to-CE communication is enabled later by the L2VPN pseudowire.
------------------------------
## 8. OSPF verification checklist

[OK] PE1-P1 adjacency = FULL
[OK] P1-P2 adjacency = FULL
[OK] P2-PE2 adjacency = FULL
[OK] PE1 can reach PE2 Loopback
[OK] PE2 can reach PE1 Loopback
[OK] Remote PE Loopback routes exist
[OK] CE interfaces are not part of OSPF

## 9. Most important commands

show ip ospf neighbor
show ip route ospf
show ip route 44.44.44.44
show ip route 11.11.11.11
ping 44.44.44.44 source 11.11.11.11
ping 11.11.11.11 source 44.44.44.44

## 10. What is proven
OSPF has successfully formed adjacency through the entire provider core and provided IP reachability between PE Loopback addresses.
This is the fundamental prerequisite for LDP and MPLS transport.
