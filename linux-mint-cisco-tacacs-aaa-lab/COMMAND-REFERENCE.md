# Linux Mint Command Reference

This reference records the Linux commands used during the network phase of this lab, with the reason for each command. The temporary address and route commands below were used to bring up and test the topology. They are not persistent across a VM restart.

## Inspect interfaces and NetworkManager

Run these on either Mint VM:

```bash
ip -br address
ip -br link
nmcli -f NAME,DEVICE,TYPE connection show --active
```

- `ip -br address` gives a short view of interface addresses.
- `ip -br link` shows interface state, MAC address, and link state.
- `nmcli ... connection show --active` identifies the active NetworkManager profile.

During this lab, both VMs reported an active profile named `ens36`. The `ens33` interface was down; `ens36` was the active lab interface.

To check carrier state on the active interface:

```bash
cat /sys/class/net/ens36/carrier
```

A result of `1` indicated an active link. Checking `ens33/carrier` returned an error because that interface was down; no further work was needed on it.

## Temporary addresses and routes

These commands were used before configuring persistent NetworkManager profiles.

Mint1:

```bash
sudo ip addr add 192.168.50.20/24 dev ens36
sudo ip route add 192.168.51.0/24 via 192.168.50.1
```

Mint2:

```bash
sudo ip addr add 192.168.51.2/24 dev ens36
sudo ip route add 192.168.50.0/24 via 192.168.51.1
```

The first command in each block assigns the VM's lab address. The second adds a route to the opposite subnet through R1. Both commands change runtime state only. The persistent NetworkManager configuration is pending and will replace this temporary setup after it is verified.

## Connectivity checks

```bash
# Run from Mint1
ping -c 3 192.168.50.1
ping -c 3 192.168.51.2

# Run from Mint2
ping -c 3 192.168.51.1
ping -c 3 192.168.50.20
```

These checks verify each VM's local R1 connection and the routed path between Mint1 and Mint2. A ping to a VM's own address, such as `ping -c 3 192.168.51.2` from Mint2, only checks the local stack and does not prove network connectivity.

## ARP and neighbor diagnosis

```bash
ip neigh show dev ens36
sudo tcpdump -ni ens36 arp
```

`ip neigh` shows the current neighbor/ARP table. `tcpdump` listens for ARP frames on the selected interface. While diagnosing Mint2, it showed repeated ARP requests for `192.168.51.1` without a reply. The corrected GNS3 adapter mapping resolved the issue.

## GNS3 adapter mapping note

Mint2's active guest interface was `ens36`, while `ens33` was down. The GNS3 VMware node initially exposed one adapter and connected R1 `Gi0/1` to Mint2 `e0`. Exposing two adapters and moving that cable to Mint2 `e1` connected the active lab NIC; the subsequent ping to `192.168.51.1` succeeded.
README.md
