## MPLS / LDP Verification## 1. Goal
MPLS and LDP provide pseudowire transport between PE routers.
Logic:

OSPF
  ↓
IP reachability
  ↓
LDP
  ↓
MPLS labels
  ↓
Pseudowire transport

LDP uses Loopback addresses of provider routers as LDP Router-ID.
------------------------------
## 2. LDP Router-ID
On PE/P devices:

show mpls ldp parameters

LDP Router-ID should be Loopback 1:

PE1 = 11.11.11.11
P1  = 1.1.1.1
P2  = 2.2.2.2
PE2 = 44.44.44.44

Configuration is:

mpls ldp router-id Loopback 1

------------------------------
## 3. Verification of MPLS interfaces
On PE/P devices:

show mpls interfaces

Provider-facing interfaces should be MPLS enabled.
Example:

PE1 Gi0/1
P1  Gi0/0
P1  Gi0/1
P2  Gi0/0
P2  Gi0/1
PE2 Gi0/1

CE-facing interfaces:

PE1 Gi0/2
PE2 Gi0/2

are not MPLS core interfaces.
------------------------------
## 4. Verification of LDP neighbors
Command:

show mpls ldp neighbor

Expected LDP neighbors:
## PE1

1.1.1.1:0
State: Oper

## P1

11.11.11.11:0
State: Oper

2.2.2.2:0
State: Oper

## P2

1.1.1.1:0
State: Oper

44.44.44.44:0
State: Oper

## PE2

2.2.2.2:0
State: Oper

------------------------------
## 5. What State: Oper means
Oper means that the LDP session with the neighbor is operational.
For this lab, there must be:

PE1 <-> P1    LDP Oper
P1  <-> P2    LDP Oper
P2  <-> PE2   LDP Oper

If the LDP session is not Oper, the MPLS pseudowire will not be able to normally use MPLS transport.
------------------------------
## 6. LDP adjacency must follow OSPF topology
Expected structure:

PE1
 |
P1
 |
P2
 |
PE2

LDP does not build a direct session:

PE1 <----> PE2

across the entire core.
LDP sessions exist between directly connected MPLS routers, while MPLS label switching allows the packet/pseudowire to pass through the entire core.
------------------------------
## 7. Verification of MPLS forwarding table
Command:

show mpls forwarding-table

The table should contain MPLS label entries.
It shows how the local router processes MPLS packets:

* Incoming Label
* Outgoing Label
* Outgoing interface
* Next-hop

Example logic:

Incoming Label
      ↓
Label lookup
      ↓
Outgoing Label
      ↓
Next-hop / Interface

The exact number of labels depends on the IOS version and the current state of LDP.
------------------------------
## 8. Verification of reachability of PE Loopback addresses
From PE1:

ping 44.44.44.44 source 11.11.11.11

From PE2:

ping 11.11.11.11 source 44.44.44.44

This confirms that IP reachability exists through the provider core.
For a complete MPLS check, we combine:

show mpls ldp neighbor
show mpls forwarding-table
ping remote-loopback

------------------------------
## 9. Important difference: OSPF vs LDP
OSPF:

PE1 ---- P1 ---- P2 ---- PE2

provides IP routing information.
LDP:

PE1 ---- P1 ---- P2 ---- PE2

distributes MPLS label information.
OSPF by itself does not create an MPLS pseudowire.
LDP by itself does not represent a customer L2VPN.
For xConnect, a complete chain is required:

OSPF
 ↓
IP reachability
 ↓
LDP
 ↓
MPLS transport
 ↓
xConnect / pseudowire
 ↓
CE-to-CE L2 connectivity

------------------------------
## 10. MPLS/LDP verification checklist

[OK] MPLS enabled on provider interfaces
[OK] LDP Router-ID = Loopback 1
[OK] PE1-P1 LDP = Oper
[OK] P1-P2 LDP = Oper
[OK] P2-PE2 LDP = Oper
[OK] MPLS forwarding entries exist
[OK] PE Loopbacks reachable

## 11. Most important commands

show mpls interfaces
show mpls ldp parameters
show mpls ldp neighbor
show mpls forwarding-table
ping 44.44.44.44 source 11.11.11.11
ping 11.11.11.11 source 44.44.44.44

## 12. What is proven
LDP is operational through the entire MPLS core, MPLS forwarding exists, and PE Loopback addresses are mutually reachable.
With this, MPLS transport is ready for the xConnect pseudowire.
