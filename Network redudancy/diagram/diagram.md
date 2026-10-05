```mermaid

flowchart TB
    %% CCNA Project – Branch Network Redundancy

    EXT["EXT-SRV<br/>8.8.8.8/24<br/>GW 8.8.8.1"]
    INET["INET<br/>G0/0 100.64.1.2/30<br/>G0/1 100.64.2.2/30<br/>G0/2 8.8.8.1/24"]

    ISP1["ISP1 (Primary)<br/>G0/0 203.0.113.1/30<br/>G0/0 100.64.1.1/30"]
    ISP2["ISP2 (Backup)<br/>G0/0 198.51.100.1/30<br/>G0/0 100.64.2.1/30"]

    R1["R1 Branch Edge Router<br/>G0/3 203.0.113.2/30<br/>G0/2 198.51.100.2/30<br/>G0/0 192.168.10.1/24<br/>G0/1 192.168.20.1/24"]

    PC1["PC1 Users<br/>192.168.10.10/24<br/>GW 192.168.10.1"]
    SRV1["SRV1 Servers<br/>192.168.20.10/24<br/>GW 192.168.20.1"]

    EXT ---|8.8.8.0/24| INET
    ISP1 ---|100.64.1.0/30| INET
    ISP2 ---|100.64.2.0/30| INET

    ISP1 ---|203.0.113.0/30| R1
    ISP2 ---|198.51.100.0/30| R1

    R1 ---|192.168.10.0/24| PC1
    R1 ---|192.168.20.0/24| SRV1
