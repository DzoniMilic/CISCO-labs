# Verification 01 — OSPF

## Objective

Verify that OSPF provides IP reachability across the service provider core and that PE1 can reach the remote PE2 loopback address.

## Commands

```cisco
show ip ospf neighbor
show ip route ospf
```

## Results — PE1

### OSPF Neighbor

Observed output:

```text
Neighbor ID     Pri   State       Address       Interface
1.1.1.1           1   FULL/DR      10.10.10.2    GigabitEthernet0/1
```

PE1 has an OSPF neighbor relationship with P1 (`1.1.1.1`) over `GigabitEthernet0/1`.

The `FULL` state indicates that the OSPF adjacency has reached the fully adjacent state.

### OSPF Routes

The following OSPF-learned routes were observed on PE1:

| Destination    | OSPF Metric | Next Hop   | Interface |
| -------------- | ----------: | ---------- | --------- |
| 1.1.1.1/32     |           2 | 10.10.10.2 | Gi0/1     |
| 2.2.2.2/32     |           3 | 10.10.10.2 | Gi0/1     |
| 10.10.10.8/30  |           2 | 10.10.10.2 | Gi0/1     |
| 10.10.10.12/30 |           3 | 10.10.10.2 | Gi0/1     |
| 44.44.44.44/32 |           4 | 10.10.10.2 | Gi0/1     |

The remote PE2 loopback `44.44.44.44/32` is present in the routing table with OSPF administrative distance 110 and metric 4.

## Interpretation

* The OSPF adjacency between PE1 and P1 is operational.
* PE1 has learned the provider core networks and router loopbacks through OSPF.
* The PE2 loopback is reachable through the provider routing domain.
* This reachability is required for the MP-BGP session between PE1 and PE2, which uses loopback addresses as BGP endpoints.

## Conclusion

**OSPF verification passed on PE1** for the observed neighbor and routing information.

This check confirms PE1's local OSPF adjacency and its learned routes. It does not, by itself, prove that every OSPF adjacency in the provider core is operational.

