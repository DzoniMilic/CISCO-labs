# Cisco Router AAA with TACACS+ on Linux Mint

This lab is being built and documented from a two-subnet GNS3 topology. Linux Mint runs in VMware Workstation; Cisco IOSv and vIOS L2 run in GNS3.

## Current status

The network path between both Mint VMs and R1 is working, including routing between the client and server subnets. TACACS+ installation, AAA configuration, SSH access, and role testing are still to be completed.
The network path between both Mint VMs and R1 is working, including persistent addressing and routing between the client and server subnets. TACACS+ installation, AAA configuration, SSH access, and role testing are still to be completed.


## Current topology

```text
Mint1 (client) -- SW1 -- R1 -- Mint2 (TACACS+ server)
                  L2     IOSv
```

| Node / interface | Address | Purpose |
|---|---:|---|
| Mint1 lab interface | `192.168.50.20/24` | SSH client |
| R1 `Gi0/0` | `192.168.50.1/24` | Client-side gateway |
| R1 `Gi0/1` | `192.168.51.1/24` | Server-side gateway |
| Mint2 lab interface | `192.168.51.2/24` | Planned TACACS+ server |

The two routed links use different subnets. Mint1 reaches the server network through R1, and Mint2 reaches the client network through R1.

## Repository map

- `docs/02-LAB-NETWORK-AND-CONNECTIVITY.md` — current topology, addressing, and connectivity setup.
- `docs/07-TROUBLESHOOTING.md` — issues observed and the fixes verified in this lab.
- `docs/COMMAND-REFERENCE.md` — Linux Mint commands used for interface discovery, temporary addressing, routing, and diagnosis.
- `docs/SCREENSHOT-INDEX.md` — evidence index to be expanded as screenshots are captured.

## Security note

Use placeholders in public examples and screenshots. Do not commit TACACS+ shared secrets, passwords, or private keys. This is an isolated learning lab; legacy SSH algorithms, if required by the selected IOS image, should be documented as lab compatibility settings only.

## Progress

The network documentation is an initial draft. Later sections will be added after the TACACS+ server and Cisco AAA stages have been completed and verified.
