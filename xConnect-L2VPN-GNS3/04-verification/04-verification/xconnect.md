# xConnect Verification

## 1. Cilj

xConnect predstavlja Layer-2 VPN pseudowire između PE1 i PE2.

U ovom labu:

```text
CE1
 |
PE1
 |
P1
 |
P2
 |
PE2
 |
CE2
```

xConnect endpoint-i:

```text
PE1 Gi0/2
     |
     | VC ID 10
     |
PE2 Gi0/2
```

PE1 koristi udaljeni PE Loopback:

```text
44.44.44.44
```

PE2 koristi:

```text
11.11.11.11
```

---

## 2. Konfiguracija PE1

```text
interface GigabitEthernet0/2
 xconnect 44.44.44.44 10 encapsulation mpls
```

## 3. Konfiguracija PE2

```text
interface GigabitEthernet0/2
 xconnect 11.11.11.11 10 encapsulation mpls
```

VC ID je:

```text
10
```

na obe strane.

---

## 4. Provera xConnect-a

Na PE1:

```text
show xconnect all
```

Očekivano:

```text
XC ST  Segment 1                      S1  Segment 2
UP     ac Gi0/2:4(Ethernet)           UP  mpls 44.44.44.44:10
```

Na PE2:

```text
show xconnect all
```

Očekivano:

```text
XC ST  Segment 1                      S1  Segment 2
UP     ac Gi0/2:4(Ethernet)           UP  mpls 11.11.11.11:10
```

---

## 5. Kako čitati rezultat

Primer:

```text
UP
```

prvi status znači da je xConnect aktivan.

Attachment Circuit:

```text
ac Gi0/2
```

predstavlja lokalnu Ethernet stranu prema CE uređaju.

Druga strana:

```text
mpls 44.44.44.44:10
```

predstavlja MPLS pseudowire prema udaljenom PE.

---

## 6. Šta mora biti UP

Za ispravan xConnect moraju biti UP:

```text
xConnect       = UP
Attachment     = UP
Pseudowire     = UP
```

Ako je:

```text
AC = DOWN
```

problem je lokalno između CE i PE.

Ako je:

```text
AC = UP
PW = DOWN
```

problem je u provider transportu / LDP / udaljenom PE / VC konfiguraciji.

---

## 7. Provera CE-facing interfejsa

Na PE1:

```text
show interfaces GigabitEthernet0/2
```

Na PE2:

```text
show interfaces GigabitEthernet0/2
```

Očekivano:

```text
GigabitEthernet0/2 is up, line protocol is up
```

Interfejs nema IP adresu:

```text
Internet protocol processing disabled
```

ili odgovarajući `unassigned` prikaz u:

```text
show ip interface brief
```

To je normalno.

---

## 8. Zašto PE Gi0/2 nema IP adresu

Gi0/2 nije Layer-3 routed interface.

On predstavlja:

```text
Attachment Circuit
```

od CE uređaja ka pseudowire-u.

CE1 ima:

```text
192.168.10.1/24
```

CE2 ima:

```text
192.168.10.2/24
```

PE1 i PE2 ne moraju imati IP adresu na tim interfejsima.

Ethernet frame sa CE1 se prosleđuje kroz pseudowire do CE2.

---

## 9. Provera pseudowire statusa

Na PE1:

```text
show xconnect all
```

Na PE2:

```text
show xconnect all
```

Mora se videti:

```text
mpls <remote-PE-loopback>:10
```

i status:

```text
UP
```

VC ID mora biti isti na obe strane:

```text
PE1 = 10
PE2 = 10
```

---

## 10. Provera pseudowire interfejsa

Na PE1:

```text
show interfaces pseudowire0
```

Na PE2:

```text
show interfaces pseudowire0
```

Treba proveriti da je pseudowire interfejs operativan.

Tokom uspešnog formiranja pseudowire-a može se pojaviti:

```text
%LINEPROTO-5-UPDOWN:
Line protocol on Interface pseudowire0, changed state to up
```

---

## 11. Provera MAC/ARP ponašanja

Na CE1:

```text
show arp
```

Na CE2:

```text
show arp
```

Nakon međusobnog pinga CE uređaji treba da nauče ARP/MAC informacije potrebne za komunikaciju.

Bitna stvar:

ARP request je broadcast.

xConnect omogućava da se Ethernet broadcast sa jednog CE-a prenese preko pseudowire-a do drugog CE-a.

Zato CE1 i CE2 mogu biti u istoj subnet mreži:

```text
192.168.10.0/24
```

iako između njih postoji MPLS provider core.

---

## 12. End-to-end test

Na CE1:

```text
ping 192.168.10.2
```

Na CE2:

```text
ping 192.168.10.1
```

Ako ping radi, dokazano je da L2VPN funkcioniše end-to-end.

---

## 13. xConnect verification checklist

```text
[OK] PE1 Gi0/2 = up/up
[OK] PE2 Gi0/2 = up/up
[OK] PE1 xConnect = UP
[OK] PE2 xConnect = UP
[OK] Attachment Circuit = UP
[OK] Pseudowire = UP
[OK] VC ID PE1 = 10
[OK] VC ID PE2 = 10
[OK] Remote PE Loopbacks reachable
[OK] CE Ethernet connectivity works
```

## 14. Najvažnije komande

```text
show xconnect all
show interfaces GigabitEthernet0/2
show interfaces pseudowire0
show ip interface brief
show arp
```

## 15. Šta je dokazano

xConnect je uspešno formirao MPLS pseudowire između PE1 i PE2.

Customer Ethernet saobraćaj može da se prenosi između CE1 i CE2 preko MPLS provider core-a.
