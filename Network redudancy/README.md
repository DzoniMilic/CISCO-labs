# Network Redundancy Lab

## 1. Project Overview

The purpose of this GNS3 lab is to demonstrate WAN redundancy using two Internet Service Providers (ISPs).

The network uses:

- **ISP1** as the primary WAN connection.
- **ISP2** as the backup WAN connection.
- Static routing.
- Floating static routes with a higher Administrative Distance (AD).
- Failover and failback testing.

The lab demonstrates how traffic can automatically switch to a backup ISP when the primary path becomes unavailable and return to the primary path when connectivity is restored.

## 2. Network Topology

```text
                         EXT-SRV
                       8.8.8.8/24
                      GW 8.8.8.1
                           |
                       INET G0/2
                       8.8.8.1/24
                           |
              +------------+------------+
              |                         |
          INET G0/0                 INET G0/1
         100.64.1.2/30             100.64.2.2/30
              |                         |
          ISP1 G0/0                 ISP2 G0/1
         100.64.1.1/30             100.64.2.1/30
              |                         |
          ISP1 G0/0                 ISP2 G0/0
         203.0.113.1/30           198.51.100.1/30
              |                         |
          R1 G0/2                  R1 G0/3
         203.0.113.2/30           198.51.100.2/30
              \                         /
               \                       /
                         R1
                      /      \
                 G0/0          G0/1
             192.168.10.1  192.168.20.1
                  |              |
                 PC1            SRV1
            192.168.10.10  192.168.20.10
```

## 3. IP Addressing Table

| Device | Interface | IP Address | Subnet Mask | Notes |
|---|---|---|---|---|
| R1 | G0/0 | 192.168.10.1 | 255.255.255.0 | PC1 gateway |
| R1 | G0/1 | 192.168.20.1 | 255.255.255.0 | SRV1 gateway |
| R1 | G0/2 | 203.0.113.2 | 255.255.255.252 | Primary WAN |
| R1 | G0/3 | 198.51.100.2 | 255.255.255.252 | Backup WAN |
| ISP1 | G0/0 | 203.0.113.1 | 255.255.255.252 | To R1 |
| ISP1 | G0/0 | 100.64.1.1 | 255.255.255.252 | To INET |
| ISP2 | G0/0 | 198.51.100.1 | 255.255.255.252 | To R1 |
| ISP2 | G0/1 | 100.64.2.1 | 255.255.255.252 | To INET |
| INET | G0/0 | 100.64.1.2 | 255.255.255.252 | To ISP1 |
| INET | G0/1 | 100.64.2.2 | 255.255.255.252 | To ISP2 |
| INET | G0/2 | 8.8.8.1 | 255.255.255.0 | External gateway |
| PC1 | Fa0 | 192.168.10.10 | 255.255.255.0 | Endpoint |
| SRV1 | Fa0 | 192.168.20.10 | 255.255.255.0 | Endpoint |
| EXT-SRV | Fa0 | 8.8.8.8 | 255.255.255.0 | External server |

**Note:** The original addressing table lists ISP1 interface G0/0 twice. The interface assignment for the connection to INET should be verified against the actual GNS3 configuration.

## 4. Routing Design

### R1

R1 uses two default static routes:

- Primary route via ISP1: `203.0.113.1`, AD 1.
- Backup floating static route via ISP2: `198.51.100.1`, AD 10.

The primary route has a lower Administrative Distance and is preferred while installed in the routing table.

The backup route becomes active when the primary route is removed from the routing table.

### ISP1

ISP1 forwards traffic towards the internal LAN networks through R1 and uses INET as its upstream path.

### ISP2

ISP2 provides the backup path towards R1 and uses INET as its upstream path.

### INET

INET has return routes towards the internal LAN networks:

- Primary static routes via ISP1, AD 1.
- Backup floating static routes via ISP2, AD 10.

INET also has static routes towards both R1–ISP transit networks.

## 5. Expected Network Behavior

### Normal Operation

All traffic between the internal LAN networks and the external server uses ISP1.

```text
PC1 / SRV1
     |
     R1
     |
   ISP1  (PRIMARY)
     |
    INET
     |
  EXT-SRV
  8.8.8.8
```

Return traffic also uses ISP1.

### Failover

When the primary path becomes physically unavailable and the primary routes are removed from the routing tables, the floating static routes with AD 10 become active.

Traffic switches to ISP2.

```text
PC1 / SRV1
     |
     R1
     |
   ISP2  (BACKUP)
     |
    INET
     |
  EXT-SRV
  8.8.8.8
```

### Failback

When the primary path is restored, the primary static routes with AD 1 become preferred again.

Traffic returns to ISP1.

## 6. Important Observation: Floating Static Route Limitations

During testing, the lab demonstrated an important limitation of traditional floating static routing.

A floating static route reacts to the availability of its next-hop or outgoing interface. It does not independently verify end-to-end connectivity to a remote destination.

This means that the following situation is possible:

- The WAN interface on R1 remains UP.
- The primary default route through ISP1 remains installed.
- The upstream connection beyond ISP1 is unavailable.

In this scenario, R1 may continue forwarding traffic towards ISP1 even though the Internet destination is unreachable.

**Possible future enhancement:** Implement IP SLA and Object Tracking to monitor remote reachability and remove the primary static route when the connectivity test fails.

This enhancement is a separate development step and is not part of the basic floating static routing configuration.

## 7. Verification Commands

The following commands are useful for checking the network configuration and routing behavior:

```cisco
show ip interface brief
show ip route
show ip route 0.0.0.0
show ip route 192.168.10.0
show ip route 192.168.20.0
show running-config
ping 8.8.8.8
```

### R1 Verification

Normal operation:

```text
S* 0.0.0.0/0 [1/0] via 203.0.113.1
```

Backup operation:

```text
S* 0.0.0.0/0 [10/0] via 198.51.100.1
```

### INET Verification

Normal operation:

```text
192.168.10.0/24 via 100.64.1.1
192.168.20.0/24 via 100.64.1.1
```

Backup operation:

```text
192.168.10.0/24 via 100.64.2.1
192.168.20.0/24 via 100.64.2.1
```

End-to-end connectivity should be tested from both PC1 and SRV1 towards `8.8.8.8`.

## 8. Project Structure

Device configurations are stored in the `configs/` directory:

```text
Network redundancy/
│
├── README.md
│
└── configs/
    ├── R1.txt
    ├── ISP1.txt
    ├── ISP2.txt
    └── INET.txt
```

The README documents the project topology, addressing, routing design and test scenarios.

For exact device configurations, refer to the corresponding configuration files in the `configs/` directory.
