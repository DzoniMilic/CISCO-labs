# OSPF Verification

## 1. Cilj

OSPF u ovom labu služi kao IGP unutar MPLS provider mreže.

OSPF obezbeđuje IP reachability između:

* PE1
* P1
* P2
* PE2

Posebno je važno da PE ruteri mogu da dosegnu Loopback interfejse udaljenog PE rutera, jer se Loopback adrese koriste kao LDP router-ID i kao endpoint adrese za MPLS pseudowire.

Customer interfejsi prema CE uređajima nisu deo OSPF-a.

---

## 2. Topologija OSPF-a

```text
PE1 -------- P1 -------- P2 -------- PE2
 |            |           |            |
Lo1          Lo1         Lo1          Lo1
11.11.11.11  1.1.1.1     2.2.2.2      44.44.44.44
```

Provider linkovi:

```text
PE1 Gi0/1 <-> P1 Gi0/0
10.10.10.1/30   10.10.10.2/30

P1 Gi0/1 <-> P2 Gi0/0
10.10.10.9/30   10.10.10.10/30

P2 Gi0/1 <-> PE2 Gi0/1
10.10.10.13/30  10.10.10.14/30
```

---

## 3. Provera OSPF susedstva

Na svakom PE/P uređaju:

```text
show ip ospf neighbor
```

Očekivano:

### PE1

```text
P1
State = FULL
```

### P1

```text
PE1
State = FULL

P2
State = FULL
```

### P2

```text
P1
State = FULL

PE2
State = FULL
```

### PE2

```text
P2
State = FULL
```

---

## 4. Šta znači FULL

`FULL` znači da je OSPF adjacency uspešno formiran i da su ruteri razmenili potrebne OSPF informacije.

Za ovaj lab mora postojati:

```text
PE1 <-> P1       FULL
P1  <-> P2       FULL
P2  <-> PE2      FULL
```

Ako bilo koji sused nije `FULL`, MPLS transport kasnije ne treba dijagnostikovati dok se OSPF problem ne reši.

---

## 5. Provera OSPF ruta

Komanda:

```text
show ip route ospf
```

Na PE1 treba da postoji ruta prema udaljenim provider mrežama i posebno prema:

```text
44.44.44.44/32
```

Na PE2 treba da postoji ruta prema:

```text
11.11.11.11/32
```

Detaljna provera:

Na PE1:

```text
show ip route 44.44.44.44
```

Na PE2:

```text
show ip route 11.11.11.11
```

---

## 6. Provera reachability-ja Loopback adresa

Sa PE1:

```text
ping 44.44.44.44 source 11.11.11.11
```

Sa PE2:

```text
ping 11.11.11.11 source 44.44.44.44
```

Očekivano:

```text
!!!!!
Success rate is 100 percent
```

Ovim dokazujemo da provider IP mreža ima end-to-end reachability između PE Loopback adresa.

---

## 7. Važna napomena

CE mreža:

```text
192.168.10.0/24
```

nije deo OSPF-a.

CE1:

```text
192.168.10.1/24
```

CE2:

```text
192.168.10.2/24
```

Komunikacija između njih se ne ostvaruje OSPF routingom.

CE-to-CE komunikaciju kasnije omogućava L2VPN pseudowire.

---

## 8. OSPF verification checklist

```text
[OK] PE1-P1 adjacency = FULL
[OK] P1-P2 adjacency = FULL
[OK] P2-PE2 adjacency = FULL
[OK] PE1 can reach PE2 Loopback
[OK] PE2 can reach PE1 Loopback
[OK] Remote PE Loopback routes exist
[OK] CE interfaces are not part of OSPF
```

## 9. Najvažnije komande

```text
show ip ospf neighbor
show ip route ospf
show ip route 44.44.44.44
show ip route 11.11.11.11
ping 44.44.44.44 source 11.11.11.11
ping 11.11.11.11 source 44.44.44.44
```

## 10. Šta je dokazano

OSPF je uspešno formirao adjacency kroz ceo provider core i obezbedio IP reachability između PE Loopback adresa.

To je osnovni preduslov za LDP i MPLS transport.

