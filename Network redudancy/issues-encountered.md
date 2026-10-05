# Issues Encountered

## 1. Return-path routing problem during WAN failover

### Symptom
During the primary WAN failure test, R1 successfully failed over to ISP2 and could reach the external server. However, **PC1 and SRV1 could not complete ping or traceroute to 8.8.8.8**.

Using Packet Tracer simulation mode showed that:
- outbound traffic from the branch correctly followed the **ISP2 path**
- return traffic from the INET router was still forwarded toward **ISP1**

Because the R1–ISP1 serial link was down, the return packets were dropped.

### Cause
The INET router still preferred ISP1 for the branch LAN return routes:
```cisco
ip route 192.168.10.0 255.255.255.0 100.64.1.1
ip route 192.168.20.0 255.255.255.0 100.64.1.1
```
INET remained unaware of the downstream failure because its own interface toward ISP1 stayed operational. This caused **asymmetric routing**, where the forward and return paths used different ISPs.

### Fix
Adjusted static routes on INET so that return traffic preference aligned with the intended WAN design.

Final configuration:
```cisco
ip route 192.168.10.0 255.255.255.0 100.64.1.1
ip route 192.168.20.0 255.255.255.0 100.64.1.1
ip route 192.168.10.0 255.255.255.0 100.64.2.1 10
ip route 192.168.20.0 255.255.255.0 100.64.2.1 10
```
This ensured that return traffic followed the **primary ISP1 path under normal conditions**, while ISP2 remained available as a standby path.

---

## 2. Missing return routes for branch WAN subnets

### Symptom
R1 initially failed to ping some upstream transit interfaces and the external server even though PC1 could reach the destination.

### Cause
The INET router did not initially have routes for the branch WAN /30 networks:

- `203.0.113.0/30`
- `198.51.100.0/30`

Without these routes, return packets to R1's WAN interfaces could not be forwarded correctly.

### Fix
Added static routes on INET for the WAN transit subnets:
```cisco
ip route 203.0.113.0 255.255.255.252 100.64.1.1
ip route 198.51.100.0 255.255.255.252 100.64.2.1
```

---

## 3. Serial interface clocking (DCE/DTE)

### Symptom
Serial WAN links initially failed to come up during topology deployment.

### Cause
Clocking must be provided by the **DCE side of a serial connection**.

### Fix
Configured the ISP routers as the DCE side and applied clocking:
```cisco
clock rate 64000
```

This allowed the serial links to establish properly.

---

## 4. Router-originated traffic behavior during failover

### Symptom
Ping results from R1 differed depending on which WAN path was active.

### Cause
Router-originated traffic follows the **currently active routing table entry**, typically the default route unless a more specific route exists.

During failover, R1 switched its default route to ISP2, which changed the path used by router-generated traffic.

### Fix
Validation focused on:
- verifying the active default route
- checking traceroute paths
- confirming end-to-end host connectivity

rather than relying solely on individual interface ping tests.




## Half-restored failback: R1 and INET disagreeing on the primary ISP


### Test setup
To simulate a primary WAN failure, the ISP1 path was taken down by shutting the R1 interface toward ISP1 (Gi0/3) and the INET interface toward ISP1 (Gi0/0). In this state the network behaved as designed:
- R1 switched its default route to the floating static route via ISP2
- INET switched the return route for the branch LAN to the backup next-hop (192.168.10.0/24 via 100.64.2.1, AD 10)


### Symptom
During the failback phase, only the R1 interface (Gi0/3) was brought back up first. After that, R1 could not reach the external server, even though ISP2 was fully functional:
```
R1#ping 8.8.8.8
.....
Success rate is 0 percent (0/5)
```

### Cause
As soon as Gi0/3 came up, R1 returned to the primary default route:
```cisco
S*  0.0.0.0/0 [1/0] via 203.0.113.1
```
However, the INET interface toward ISP1 (Gi0/0, 100.64.1.2) was still administratively shut down. This created a split state:

- R1 considered ISP1 the primary and healthy path, because its local link was up
- ISP1 tried to forward traffic toward 100.64.1.2, which was down
- INET was still using ISP2 for the return path (AD 10 route)

The two sides disagreed about which ISP was active, so the traffic path was broken in the middle of the network, while the ISP2 path was never used because R1 preferred the AD 1 route.

### Fix
Brought up the INET interface toward ISP1:
```
conf t
interface Gi0/0
no shutdown
end
```
### Verification
INET returned to the primary return route:
```INET#show ip route 192.168.10.0
Routing entry for 192.168.10.0/24
  Known via "static", distance 1, metric 0
  Routing Descriptor Blocks:
  * 100.64.1.1
      Route metric is 0, traffic share count is 1
```
And R1 reached the external server again through ISP1:
```
R1#ping 8.8.8.8
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 6/7/10 ms
```
### Test Results Summary

Phase	Active path	Result
Normal operation	PC1 - R1 - ISP1 - INET - 8.8.8.8	OK
ISP1 link down (R1 Gi0/3 and INET Gi0/0 shut)	PC1 - R1 - ISP2 - INET - 8.8.8.8	OK (failover)
R1 Gi0/3 restored, INET Gi0/0 still down	R1 selects ISP1, path broken at INET	FAIL (partial state)
INET Gi0/0 restored	PC1 - R1 - ISP1 - INET - 8.8.8.8	OK (failback, 5/5)


