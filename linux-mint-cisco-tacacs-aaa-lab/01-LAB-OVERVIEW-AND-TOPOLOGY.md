# Lab Network and Connectivity

## Topology

The lab uses two separate IPv4 networks with R1 routing between them:

```text
Mint1 -- SW1 -- R1 -- Mint2
client         router  TACACS+ server
```

Mint1 connects to the Layer 2 switch. The switch uplinks to R1 `GigabitEthernet0/0`. Mint2 connects to R1 `GigabitEthernet0/1`.

## Address plan

| Device | Interface | Address |
|---|---|---|
| Mint1 | active lab NIC | `192.168.50.20/24` |
| SW1 | access/uplink ports | Layer 2 only |
| R1 | `GigabitEthernet0/0` | `192.168.50.1/24` |
| R1 | `GigabitEthernet0/1` | `192.168.51.1/24` |
| Mint2 | active lab NIC | `192.168.51.2/24` |

## R1 interface setup

```text
configure terminal
interface GigabitEthernet0/0
 description TO-SW1-CLIENT-NETWORK
 ip address 192.168.50.1 255.255.255.0
 no shutdown
exit
interface GigabitEthernet0/1
 description TO-MINT2-SERVER-NETWORK
 ip address 192.168.51.1 255.255.255.0
 no shutdown
end
show ip interface brief
```

Both routed interfaces reached `up/up` after configuration.

## Temporary Linux addressing used during the build

The first connectivity checks used runtime-only `ip` commands. These settings disappear when a VM is powered off or restarted.

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

The final interface names and persistent NetworkManager settings still need to be recorded after they are configured and verified.

## Connectivity checks

The following paths were verified:

- Mint1 to R1 client interface: `192.168.50.1`
- Mint2 to R1 server interface: `192.168.51.1`
- Mint1 to Mint2 across R1: `192.168.51.2`
- Mint2 to Mint1 across R1: `192.168.50.20`

The first echo request sometimes timed out while ARP resolution completed; subsequent replies succeeded. Repeat the checks after persistent addressing is configured and capture the final outputs for the screenshot index.
