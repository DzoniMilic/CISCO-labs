# MPLS L2VPN xConnect — Concepts, Configuration, and Verification

## 1. Lab Objective

The goal of this lab was to connect two remote customer locations, CE1 and CE2, through the service provider's MPLS network so that they can communicate as if a virtual Ethernet connection existed between them.

Topology:

```text
CE1 ---- PE1 ---- P1 ---- P2 ---- PE2 ---- CE2
```

* **CE1 and CE2 (Customer Edge)** represent customer routers at remote locations.
* **PE1 and PE2 (Provider Edge)** represent service provider routers where the customer L2VPN service terminates.
* **P1 and P2 (Provider)** represent routers inside the service provider's core network. Their role is to forward MPLS traffic between the PE routers.

Customer network IP addresses:

* CE1: `192.168.10.1/24`
* CE2: `192.168.10.2/24`

Both CE routers belong to the same IP subnet, `192.168.10.0/24`.

**Expected result:** CE1 and CE2 can communicate with each other, while P1 and P2 do not need interfaces in the customer network `192.168.10.0/24`.

---

## 2. Why Is This L2VPN and Not L3VPN?

L2VPN stands for Layer 2 Virtual Private Network. In this lab, the service provider delivers a virtual Ethernet connection between two customer locations.

To understand the difference, it is necessary to distinguish between two network layers:

* **Layer 2** processes Ethernet frames and MAC addresses.
* **Layer 3** processes IP packets and IP addresses, and routers make forwarding decisions based on IP routing.

In this lab, PE1 and PE2 connect their Ethernet interfaces using an xConnect configuration. Ethernet traffic from one side is transported through the MPLS network to the other side.

The PE routers do not route customer traffic through separate customer VRF tables, as is typically done in MPLS L3VPN. Instead, they transport Ethernet frames through a pseudowire.

In L3VPN, PE routers participate in routing customer IP networks, usually using VRFs and MP-BGP. In this L2VPN service, the focus is on connecting Ethernet interfaces.

**Remember:** L2VPN provides a virtual Layer 2 connection, while L3VPN provides a routed Layer 3 VPN service.

---

## 3. IP Addressing and Loopback Interfaces

Before MPLS can transport traffic, the routers must have IP connectivity through the provider network.

### 3.1. IP Addresses on Links

I used `/30` subnets between neighboring routers:

| Link   | First address    | Second address    | Subnet           |
| ------ | ---------------- | ----------------- | ---------------- |
| PE1–P1 | PE1 `10.10.10.1` | P1 `10.10.10.2`   | `10.10.10.0/30`  |
| P1–P2  | P1 `10.10.10.9`  | P2 `10.10.10.10`  | `10.10.10.8/30`  |
| P2–PE2 | P2 `10.10.10.13` | PE2 `10.10.10.14` | `10.10.10.12/30` |

A `/30` subnet contains four addresses: the network address, two usable interface addresses, and the broadcast address.

This choice is suitable for point-to-point links connecting two routers.

### 3.2. Loopback Addresses

In this lab, I used the following loopback addresses:

| Router | Loopback address |
| ------ | ---------------- |
| PE1    | `11.11.11.11/32` |
| P1     | `1.1.1.1/32`     |
| P2     | `2.2.2.2/32`     |
| PE2    | `44.44.44.44/32` |

A loopback is a logical interface that is not tied to a single physical link. It is used as a stable router identity.

The `/32` mask identifies one specific IP address.

Loopback addresses are important because we use them as LDP identities and as the addresses of remote PE routers in the xConnect configuration.

For example, `44.44.44.44` identifies PE2 in the configuration on PE1.

However, the loopback address itself does not provide reachability. Other routers must learn a route to it through a routing protocol.

---

## 4. OSPF — Providing IP Routing Through the Core

OSPF (Open Shortest Path First) is an IGP used for routing within the service provider's network.

Its role in this lab is to allow routers to learn routes to remote networks and loopback addresses.

For example, PE1 must know how to reach PE2's loopback address `44.44.44.44`. The path goes through P1 and P2:

```text
PE1 → P1 → P2 → PE2
```

OSPF enables the exchange of routing information and the calculation of the best path through the network.

### What Does OSPF Do?

1. Enables neighbor adjacency formation between appropriate routers.
2. Exchanges information about available networks.
3. Enables remote loopback addresses to be learned.
4. Provides the IP reachability required for MPLS transport.

OSPF does not establish a pseudowire and does not transport Ethernet frames as an L2VPN service. It provides the IP routing foundation on which MPLS operates.

### Verification

```cisco
show ip ospf neighbor
show ip route
show ip route 44.44.44.44
```

On PE1, I expect a route to `44.44.44.44/32` to exist. OSPF adjacencies on the appropriate links should be in the `FULL` state.

If OSPF does not work, I first check IP addresses, masks, interface status, OSPF configuration, and area membership.

---

## 5. MPLS and LDP — Label-Based Transport

MPLS (Multiprotocol Label Switching) enables traffic forwarding using labels.

A label is a numeric identifier contained in the MPLS header. A router uses its forwarding table to determine what to do with the label, rather than relying exclusively on the destination IP address at every step.

LDP (Label Distribution Protocol) enables MPLS routers to establish sessions and exchange label information.

### 5.1. LDP Router ID

In the lab, I used:

```cisco
mpls ldp router-id Loopback1
```

This command specifies the loopback interface whose address will be used as the router's LDP identity.

This provides a stable LDP identity that is not directly tied to the address of a single physical interface.

### 5.2. MPLS LDP Autoconfiguration

In the OSPF configuration, I used:

```cisco
mpls ldp autoconfig
```

This command enables automatic activation of MPLS/LDP functionality on the appropriate OSPF interfaces, subject to the capabilities of the IOS platform.

### 5.3. How Does Transport Work?

At a high level:

1. OSPF provides an IP path to the remote PE router.
2. LDP exchanges label information with neighboring MPLS routers.
3. The ingress PE sends MPLS traffic through the core.
4. P routers use their MPLS forwarding tables to forward traffic toward the next hop.
5. The traffic reaches the egress PE router.

In a pseudowire architecture, a transport label is typically used to reach the remote PE, while a pseudowire label identifies the specific L2VPN connection.

Label values are not universal. Routers may assign different local values.

### Verification

```cisco
show mpls interfaces
show mpls ldp neighbor
show mpls forwarding-table
```

`show mpls interfaces` checks whether MPLS is active on the interfaces.

`show mpls ldp neighbor` checks whether LDP neighbors exist and whether the sessions are operational.

`show mpls forwarding-table` displays information about labels and MPLS forwarding.

It is important to understand that an active MPLS interface does not, by itself, prove that all LDP sessions are established. Therefore, I also check LDP neighbors.

---

## 6. xConnect — Configuring the L2VPN Service

Once IP routing and MPLS transport are working, I need to create a virtual Ethernet connection between PE1 and PE2.

I do this using the xConnect command.

### 6.1. Configuration on PE1

```cisco
interface GigabitEthernet0/2
 no ip address
 no shutdown
 xconnect 44.44.44.44 10 encapsulation mpls
```

### 6.2. Configuration on PE2

```cisco
interface GigabitEthernet0/2
 no ip address
 no shutdown
 xconnect 11.11.11.11 10 encapsulation mpls
```

### Command Explanation

`interface GigabitEthernet0/2` selects the Ethernet interface connected to the customer CE router.

`no ip address` removes Layer 3 addressing from the interface used as an Ethernet attachment circuit. Traffic is transported as a Layer 2 service rather than being routed as it would be on a regular Layer 3 interface.

`no shutdown` enables the interface.

`xconnect 44.44.44.44 10 encapsulation mpls` on PE1 defines the remote PE2, identified by loopback address `44.44.44.44`, VC ID `10`, and MPLS encapsulation.

On PE2, the opposite loopback address, `11.11.11.11`, is used with the same VC ID.

### 6.3. What Is a VC ID?

VC stands for Virtual Circuit. The VC ID identifies the virtual connection between the PE routers.

In this lab, both ends use VC ID `10`.

The VC ID is not an IP address and is not an MPLS label. It identifies the pseudowire service at both ends.

If multiple pseudowires existed between PE routers, the corresponding identifiers would allow them to be distinguished.

### 6.4. What Is a Pseudowire?

A pseudowire is a logical connection that transports Ethernet frames between two remote attachment circuits across the provider network.

In our topology:

* PE1 `Gi0/2` is the attachment circuit toward CE1.
* PE2 `Gi0/2` is the attachment circuit toward CE2.
* The pseudowire transports Ethernet traffic between these two endpoints through the MPLS core.

A pseudowire can be thought of as a virtual Ethernet cable between PE1 and PE2, although the actual transport takes place through multiple routers and MPLS labels.

---

## 7. An Attachment Circuit and a Pseudowire Are Not the Same

An attachment circuit (AC) is the local customer-facing connection on a PE router.

A pseudowire is the virtual connection between remote PE routers.

In this lab:

```text
CE1 — AC — PE1 ===== Pseudowire over MPLS ===== PE2 — AC — CE2
```

PE1 receives Ethernet frames from CE1 through its AC. It then transports them through the pseudowire to PE2. PE2 delivers them through its AC toward CE2.

That is why xConnect displays two components:

* The local AC.
* The remote MPLS pseudowire.

Both must be operational for the service to function as expected.

---

## 8. How Does Traffic Travel from CE1 to CE2?

On CE1, I executed:

```cisco
ping 192.168.10.2
```

CE1 has the address `192.168.10.1/24`. Since the destination `192.168.10.2` belongs to the same subnet, CE1 considers it directly reachable.

### Step 1 — ARP

CE1 needs to learn the MAC address of the device with IP address `192.168.10.2`. It therefore uses ARP.

Depending on the state of the ARP table, an ARP exchange may be required before the ICMP request is sent.

### Step 2 — The Ethernet Frame Reaches PE1

CE1 sends an Ethernet frame over its link to PE1. PE1 receives the frame on the interface configured as the xConnect attachment circuit.

### Step 3 — PE1 Transports the Frame Through the Pseudowire

PE1 encapsulates the customer Ethernet traffic for transport through the MPLS network.

The transport MPLS label is used to reach the remote PE, while the pseudowire label identifies the specific virtual Layer 2 connection.

### Step 4 — P1 and P2 Forward the MPLS Traffic

The P routers use their MPLS forwarding tables to forward the traffic toward PE2.

They do not need interfaces in the customer network `192.168.10.0/24` or need to route between CE1 and CE2.

### Step 5 — PE2 Delivers the Ethernet Frame

PE2 processes the MPLS traffic, identifies the appropriate pseudowire, and sends the original Ethernet traffic through its attachment circuit toward CE2.

### Step 6 — CE2 Returns the Reply

CE2 receives the ICMP Echo Request and sends an ICMP Echo Reply. The response returns through the same logical L2VPN connection in the opposite direction.

This allows CE1 and CE2 to communicate through the MPLS infrastructure even though P1 and P2 do not participate in routing the customer IP network.

---

## 9. Verifying the Service

### 9.1. Checking xConnect

On a PE router:

```cisco
show xconnect all
```

In the lab, I received the following status:

```text
UP pri   ac Gi0/2:4(Ethernet)  UP mpls 44.44.44.44:10  UP
```

This means:

* The AC is operational.
* The MPLS pseudowire is operational.
* The remote endpoint is PE2 at `44.44.44.44`.
* The VC ID is `10`.

The `UP` status indicates that the xConnect service is established.

### 9.2. Checking Customer Traffic

On CE1:

```cisco
ping 192.168.10.2
```

The ping succeeded.

This proves that not only is the control connection between the PE routers established, but customer traffic is actually passing through the complete L2VPN service.

`show xconnect all` checks the service status, while ping checks IP communication between the customer endpoints.

---

## 10. Troubleshooting Order if the L2VPN Does Not Work

When CE1 cannot communicate with CE2, I should not change the configuration randomly. I should perform the checks in order.

1. **Physical interfaces and IP addressing**

   ```cisco
   show ip interface brief
   ```

   Check that the interfaces are enabled and that IP addresses and masks are correct.

2. **OSPF adjacencies**

   ```cisco
   show ip ospf neighbor
   ```

   Check that the expected adjacencies are established.

3. **IP route to the remote PE loopback**

   ```cisco
   show ip route 44.44.44.44
   ```

   On PE1, check whether a route to the PE2 loopback address exists.

4. **MPLS interfaces and LDP neighbors**

   ```cisco
   show mpls interfaces
   show mpls ldp neighbor
   ```

   Check whether MPLS is active on the transport interfaces and whether LDP sessions are operational.

5. **MPLS forwarding**

   ```cisco
   show mpls forwarding-table
   ```

   Check whether the appropriate label information exists for MPLS forwarding.

6. **xConnect status**

   ```cisco
   show xconnect all
   ```

   Check whether the AC and pseudowire are operational and whether the VC ID and remote loopback addresses are correct.

7. **CE-to-CE communication**

   ```cisco
   ping 192.168.10.2
   ```

   Finally, verify actual customer traffic.

This sequence makes it possible to narrow a fault down to a specific part of the network: IP connectivity, OSPF, LDP/MPLS transport, xConnect, or the customer side.

---

## 11. What Did I Learn in This Lab?

In this lab, I combined four different functions:

* **IP addressing** provides basic connectivity between routers.
* **OSPF** enables routing through the provider core and reachability to loopback addresses.
* **LDP/MPLS** provides label-based traffic transport.
* **L2VPN xConnect** connects remote Ethernet attachment circuits using a pseudowire.

These functions have different roles. OSPF does not create an L2VPN. LDP does not define the customer Ethernet service. xConnect does not replace IP routing or MPLS transport.

The successful ping from CE1 to CE2 confirmed that addressing, MPLS transport, the pseudowire, and customer connectivity are functional in this lab.

**Main conclusion:** I created a virtual Ethernet connection between two customer locations across the service provider's MPLS network. PE1 and PE2 are the endpoints of the L2VPN service, P1 and P2 provide transport through the MPLS core, OSPF provides IP reachability, LDP exchanges label information, and xConnect defines the pseudowire that transports Ethernet traffic between the two locations.
