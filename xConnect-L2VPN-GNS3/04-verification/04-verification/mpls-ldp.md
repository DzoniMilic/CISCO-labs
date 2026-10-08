
# MPLS / LDP Verification

## 1. Cilj

MPLS i LDP obezbeđuju transport pseudowire-a između PE rutera.

Logika:

```text
OSPF
  ↓
IP reachability
  ↓
LDP
  ↓
MPLS labels
  ↓
Pseudowire transport
```

LDP koristi Loopback adrese provider rutera kao LDP Router-ID.

---

## 2. LDP Router-ID

Na PE/P uređajima:

```text
show mpls ldp parameters
```

LDP Router-ID treba da bude Loopback 1:

```text
PE1 = 11.11.11.11
P1  = 1.1.1.1
P2  = 2.2.2.2
PE2 = 44.44.44.44
```

Konfiguracija je:

```text
mpls ldp router-id Loopback 1
```

---

## 3. Provera MPLS interfejsa

Na PE/P uređajima:

```text
show mpls interfaces
```

Provider-facing interfejsi treba da budu MPLS enabled.

Primer:

```text
PE1 Gi0/1
P1  Gi0/0
P1  Gi0/1
P2  Gi0/0
P2  Gi0/1
PE2 Gi0/1
```

CE-facing interfejsi:

```text
PE1 Gi0/2
PE2 Gi0/2
```

nisu MPLS core interfejsi.

---

## 4. Provera LDP suseda

Komanda:

```text
show mpls ldp neighbor
```

Očekivani LDP susedi:

### PE1

```text
1.1.1.1:0
State: Oper
```

### P1

```text
11.11.11.11:0
State: Oper

2.2.2.2:0
State: Oper
```

### P2

```text
1.1.1.1:0
State: Oper

44.44.44.44:0
State: Oper
```

### PE2

```text
2.2.2.2:0
State: Oper
```

---

## 5. Šta znači State: Oper

`Oper` znači da je LDP session sa susedom operativan.

Za ovaj lab mora postojati:

```text
PE1 <-> P1    LDP Oper
P1  <-> P2    LDP Oper
P2  <-> PE2   LDP Oper
```

Ako LDP session nije `Oper`, MPLS pseudowire neće moći normalno da koristi MPLS transport.

---

## 6. LDP adjacency mora pratiti OSPF topologiju

Očekivana struktura:

```text
PE1
 |
P1
 |
P2
 |
PE2
```

LDP ne pravi direktan session:

```text
PE1 <----> PE2
```

preko celog core-a.

LDP sessions postoje između direktno povezanih MPLS rutera, dok MPLS label switching omogućava da paket/pseudowire prođe kroz ceo core.

---

## 7. Provera MPLS forwarding table

Komanda:

```text
show mpls forwarding-table
```

Tabela treba da sadrži MPLS label entries.

Ona pokazuje kako lokalni router obrađuje MPLS pakete:

* Incoming Label
* Outgoing Label
* Outgoing interface
* Next-hop

Primer logike:

```text
Incoming Label
      ↓
Label lookup
      ↓
Outgoing Label
      ↓
Next-hop / Interface
```

Tačan broj labela zavisi od IOS verzije i trenutnog stanja LDP-a.

---

## 8. Provera reachability-ja PE Loopback adresa

Sa PE1:

```text
ping 44.44.44.44 source 11.11.11.11
```

Sa PE2:

```text
ping 11.11.11.11 source 44.44.44.44
```

Ovo potvrđuje da postoji IP reachability kroz provider core.

Za potpunu MPLS proveru kombinujemo:

```text
show mpls ldp neighbor
show mpls forwarding-table
ping remote-loopback
```

---

## 9. Važna razlika: OSPF vs LDP

OSPF:

```text
PE1 ---- P1 ---- P2 ---- PE2
```

obezbeđuje IP routing informacije.

LDP:

```text
PE1 ---- P1 ---- P2 ---- PE2
```

distribuira MPLS label informacije.

OSPF sam po sebi ne pravi MPLS pseudowire.

LDP sam po sebi ne predstavlja customer L2VPN.

Za xConnect je potreban kompletan lanac:

```text
OSPF
 ↓
IP reachability
 ↓
LDP
 ↓
MPLS transport
 ↓
xConnect / pseudowire
 ↓
CE-to-CE L2 connectivity
```

---

## 10. MPLS/LDP verification checklist

```text
[OK] MPLS enabled on provider interfaces
[OK] LDP Router-ID = Loopback 1
[OK] PE1-P1 LDP = Oper
[OK] P1-P2 LDP = Oper
[OK] P2-PE2 LDP = Oper
[OK] MPLS forwarding entries exist
[OK] PE Loopbacks reachable
```

## 11. Najvažnije komande

```text
show mpls interfaces
show mpls ldp parameters
show mpls ldp neighbor
show mpls forwarding-table
ping 44.44.44.44 source 11.11.11.11
ping 11.11.11.11 source 44.44.44.44
```

## 12. Šta je dokazano

LDP je operativan kroz ceo MPLS core, MPLS forwarding postoji i PE Loopback adrese su međusobno dostupne.

Time je MPLS transport spreman za xConnect pseudowire.
