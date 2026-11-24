# Enterprise Wireless Network Configuration Using Cisco Packet Tracer

## 1. Project Overview

This lab documents the design and configuration of a centrally managed enterprise wireless network using Cisco Packet Tracer.

The network serves four departments: IT, HR, Finance and Management. Each department uses a separate WLAN and VLAN. A Wireless LAN Controller (WLC) manages four lightweight access points (LAPs). A router provides inter-VLAN routing, and a server provides DHCP and DNS services.

This README will be updated as the lab is configured and tested.

## 2. Project Objectives

- Configure a router for inter-VLAN routing using router-on-a-stick.
- Create VLANs for IT, HR, Finance and Management.
- Configure a WLC to manage four lightweight access points.
- Create a separate WLAN and SSID for each department.
- Map each WLAN to its corresponding VLAN.
- Configure DHCP and DNS services.
- Verify client addressing and network connectivity.
- Document configuration issues, troubleshooting and lessons learned.

## 3. Tools and Technologies

- Cisco Packet Tracer
- Cisco ISR4331 router
- Cisco Catalyst 2960-24TT switch
- Cisco WLC-2504
- AIR-CAP3702I-A-K9 lightweight access points
- VLANs and 802.1Q trunking
- DHCP and DNS
- Wireless LANs (WLANs)
- Router-on-a-stick

## 4. Network Topology

### Devices

| Device | Quantity |
|---|---:|
| Router | 1 |
| Switch | 1 |
| Wireless LAN Controller | 1 |
| Server | 1 |
| Admin PC | 1 |
| Lightweight APs | 4 |
| Wireless tablets | 4 |

### Physical Connections

| Switch/Router Port | Connected Device |
|---|---|
| Router G0/0 | Server Fa0 |
| Router G0/1 | Switch Fa0/1 |
| Switch Fa0/2 | WLC GigabitEthernet1 |
| Switch Fa0/3 | LAP-PT ADMIN |
| Switch Fa0/4 | LAP-PT HR |
| Switch Fa0/5 | LAP-PT FIN |
| Switch Fa0/6 | LAP-PT IT |
| Switch Fa0/7 | Admin PC Fa0 |

### Topology Diagram

_Add the topology diagram here._

## 5. IP Addressing and VLAN Plan

| Department/Network | VLAN | Network | Default Gateway |
|---|---:|---|---|
| Server network | — | `192.168.1.0/24` | `192.168.1.1` |
| IT | 10 | `192.168.10.0/24` | `192.168.10.1` |
| HR | 20 | `192.168.20.0/24` | `192.168.20.1` |
| Finance | 30 | `192.168.30.0/24` | `192.168.30.1` |
| Management | 50 | `192.168.50.0/24` | `192.168.50.1` |

### Static Addresses

| Device | IP Address | VLAN/Network |
|---|---|---|
| Server | `192.168.1.5/24` | Server network |
| WLC management | `192.168.50.5/24` | VLAN 50 |
| Admin PC | `192.168.50.13/24` | VLAN 50 |

## 6. WLAN and SSID Plan

| Department | WLAN Profile | SSID | VLAN |
|---|---|---|---:|
| IT | `IT-WIFI` | `IT` | 10 |
| HR | `HR-WIFI` | `HR` | 20 |
| Finance | `FIN-WIFI` | `FIN` | 30 |
| Management | `MGT-WIFI` | `MGT` | 50 |

_Wireless passwords are omitted from this public documentation._

## 7. Implementation

### 7.1 Physical Topology

_Record device placement, cabling and any differences from the planned topology._

### 7.2 Router Configuration

_Record interface addressing, VLAN subinterfaces, 802.1Q encapsulation and DHCP relay configuration._

### 7.3 Switch Configuration

_Record VLAN creation, trunk ports, native VLAN and access port assignments._

### 7.4 Server Configuration

_Record the server's static IP configuration, DHCP pools and DNS settings._

### 7.5 WLC Configuration

_Record management interface settings, dynamic interfaces, WLANs, security settings and AP registration._

### 7.6 Wireless Client Configuration

_Record each tablet's SSID, security settings and DHCP address._

## 8. Verification

| Test | Expected Result | Actual Result |
|---|---|---|
| AP registration with WLC | All four APs register | _Pending_ |
| WLAN availability | All four SSIDs are available | _Pending_ |
| Wireless client DHCP | Each tablet gets an address in its VLAN | _Pending_ |
| Client-to-gateway ping | Successful | _Pending_ |
| Admin PC-to-wireless-client ping | Successful, subject to client settings | _Pending_ |
| DNS lookup | Server resolves configured records | _Pending_ |

## 9. Troubleshooting Notes

### Issue 1: _Add issue title_

- **Symptom:** _What failed?_
- **Investigation:** _What was checked?_
- **Cause:** _What caused the issue?_
- **Resolution:** _What change fixed it?_
- **Verification:** _How was the fix confirmed?_

## 10. Key Lessons Learned

_Add lessons as the lab progresses._

## 11. Project Files

| Directory | Contents |
|---|---|
| `packet-tracer/` | Packet Tracer project file |
| `topology/` | Network topology diagram |
| `screenshots/` | Configuration and verification screenshots |

## 12. Conclusion

_Summarise the completed configuration, verification results and lessons learned._
