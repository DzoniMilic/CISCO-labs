# Project Network redundancy Addressing Table

| Device | Interface | IP Address | Mask | Notes |
|---|---|---:|---:|---|
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
