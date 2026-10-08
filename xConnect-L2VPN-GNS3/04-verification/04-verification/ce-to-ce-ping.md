# CE-to-CE Ping Verification

## 1. Cilj

Ovo je završni end-to-end test celog laba.

Customer uređaji:

```text
CE1
Gi0/0 = 192.168.10.1/24
```

i:

```text
CE2
Gi0/0 = 192.168.10.2/24
```

nalaze se u istoj Layer-3 subnet mreži:

```text
192.168.10.0/24
```

Između njih nema Layer-3 routing-a na PE uređajima.

Saobraćaj se prenosi kao Layer-2 Ethernet preko xConnect pseudowire-a.

---

## 2. Provera CE1 interfejsa

Na CE1:

```text
show ip interface brief
```

Očekivano:

```text
GigabitEthernet0/0    192.168.10.1    up    up
```

---

## 3. Provera CE2 interfejsa

Na CE2:

```text
show ip interface brief
```

Očekivano:

```text
GigabitEthernet0/0    192.168.10.2    up    up
```

---

## 4. CE1 -> CE2 ping

Na CE1:

```text
ping 192.168.10.2
```

Uspešan rezultat:

```text
!!!!!
Success rate is 100 percent (5/5)
```

U ovom labu dobijen je uspešan rezultat:

```text
Success rate is 100 percent (5/5)
round-trip min/avg/max = 12/16/24 ms
```

---

## 5. CE2 -> CE1 ping

Na CE2:

```text
ping 192.168.10.1
```

Očekivano:

```text
!!!!!
Success rate is 100 percent (5/5)
```

Obostrani ping je važan jer potvrđuje da komunikacija radi u oba smera.

---

## 6. Šta se dešava kada CE1 ping-uje CE2

CE1 proverava da je:

```text
192.168.10.2
```

u istoj lokalnoj subnet mreži.

Zatim CE1 koristi ARP da sazna MAC adresu CE2.

ARP broadcast ide:

```text
CE1
 ↓
PE1 Gi0/2
 ↓
xConnect
 ↓
MPLS pseudowire
 ↓
PE2 Gi0/2
 ↓
CE2
```

CE2 odgovara svojim MAC adresama.

Nakon toga ICMP Ethernet frame ide istim L2VPN putem između CE uređaja.

---

## 7. Bitna stvar: PE ruteri ne rutiraju 192.168.10.0/24

PE1 nema potrebu za:

```text
192.168.10.0/24
```

u svojoj routing tabeli zbog ovog xConnect-a.

Isto važi za PE2.

Provider core zna kako da dođe do:

```text
PE1 Loopback = 11.11.11.11
PE2 Loopback = 44.44.44.44
```

MPLS/LDP obezbeđuje transport kroz core.

xConnect povezuje customer Ethernet domene.

---

## 8. End-to-end lanac

```text
CE1
192.168.10.1
   |
   | Ethernet
   |
PE1
11.11.11.11
   |
   | MPLS
   |
P1
1.1.1.1
   |
   | MPLS
   |
P2
2.2.2.2
   |
   | MPLS
   |
PE2
44.44.44.44
   |
   | Ethernet
   |
CE2
192.168.10.2
```

---

## 9. Šta CE-to-CE ping dokazuje

Uspešan ping dokazuje da su istovremeno funkcionalni:

```text
CE interfaces
      ↓
PE attachment circuits
      ↓
xConnect
      ↓
MPLS pseudowire
      ↓
LDP
      ↓
MPLS core
      ↓
OSPF reachability
      ↓
remote PE
      ↓
CE2
```

Zato je CE-to-CE ping najvažniji završni test ovog laba.

---

## 10. Troubleshooting ako ping ne radi

Ako:

```text
CE1 -> CE2 = FAIL
```

ne treba odmah menjati CE konfiguraciju.

Proveravati redom:

### 1. CE interfejsi

```text
show ip interface brief
```

Mora biti:

```text
up/up
```

### 2. PE attachment circuit

Na PE1/PE2:

```text
show interfaces GigabitEthernet0/2
```

Mora biti:

```text
up/up
```

### 3. xConnect

```text
show xconnect all
```

Mora biti:

```text
UP
```

### 4. LDP

```text
show mpls ldp neighbor
```

Svi relevantni susedi moraju biti:

```text
Oper
```

### 5. PE Loopback reachability

Sa PE1:

```text
ping 44.44.44.44 source 11.11.11.11
```

### 6. ARP

Na CE1:

```text
show arp
```

Na CE2:

```text
show arp
```

Ako CE uređaji ne uče međusobne MAC/ARP informacije, proveriti L2 putanju.

---

## 11. Završni verification checklist

```text
[OK] CE1 Gi0/0 = 192.168.10.1/24
[OK] CE2 Gi0/0 = 192.168.10.2/24
[OK] CE1 interface = up/up
[OK] CE2 interface = up/up

[OK] PE1 Gi0/2 = up/up
[OK] PE2 Gi0/2 = up/up

[OK] OSPF adjacencies = FULL
[OK] PE Loopbacks reachable

[OK] LDP neighbors = Oper
[OK] MPLS forwarding operational

[OK] PE1 xConnect = UP
[OK] PE2 xConnect = UP
[OK] Attachment circuits = UP
[OK] Pseudowire = UP

[OK] CE1 -> CE2 ping = 100%
[OK] CE2 -> CE1 ping = 100%
```

---

## 12. Final result

Cisco xConnect L2VPN je uspešno realizovan preko MPLS provider mreže.

Customer mreža:

```text
192.168.10.0/24
```

funkcioniše kao jedna Layer-2 Ethernet domena preko provider infrastrukture:

```text
CE1 — PE1 — P1 — P2 — PE2 — CE2
```

Provider core koristi:

```text
OSPF
LDP
MPLS
```

dok xConnect obezbeđuje:

```text
Layer-2 pseudowire
```

između PE1 i PE2.

Konačni dokaz funkcionalnosti:

```text
CE1 -> CE2 = 100%
CE2 -> CE1 = 100%
```

