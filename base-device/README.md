# Lab: base-device

A router and a Layer 2 switch on one management subnet, hardened with a hashed enable secret, a local admin user, an MOTD banner, and SSH-only remote access. This is the foundation lab for two articles:

* Cisco Device Base Configuration
* Configure SSH on Cisco Routers and Switches

## Topology

```text
       gi0/0               gi0/0
     [ R1 ]--------------------[ SW1 ]
     c/200                     IOSvL2
     192.168.10.1/24           192.168.10.2/24 (vlan1)
```

## Addressing

| Device | Role | Interface | IP address |
| :--- | :--- | :--- | :--- |
| R1 | Router | GigabitEthernet0/0 | 192.168.10.1/24 |
| SW1 | L2 switch | Vlan1 (SVI) | 192.168.10.2/24 |

Credentials (lab only, change before real use):

* **enable secret**: `C1sc0-Lab1`
* **local user**: `admin` / `Admin-Lab!` (privilege 15)
* **console password**: `Con-Lab!`
