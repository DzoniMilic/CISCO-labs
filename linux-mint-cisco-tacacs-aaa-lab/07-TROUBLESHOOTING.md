# Troubleshooting Notes

This file records issues encountered while building this specific Mint/GNS3 lab. Each entry should include the symptom, the evidence used to narrow it down, the change made, and the verification result.

## Mint2 could not reach R1 on the server subnet

**Symptom:** Mint2 had `192.168.51.2/24`, but pinging R1 at `192.168.51.1` returned `Destination Host Unreachable`. R1 `GigabitEthernet0/1` showed `up/up`, so the topology appeared connected at first glance.

**Evidence:** `tcpdump -ni ens36 arp` showed repeated ARP requests from `192.168.51.2` asking for `192.168.51.1`, with no reply. The Mint2 guest had two interfaces: `ens33` was down and `ens36` was the active lab interface. R1 initially showed no received traffic on `GigabitEthernet0/1`.

**Cause and fix:** The GNS3 VMware node exposed one adapter and the topology link used `e0`, while the active Mint2 lab interface corresponded to the second adapter. The GNS3 adapter count was changed to two, and the R1 `Gi0/1` link was moved to Mint2 `e1`.

**Verification:** Mint2 then reached `192.168.51.1`; after adding a route through R1 on each Mint VM, both VMs reached each other across the router.

**Lesson:** Check the guest interface-to-GNS3-port mapping when a VM link looks up but ARP receives no reply. A green link and an `up/up` router interface do not, by themselves, prove that the intended guest NIC is carrying the lab traffic.

## First ping packet timed out

The first echo request occasionally timed out while the adjacent devices resolved each other's MAC addresses. The next two replies succeeded. Record this as ARP warm-up only when the repeat test is clean; persistent loss needs separate investigation.
